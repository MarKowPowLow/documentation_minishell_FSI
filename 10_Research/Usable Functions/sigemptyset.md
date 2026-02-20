**`sigemptyset`** : Manipule les masques de signaux.

```
sigset_t set;
sigemptyset(&set); // Initialise vide
sigaddset(&set, SIGQUIT); // Ajoute Ctrl-\ au set
```