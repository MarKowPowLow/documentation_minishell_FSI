# Les Builtins (Les Commandes Internes)

### 1. Le Piège Fondamental : Parent vs Enfant

Avant de lister nos commandes, il faut comprendre **pourquoi** elles existent. Pourquoi recoder `cd` alors que `/usr/bin/cd` (ou équivalent) pourrait exister ?

Parce qu'un programme externe ne peut pas modifier l'état de notre minishell !
(Hé oui Jamy !)

- Si tu fais un `fork()` et que l'enfant exécute `cd /tmp`, l'enfant change de dossier... puis il meurt. Notre minishell (le parent) n'a pas bougé !

- Les builtins qui modifient l'état du shell (`cd`, `export`, `unset`, `exit`) **DOIVENT être exécutés dans le processus parent**. On ne fait pas de `fork()` pour eux (sauf s'ils sont dans un pipe, ex: `ls | cd ..`, auquel cas ils n'affectent que le sous-shell du pipe).

---

### 2. Le Récapitulatif des 7 Builtins

Voici ce qu'on doit prévoir pour chacun d'entre eux.

#### 1. `echo` 

- **Ce qu'il fait :** Affiche ses arguments séparés par un espace, suivi d'un retour à la ligne (`\n`).

- **L'option à gérer :** `-n` (ne pas afficher le `\n` final).

- **Les pièges à prévoir :**

    - Gérer les `-n` multiples combinés : `echo -n -n -nnnnn Salut` doit fonctionner comme un seul `-n`.

    - Pas besoin de se prendre la tête avec les espaces, notre lexer fera le tri ! (`echo a b`). On recevra juste le tableau `["echo", "a", "b", NULL]`. Il y aura juste à faire une boucle d'affichage avec un espace entre chaque.


#### 2. `cd` 

- **Ce qu'il fait :** Change le dossier courant avec la fonction système `chdir()`.

- **Les pièges à prévoir :**

    - **Le vrai boulot de `cd` :** Ce n'est pas juste de faire `chdir()`. On doit mettre à jour les variables d'environnement `PWD` (dossier actuel) et `OLDPWD` (dossier précédent) dans la liste chaînée !

    - `cd` tout court (sans argument) doit te ramener au dossier de la variable `$HOME`.

    - Gérer l'erreur si le dossier n'existe pas ou si les droits manquent (`perror`).


#### 3. `pwd` 

- **Ce qu'il fait :** Affiche le chemin absolu du dossier courant.

- **Les pièges à prévoir :**

    - Très simple en apparence : un coup de `getcwd()` et on l'affiche.

    - _Cas extrême :_ Si un autre terminal supprime le dossier dans lequel on se trouve, `getcwd()` peut renvoyer NULL. Dans ce cas, notre minishell va lire la variable `$PWD` de son environnement pour s'en sortir.


#### 4. `export` 

- **Ce qu'il fait :** Ajoute ou modifie une variable dans notre environnement.

- **Les pièges à prévoir :**

    - **La syntaxe stricte :** Une clé de variable ne peut contenir que des lettres, des chiffres et des underscores (`_`), et ne **peut pas** commencer par un chiffre. `export 1VAR=bonjour` doit renvoyer une erreur (`not a valid identifier`).

    - **Cas sans valeur :** `export VAR` (sans `=`) crée la variable mais elle n'a pas de valeur. Elle existe quand même !

    - **`export` tout court :** S'il n'y a pas d'arguments, il doit afficher tout l'environnement **trié par ordre alphabétique** avec le préfixe `declare -x`. (Ex: `declare -x USER="Jarvis"`). C'est souvent la fonction la plus longue à coder des builtins !


#### 5. `unset` 

- **Ce qu'il fait :** Supprime une variable de l'environnement.

- **Les pièges à prévoir :**

    - C'est ici qu'on va tester la solidité de notre liste chaînée. On doit reconnecter le maillon précédent au maillon suivant, et bien faire un `free()` de la clé, de la valeur, et du nœud.

    - Si la variable n'existe pas, `unset` ne fait rien (et ne renvoie pas d'erreur).

    - Attention à la même syntaxe stricte que `export` pour le nom de la variable.


#### 6. `env` 

- **Ce qu'il fait :** Affiche la liste des variables d'environnement.

- **Les pièges à prévoir :**

    - Contrairement à `export` sans argument, `env` ne trie pas la liste.

    - `env` n'affiche **que** les variables qui ont un `=` (une valeur). Celles créées par `export VAR` sans valeur ne doivent pas apparaître ici.

    - Le sujet demande `env` sans options ni arguments. S'il y a des arguments, On peut juste renvoyer une erreur ou l'ignorer selon comment on va calquer le bash d'origine.


#### 7. `exit` 

- **Ce qu'il fait :** Quitte Minishell avec un code de retour spécifique.

- **Les pièges à prévoir :**

    - **Le ménage :** Avant d'appeler la fonction `exit()`, on DOIT appeler notre "[[_Garbage Collector|Garbage Collector]]" (`free_all()`) pour libérer notre [[_Environnement|environnement]], l'[[AST]], etc...

    - **L'argument numérique :** `exit 42` doit quitter avec le code `42`.

    - **Trop d'arguments :** `exit 1 2` doit afficher l'erreur `minishell_from_stark_industries: exit: too many arguments` et **ne pas** quitter.

    - **Argument non numérique :** `exit lol` doit afficher `minishell_from_stark_industries: exit: lol: numeric argument required` et quitter avec le code `2`.


---