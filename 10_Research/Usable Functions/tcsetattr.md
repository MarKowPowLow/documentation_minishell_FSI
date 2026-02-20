**`tcsetattr`** : Modifie la config du terminal.

```
struct termios term;
tcgetattr(0, &term); // Lit la config
term.c_lflag &= ~ECHOCTL; // Désactive l'affichage du "^C"
tcsetattr(0, TCSANOW, &term); // Applique
```