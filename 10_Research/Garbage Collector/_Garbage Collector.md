### Notre cauchemar pour minishell

Minishell est un programme qui tourne en boucle infinie. C'est le terrain de jeu parfait pour les fuites mémoires. Si on perds 10 octets à chaque commande, notre shell finira par planter au bout de quelques heures.

Voici les **4 grandes familles de fuites** qu'on devrait rencontré (ou pas si on fait les choses bien :D) :

#### A. Vous avez dit `readline` ?

La fonction `readline("minishell_from_stark_industries> ")` fait un `malloc` à chaque fois que l'utilisateur tape sur Entrée.

- **Le problème :** Si on ne fait pas `free(line)` à la fin de notre boucle `while`, on perdra cette string à chaque commande.

- **La règle :** Tout ce qui sort de `readline` doit être free avant de réafficher le prompt suivant.


#### B. Le Lexer & le Parser

Pour chaque commande tapée (ex: `ls -l | wc`), on va allouer :

1. Une liste chaînée de tokens ([[_Lexer|Lexer]]).

2. Un arbre [[AST]] ([[_Parser|Parser]]).

- **Le problème :** Une fois que la commande est exécutée, ces structures ne servent plus à rien.

- **La règle :** On devra coder une fonction qui va free nos ast (`free_ast(t_ast *node)`) et aussi free nos tokens (`free_tokens(t_token *list)`), qui nettoient tout _avant_ de repasser au début de la boucle.


#### C. L'Environnement : `export` et `unset`

Au début de notre programme, on va copier `envp` dans notre propre structure.

- **Le problème avec `export` :** Si on fait `export USER=nouveau`, on doit remplacer l'ancienne valeur. Si on oublie de faire un `free` sur l'ancienne string `"Jarvis"`, c'est un leak !

- **Le problème avec `unset` :** Si on supprime une variable, on doit libérer la clé (`USER`), la valeur (`Jarvis`), et aussi le maillon de la liste chaînée.


#### D. Le cimetière des enfants : Le `fork` (Pipex)

Quand on fait un `fork()`, le processus enfant hérite d'une copie exacte de la mémoire du parent.

- **Le problème :** Si notre enfant fait `exit()` (par exemple après avoir lancé un [[_Builtins|builtins]] dans un pipe), tout ce que le parent avait alloué (l'[[_Environnement|environnement]], l'[[AST]] en cours) est détruit par le système (donc techniquement géré par l'OS). **MAIS** Valgrind va quand même nous défoncer en disant que l'enfant a quitté sans faire de `free`.

- **La bonne pratique :** Juste avant qu'un processus enfant fasse `exit()`, on devra appeler notre [[_Garbage Collector|garbage collector]], pour que Valgrind soit content.