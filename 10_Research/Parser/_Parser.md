# Le Parser (L'Analyseur Syntaxique)

### 1. Qu'est-ce qu'un Parser ?

Si on reprend notre métaphore précédente :

- **Notre Lexer** a fabriqué des briques de lego individuelles (nos tokens : `[ls]`, `[|]`, `[wc]`). Il a juste mis une étiquette sur chaque brique.

- **Notre Parser**, c'est l'architecte. Il prend la notice de montage (la grammaire du Shell) et assemble ces briques pour construire une maison.

L'objectif du Parser est de transformer notre liste chaînée de tokens (qui est plate, en 1D) en un Arbre Syntaxique Abstrait ([[AST]]) (qui a une hiérarchie, en 2D).

Tu vas me dire, mais pourquoi faire ? Parce que l'ordinateur ne peut pas exécuter `ls | wc` de gauche à droite bêtement. Il doit comprendre que le symbole `|` est le "chef" qui relie `ls` (à sa gauche) et `wc` (à sa droite). 
L'arbre permet de donner un ordre de priorité.

---

### 2. Les priorités

Pour construire cet arbre, le Parser cherche les symboles dans un ordre bien précis, du **moins prioritaire au plus prioritaire**. _(Rappel : Le moins prioritaire finit tout en haut de l'arbre, car c'est lui qui divise le plus la commande)._

Voici l'ordre de recherche de notre Parser :

1. **Les Opérateurs Logiques (`&&`, `||`) :** Ce sont les grands chefs. Ils divisent la ligne en gros blocs conditionnels.

2. **Les Pipes (`|`) :** Ils lient les processus entre eux.

