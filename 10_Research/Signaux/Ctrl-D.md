### `Ctrl-D` : L'Imposteur (EOF)

Attention : Ce n'est PAS un signal !

Nom technique : `EOF` (End Of File).

Imagine que ton entrée standard (stdin) est un fichier. `Ctrl-D`, c'est le caractère invisible qui dit "C'est la fin du fichier".

- Comment le détecter ? Tu ne peux pas utiliser `signal()` ou `sigaction` pour lui.

- Comment on va faire ? :

    - Sur le prompt : La fonction `readline()` va essayer de lire ce que tu tapes. Si tu fais `Ctrl-D` sur une ligne vide, `readline` ne renvoie pas une string vide `""`, elle renvoie **`NULL`**.

    - Action : Si `readline` renvoie `NULL`, ça veut dire que l'utilisateur veut partir. Il faudra "exit", libérer la mémoire et quitter proprement le Minishell.

    - Dans une commande (ex: `cat`) : Ça ferme l'entrée standard de `cat`, donc `cat` s'arrête proprement (il croit que le fichier est fini).