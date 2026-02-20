# Le Débuggage

Minishell est un projet énorme (Non 100 blagues ? À un franc la blague ça fait 100 Francs.). 
Si on code tout d'un coup et qu'on lance `./minishell_from_stark_industries`, ça va exploser en vol et on ne saura jamais pourquoi. Voici notre boîte à outils.

### 1. Les fonctions d'affichage

Avant même de coder l'[[_Executor|Executor]], on devra écrire des fonctions qui dessinent ce que fait notre programme. C'est du temps "perdu" au début qui vont nous sauver des semaines à la fin.

- **`print_tokens(t_token *list)` :** Affiche chaque token et son type.

    - _Utilité :_ On tape `ls -l | wc`. Si l'affichage montre `[CMD: ls -l]` au lieu de `[CMD: ls] -> [ARG: -l]`, on sait immédiatement que notre [[_Lexer|Lexer]] est cassé, inutile de chercher dans le [[_Parser|Parser]].

- **`print_ast(t_ast_node *node, int level)` :** Une fonction récursive qui affichera notre arbre avec des espaces pour l'indentation.

    - _Utilité :_ Permet de vérifier que les priorités (`|` vs `&&`) sont bien respectées.


### 2. Valgrind et le faux-positif `readline`

On va lancer notre projet avec : `valgrind --leak-check=full --show-leak-kinds=all ./minishell_from_stark_industries`

La fonction `readline` (qui vient d'une bibliothèque externe) a des fuites mémoires internes qu'on ne peut pas corriger. Valgrind va nous pourrir, rendant la lecture de nos _vraies_ fuites casse-...bonbons.

On va devoir créer un fichier texte (ex: `readline.supp`) pour dire à Valgrind de fermer les yeux sur `readline`.

```
{
    ignore_readline_leaks
    Memcheck:Leak
    ...
    fun:readline
}
{
    ignore_add_history_leaks
    Memcheck:Leak
    ...
    fun:add_history
}
```

_Pour le lancer :_ `valgrind --suppressions=readline.supp ./minishell_from_stark_industries

### 3. Les tests de l'extrême

J'ai trouvé 3 tests débiles qui sont apparamment bien connu, on va pouvoir check si on est nuls ou pas :D :

1. **Le test de l'environnement vide (`env -i`) :** On lance notre shell avec `env -i ./minishell_from_stark_industries`. Ça démarre notre programme sans **aucune** variable d'environnement (pas de `PATH`, pas de `USER`). _Notre shell ne doit pas segfault !_ Il doit recréer un `PWD` minimum et comprendre que les commandes externes comme `ls` ne marcheront plus (sauf si tapées avec le chemin absolu `/bin/ls`).

2. **La mitraillette à `Ctrl-C` :** On lance une commande bloquante (ex: `cat` sans arguments), puis spamme `Ctrl-C`, `Ctrl-\`, `Ctrl-D` dans tous les sens. Rien ne doit crasher, aucun processus zombie ne doit survivre.

3. **Le test des quotes vides :** On tape `echo "" '' ""`. Ça a l'air bête, mais ça casse 50% des Expanders mal codés. Ça doit juste afficher une ligne vide (ou des espaces), mais pas segfault !

Notre seigneur Eliot m'a fait part d'un test tout con mais qui peut casser des minishell mal fait aussi ! 

`cat | ls` puis `CTRL-C` cela donne `echo $?` = `0` (echo $? de ls ou de la dernière commande ex `cat | lol` = `127`, parce que lol n'existerait pas).


Faudra qu'on mettes à jour la liste avec d'autres tests à la con plus tard ! 