3. **Les Commandes et Redirections (`ls`, `cat`, `>`, `<`) :** Ce sont les ouvriers (les feuilles de l'arbre). Une commande garde toujours ses redirections et ses arguments avec elle.

---

### 3. L'Exemple visuel

Imaginons la commande : `cat < file.txt | grep "erreur" && echo "Fini"`

**1. Ce que donne le Lexer :** `[cat]` -> `[<]` -> `[file.txt]` -> `[|]` -> `[grep]` -> `["erreur"]]` -> `[&&]` -> `[echo]` -> `["Fini"]`

**2. Ce que fait le Parser :**

- Il scanne la liste et cherche le grand chef : _Y a-t-il un `&&` ou `||` ?_ -> **OUI**.

- Il coupe la liste en deux autour du `&&`. Le `&&` devient la **Racine**.

- Il regarde à gauche du `&&` : `cat < file.txt | grep "erreur"`. _Y a-t-il un Pipe ?_ -> **OUI**.

- Il coupe autour du `|`. Le `|` devient le nœud enfant de gauche.

- Il regarde ce qu'il reste : ce sont de simples commandes avec leurs arguments et redirections. Ce seront les feuilles de l'arbre.


**3. Le Résultat (L'Arbre AST) :**

```
                  [ AND (&&) ]
                 /            \
          [ PIPE (|) ]       [ CMD: echo "Fini" ]
          /          \
[ CMD: cat ]        [ CMD: grep "erreur" ]
[ IN : file.txt]
```

---

### 4. À quoi ça ressemble ?

Pour coder ça, on va utiliser de la récursivité et des structures qui peuvent pointer vers d'autres structures de la même famille (et aussi une énumération ^^), dans ce genre là :

```
typedef enum e_node_type {
    NODE_CMD,
    NODE_PIPE,
    NODE_AND,
    NODE_OR
} t_node_type;

typedef struct s_ast_node {
    t_node_type         type;
    
    // Si c'est un nœud de contrôle (PIPE, AND, OR), on utilise les branches
    struct s_ast_node   *left;
    struct s_ast_node   *right;
    
    // Si c'est un nœud CMD, c'est une feuille :D :
    char                **args;       // ex: ["ls", "-l", NULL]
    t_redir             *redirections;// Liste chaînée des redirections (<, >)
} t_ast_node;
```

---

### 5. Comment le Parser va gérer les Redirections ?

Pour gérer les redirections dans notre parser, il faudra qu'on le lie avec une commande. Même si l'utilisateur tape `< Makefile cat > out.txt`, le Parser doit être intelligent et regrouper tout ça dans un seul nœud `NODE_CMD` :

- `args` = `["cat", NULL]`

- `redirections` = `[Type: IN, Fichier: "Makefile"] -> [Type: OUT, Fichier: "out.txt"]`


Quand on construit un nœud `NODE_CMD`, on parcourt les tokens restants. Tout ce qui n'est pas une redirection devient un argument. Dès qu'on voit une redirection, on prend le token suivant (le nom du fichier) et on l'ajoute à la liste des redirections de cette commande.

---

Une fois que notre première logique a découpé la ligne sur les `&&`, `||` et `|`, elle finit par isoler des "blocs" qui ne contiennent plus que du texte et des redirections. C'est à ce moment-là qu'intervient un deuxième check.

Voici comment ça se passe concrètement quand notre parser décide de créer un nœud feuille (`NODE_CMD`).

### 1. Le but du deuxième check

Le Parser a isolé ce sous-groupe de tokens pour une commande : `[<] -> [entree.txt] -> [cat] -> [-e] -> [>] -> [sortie.txt]`

Le but de notre fonction (probablement un truc du genre `parse_cmd(t_token *sub_list)`) est de trier ces éléments dans deux boîtes différentes à l'intérieur de notre `t_ast_node` :

1. **La boîte `args`** (un `char **` pour [[execve]]) : `["cat", "-e", NULL]`

2. **La boîte `redirections`** (une liste chaînée) : `[IN: entree.txt] -> [OUT: sortie.txt]`


### 2. L'Algorithme de tri

C'est une boucle `while` très classique qui parcourt la sous-liste de tokens. La règle est simple : **Tout ce qui n'est pas une redirection (ou la cible d'une redirection) est un argument.**

Voici la logique pas à pas :

1. Je lis le token actuel.

2. **Cas A : C'est un chevron (`<`, `>`, `<<`, `>>`)**

    - Je regarde le token _suivant_ (c'est forcément le nom du fichier).

    - Je crée un maillon de redirection avec le type et le fichier.

    - J'ajoute ce maillon à la liste `node->redirections`.

    - Il ne faut surtout pas oublié d'avancer de **2 cases** dans la liste de tokens (pour sauter le chevron ET le fichier).

3. **Cas B : C'est un mot normal (`TOKEN_WORD`)**

    - Je l'ajoute à mon tableau `node->args`.

    - J'avance de **1 case**.


### 3. L'Ordre ?!

Pourquoi est-ce qu'on doit faire ce tri dynamique au lieu de juste supposer que la commande est au début ? Parce que dans Bash (et donc dans notre minishell), **l'utilisateur a le droit d'écrire n'importe comment !**

Ces trois commandes sont strictement identiques pour Bash :

- `cat -e < in.txt > out.txt`

- `< in.txt cat > out.txt -e`

- `> out.txt < in.txt cat -e`


Si notre algorithme applique le tri décrit au-dessus, il n'aura aucun problème avec ça. Il va picorer les mots un par un :

- Un `<` ? Je prends `in.txt` et je les mets dans la boîte des redirections.

- Le mot `cat` ? Je le mets dans la boîte des arguments.

- Un `>` ? Je prends `out.txt` pour les redirections.

- Le mot `-e` ? Je le mets dans les arguments.


À la fin, on aura bien `args = ["cat", "-e", NULL]`. L'ordre des arguments est conservé, et l'ordre des redirections aussi.

---

### Résumé de notre architecture du parser :

1. **Niveau 1 :** `parse_line()` coupe sur `&&` / `||`.

2. **Niveau 2 :** `parse_pipe()` coupe sur les `|`.

3. **Niveau 3 :** `parse_cmd()` trie les mots et les chevrons d'un bloc isolé.

---

On va voir un peu comment fonctionne et à quoi devrait ressembler notre boîte de tri.

### 1. Les structures

Pour que la commande finale soit bien rangée, on a besoin d'une liste chaînée juste pour les redirections, qui sera stockée à l'intérieur de notre nœud d'arbre ([[AST]]) et notre énumération pour savoir quel type de redirection nous a été donné :

```
typedef enum e_redir_type {
    REDIR_IN,       // <
    REDIR_OUT,      // >
    REDIR_APPEND,   // >>
    REDIR_HEREDOC   // <<
} t_redir_type;

typedef struct s_redir {
    t_redir_type    type;
    char            *file;
    struct s_redir  *next;
} t_redir;

typedef struct s_cmd_node {
    char            **args;
    t_redir         *redirs;
} t_cmd_node;
```

---

### 2. L'Algorithme de tri

Pour parcourir notre sous-liste de tokens (qui ne contient plus de `|` ni de `&&`, juste les mots et les chevrons de la commande actuelle), on va probablement passer dans une fonction dans ce genre là  :

```
t_cmd_node *parse_cmd(t_token **sub_list)
{
	t_redir_type type;
	char *file;
    t_cmd_node  *cmd;

    cmd = malloc(sizeof(t_cmd_node))
    cmd->redirs = NULL;
	// On boucle tant que notre sub_list existe et que c'est pas un opérateur de contrôle (&& & |)
    while (*sub_list && !is_control_operator(*sub_list))
    {
        if (is_redirection(*sub_list))
        {
            type = get_redir_type(*sub_list);
            if (!(*sub_list)->next)
	            return (error !?!) // Ça voudrait dire qu'il y a un chevron sans rien derrière !
            *sub_list = (*sub_list)->next; 
            file = (*sub_list)->value;
            add_redir_to_list(&(cmd->redirs), type, file);
        }
        else
        {
            add_arg_to_array(&(cmd->args), (*sub_list)->value);
        }
        if ((*sub_list)->next)
	        *sub_list = (*sub_list)->next;
    }
    return (cmd);
}
```

---

### 3. Pour notre tableau `args`

Dans mon exemple au-dessus, j'ai mis une fonction `add_arg_to_array`. **Le problème**, c'est que pour créer un `char **args`, on doit connaître à l'avance le nombre d'arguments pour faire notre `malloc`. Mais quand on tri notre liste, on ne sait pas encore combien il y en a !
Comment faire alors ? :

1. Au lieu de remplir un `char **` directement, on stocke temporairement tes arguments dans une **liste chaînée classique** (ex: `t_list *temp_args`).

2. À la fin de notre `while` (quand on a fini de lire la commande), on compte combien de maillons il y a dans `temp_args`.

3. On fait un seul gros `malloc` pour ton `char **` de la bonne taille.

4. On copie les mots de la liste vers le tableau.

5. On libère la liste temporaire.


C'est beaucoup plus propre et ça évitera de faire des reallocs et donc, plus d'erreurs possibles xD.

---

### 4. Ce que ça donne à l'exécution

Si l'utilisateur tape : `> log.txt ls -l -a < in.txt`

1. Ton code voit `>` : Il crée un maillon `[OUT | "log.txt"]`. Il avance de 2.

2. Ton code voit `ls` : Il le met dans la liste temporaire des args. Il avance de 1.

3. Ton code voit `-l` : Pareil.

4. Ton code voit `-a` : Pareil.

5. Ton code voit `<` : Il crée un maillon `[IN | "in.txt"]`. Il avance de 2.


**Résultat final dans le nœud :**

- `cmd->args` = `["ls", "-l", "-a", NULL]`

- `cmd->redirs` = `[OUT: "log.txt"] -> [IN: "in.txt"] -> NULL`

Ça sera parfait pour notre [[_Executor|Executor]] ! Il n'aura plus qu'à ouvrir les fichiers de la liste `redirs` avec `dup2`, puis balancer le tableau `args` dans `execve`.

---
---

### On va passer au concept du Heredoc : 

Lire en boucle, stocker, et s'arrêter quand on croise le mot magique (le délimiteur).

### 1. Le Concept de Base

Quand ton Parser croise un token `<<` suivi d'un `LIMITER` (par exemple `EOF`) :

1. Il doit mettre son travail de parsing en pause.

2. Il ouvre un prompt secondaire (souvent `>` dans le vrai bash, donc on utilisera sûrement `readline("> ")`).

3. Il lit l'entrée de l'utilisateur ligne par ligne.

4. Il s'arrête **uniquement** quand la ligne tapée est _exactement_ égale au `LIMITER` (sans espaces avant ni après).

---

### 2. Où stocker ce qu'on lit 

C'est la grande question. Pour pipex, je n'avais pas à le gérer, vu que Bash le faisait pour moi, mais là, c'est autre chose !

#### Méthode 1 : Le Pipe

Tu crées un `pipe()`, tu écris tout dedans, et tu gardes le côté "lecture" pour ta commande.

- C'est pas dingue de faire ça, un pipe a une taille limite (souvent 64Ko). Si l'utilisateur colle un texte énorme dans le Heredoc, le programme va bloquer (deadlock) pendant l'écriture.


#### Méthode 2 : Le Fichier Temporaire

Ça sera plus simple à coder :

1. Quand tu rencontres `<<`, tu ouvres un vrai fichier caché avec `open("/tmp/.minishell_from_stark_industries_heredoc", O_CREAT | O_WRONLY | O_TRUNC, 0644)`.

2. Dans ta boucle `readline`, tu écris chaque ligne (suivie d'un `\n`) dans ce fichier.

3. Quand tu croises le limiteur, tu fermes le fichier (`close`).

4. Ce qui est bien, c'est que dans notre nœud d'Arbre ([[AST]]), on va transformer discrètement cette redirection en une **redirection d'entrée classique (`<`)** qui pointe vers `/tmp/.minishell_from_stark_industries_heredoc`.

5. Notre [[_Executor|Executor]] (qui viendra plus tard) n'y verra que du feu. Il fera juste un `open` en lecture sur ce fichier !

6. Par contre, il ne faudra pas oublié d'utiliser `unlink("/tmp/.minishell_from_stark_industries_heredoc")` à la toute fin de notre commande pour clean correctement le fichier (et éviter des problèmes :D).

---

### 3. L'Expansion des Variables

Dans un Heredoc, le comportement du `$VAR` dépend de **comment le délimiteur a été écrit**.

**Cas A : Le délimiteur est "nu" (`<< EOF`)**

- Bash considère que le Heredoc est "ouvert".

- **Règle :** Tu **DOIS** envoyer chaque ligne lue à ton [[_Expander|Expander]] avant de l'écrire dans le fichier.

- _Exemple :_ Si je tape `Bonjour $USER`, ça écrira `Bonjour Jarvis` dans le fichier.


**Cas B : Le délimiteur a des quotes (`<< "EOF"` ou `<< 'EOF'`)**

- Bash considère que le Heredoc est "verrouillé".

- **Règle :** Tu **NE DOIS PAS** étendre les variables. Tu écris le texte brut.

- _Exemple :_ Si je tape `Bonjour $USER`, ça écrira `Bonjour $USER` dans le fichier.

- _(Note de parsing : Le délimiteur qui arrêtera la boucle sera bien `EOF`, sans les quotes)._

---

### 4. Le Timing

Les Heredocs se lisent **pendant la phase de Parsing** (ou juste après, dans une phase de "Pré-exécution" dans le processus parent), _avant_ de faire le moindre `fork()`. Il faudra parcourir notre arbre, repérer tous les `<<`, faire des `readline`, remplir nos fichiers temporaires, et **ensuite seulement**, on lancera l'exécution globale.

---
