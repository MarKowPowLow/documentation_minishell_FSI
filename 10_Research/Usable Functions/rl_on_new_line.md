**`rl_on_new_line`** : Dit à readline "Fais comme si on était sur une nouvelle ligne".

```
// Utilisé dans le signal handler Ctrl-C
printf("\n");
rl_on_new_line();
rl_replace_line("", 0);
rl_redisplay();
```