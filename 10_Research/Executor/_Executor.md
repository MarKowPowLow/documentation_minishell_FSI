# L'Executor

### 1. Qu'est-ce qu'un Executor ?

Si on reprend notre fil rouge :

- **Le [[_Lexer|Lexer]]** a fabriqué les briques (Tokens).

- **Le [[_Parser|Parser]]** a dessiné le plan de la maison (L'Arbre [[AST]]).

- **L'Executor**, c'est le **chef de chantier**. Il prend le plan, lit les instructions, et fait le vrai travail : brancher les tuyaux, allumer les machines, et lancer les processus.


Son but est de parcourir ton arbre ([[AST]]) et d'appeler les fonctions système (`pipe`, `fork`, `dup2`, `execve`, `waitpid`).

---

### 2. La récursivité

L'Executor est, de par nature, au vu de notre liste chaînée, une fonction récursive. Il commence toujours par la **Racine** de l'arbre et descend. À chaque fois qu'il arrive sur un nœud, il se pose une seule question : _"Quel est mon type ?"_

Voici comment il va réagir à nos 3 grandes familles de nœuds :

#### A. Les nœuds logiques (`&&`, `||`) : Nos Superviseurs

Ils ne créent pas de processus. Ils gèrent le **temps**.

1. L'Executor appelle `exec_node(branche_gauche)`.

2. Il récupère le code de retour (`$?`).

3. Si c'est un `&&` et que la gauche a réussi (0) ➔ Il appelle `exec_node(branche_droite)`.

4. Si c'est un `||` et que la gauche a échoué (!= 0) ➔ Il appelle `exec_node(branche_droite)`.


#### B. Les nœuds pipes (`|`) : Nos Plombiers

Ils gèrent l'**espace**.

1. L'Executor crée un `pipe(fd)`.

2. Il fait un `fork()` pour l'enfant de GAUCHE ➔ Brache la sortie sur `fd[1]`, ferme `fd[0]`, et appelle `exec_node(branche_gauche)`.

3. Il fait un `fork()` pour l'enfant de DROITE ➔ Branche l'entrée sur `fd[0]`, ferme `fd[1]`, et appelle `exec_node(branche_droite)`.

4. Le parent ferme les deux bouts du pipe et fait deux `waitpid()`.


#### C. Les nœuds commandes (Les Feuilles) : Nos Ouvriers

C'est ici que toute notre action se passe vraiment.

1. **Redirections :** L'Executor parcourt la liste des redirections (`<`, `>`). Il ouvre les fichiers et utilise `dup2` pour remplacer l'entrée/sortie standard.

2. **[[_Builtins|Builtins]] ou Binaire ?** Il regarde le premier mot (`args[0]`).

    - Si c'est `cd`, `echo`, `exit`... ➔ Il lance notre fonction correspondante.

    - Si c'est `ls`, `cat`... ➔ Il cherche le chemin dans le `$PATH`.

3. **Exécution :** Il lance `execve()`. (S'il n'est pas déjà dans un processus enfant créé par un pipe, il doit faire un `fork()` juste avant !).

---

### 3. Les FDs qui fuient (J'en sais quelque chose xD)

C'est l'erreur numéro 1 sur pipex et donc, probablement sur minishell également.

**Le problème :** Un programme comme `cat` ou `grep` lit son entrée standard jusqu'à recevoir un signal "Fin de Fichier" (EOF). Ce signal n'est envoyé par le système que lorsque **TOUS les descripteurs d'écriture (`fd[1]`) pointant vers ce pipe sont fermés**.

Si le processus parent (notre minishell) oublie de fermer ses copies des FDs du pipe après avoir fait ses `fork`, le `cat` enfant va attendre éternellement, en pensant que quelqu'un pourrait encore écrire. Le minishell va "freeze".

**La Règle d'Or des Pipes :** Dès qu'un processus n'a plus besoin d'un bout de pipe, il **DOIT** le fermer avec `close()`. Le parent doit absolument fermer `fd[0]` et `fd[1]` juste après avoir lancé ses deux enfants et avant de faire `waitpid()`.

---

---

### La prochaine étape

L'Executor va se reposer sur deux gros piliers (du même genre que Pipex) :

1. **`exec_pipe_node`** : La gestion pure des tuyaux et des processus multiples.

2. **`exec_simple_cmd`** : La résolution du `$PATH` et l'application des redirections (`dup2`).

---

### 1. L'Exécution complèxe (`exec_pipe_node`)

Quand l'Executor tombe sur un nœud `NODE_PIPE`, il sait qu'il doit connecter la branche de **gauche** (qui produit du texte) à la branche de **droite** (qui lit ce texte).

Pour faire ça, il a besoin de 3 outils :

1. **`pipe(fd)`** : Crée un tuyau invisible dans le système avec deux bouts : `fd[0]` (la sortie pour lire) et `fd[1]` (l'entrée pour écrire).

2. **`fork()` x2** : Crée deux processus enfants pour exécuter les deux côtés en même temps (en parallèle).

3. **`dup2()`** : Débranche l'écran/clavier standard et le remplace par les bouts du tuyau.


---

### 2. L'Algorithme Pas-à-Pas

Voici l'histoire exacte de ce qui doit se passer dans notre fonction :

1. **Le Parent (minishell) fabrique le tuyau :** `pipe(fd)`.

2. **Le Parent clone un Enfant GAUCHE (`fork`) :**

    - _L'enfant Gauche se dit :_ "Je dois écrire dans le tuyau."

    - Il branche sa sortie standard (`STDOUT`) sur l'entrée du tuyau (`fd[1]`).

    - Il ferme le bout de lecture (`fd[0]`) car il ne s'en sert pas.

    - Il rappelle `exec_node(node->left)`.

    - Il fait un `exit()` avec le résultat.

3. **Le Parent clone un Enfant DROIT (`fork`) :**

    - _L'enfant Droit se dit :_ "Je dois lire depuis le tuyau."

    - Il branche son entrée standard (`STDIN`) sur la sortie du tuyau (`fd[0]`).

    - Il ferme le bout d'écriture (`fd[1]`) car il ne s'en sert pas.

    - Il rappelle `exec_node(node->right)`.

    - Il fait un `exit()` avec le résultat.

4. **Le Parent nettoie et attend :**

    - Le parent **FERME** ses propres copies de `fd[0]` et `fd[1]`. C'est vital !

    - Il fait deux `waitpid()` pour attendre que la gauche et la droite terminent.


---
### 3. Le code de retour ([[$?]])

On attend les deux enfants, mais on ne renvoie que le `status_right`.

**Pourquoi ?** C'est la norme POSIX. Si tu tapes `ls inexistant | wc -l` dans un vrai Bash :

1. `ls` va planter (code d'erreur `2`).

2. `wc` va compter 0 ligne et réussir (code de succès `0`).

3. Si tu tapes `echo $?` juste après, Bash affichera `0`.

Le pipe entier prend la valeur de succès ou d'échec de **la dernière commande de la chaîne**. C'est pour ça que notre `exec_pipe_node` renvoie le statut de l'enfant de droite !

---

---

# L'Exécution Simple (`exec_simple_cmd`)

### 1. Le Bout de la Chaîne

Quand ton Executor arrive sur un nœud `NODE_CMD`, il a devant lui une structure propre contenant deux choses :

1. Un tableau d'arguments (ex: `["ls", "-l", NULL]`).

2. Une liste de redirections (ex: `[OUT: "result.txt"]`).


À ce stade, l'Executor (qui est souvent déjà dans un processus enfant créé par le `pipe` précédent) doit préparer le terrain, trouver le programme, et s'autodétruire pour lui laisser la place (via `execve`).

---

### 2. L'Ordre des Opérations

Pour que ça marche comme dans le vrai Bash, on devra respecter cet ordre précis :

1. **Appliquer les Redirections :** Ouvrir les fichiers et brancher les tuyaux (`dup2`). Si un fichier n'existe pas ou n'a pas les droits, on s'arrête direct (erreur) (Si c'est pas une commande toute seule, par exemple avec un `coucou.txt < ls | echo "Coucou" > working.txt`, la commande de gauche ne marchera pas, mais celle de droite devra fonctionner .

2. **Vérifier les Builtins :** Est-ce que la commande est "interne" à notre minishell (ex: `cd`, `echo`, `exit`) ? Si oui, on lance notre propre code.

3. **Chercher dans le `$PATH` :** Si ce n'est pas un [[_Builtins|builtins]], c'est une fonction externe. On doit trouver son adresse complète.

4. **Lancer `execve` :** On écrase le processus actuel avec le nouveau programme.


---

#### 3. Étape A : Appliquer les Redirections

Il suffira de parcourir la liste chaînée `node->redirs`.

---

#### 4. Étape B : La Chasse au Trésor

Si l'utilisateur tape `/bin/ls`, c'est facile. Mais s'il tape juste `ls`, notre minishell doit fouiller dans la variable d'environnement `PATH`.

Le `$PATH` ressemble à ça : `/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin`

**Notre algorithme "Chercheur de Chemin" :**

1. On récupère la chaîne `$PATH` dans notre environnement.

2. On la découpe (`ft_split`) en utilisant `:` comme séparateur. On obtient un tableau de dossiers.

3. Pour chaque dossier, on colle un `/` puis le nom de notre commande (ex: `/usr/bin` + `/` + `ls`).

4. On teste si ce chemin complet existe et est exécutable avec la fonction `access()`.

5. Dès que `access()` dit "Oui !" (renvoie 0), on a trouvé !

---

### 5. Au final (`exec_simple_cmd`)

On rassemble tout. Attention, si on n'est pas dans un pipe, on doit **créer un enfant** (`fork`) juste pour lancer `execve`, sinon `execve` va remplacer notre minishell et le programme s'arrêtera ! (Mais normalement, on call notre fonction dans l'enfant pour vérifier, donc il y aura pas de problèmes ^.^)

---
