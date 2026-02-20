**`tgetent`**, **`tgetflag`**, **`tgetnum`**, **`tgetstr`**, **`tgoto`**, **`tputs`** : Ce sont les fonctions de la librairie **termcap** (gestion du curseur). _Note : Elles fonctionnent ensemble._

```
char buf[1024];
tgetent(buf, getenv("TERM")); // 1. Initialise
char *move = tgetstr("cm", NULL); // 2. Récupère le code "cursor move"
char *cmd = tgoto(move, 10, 5); // 3. Prépare le déplacement en x=10, y=5
tputs(cmd, 1, putchar); // 4. Exécute
```