**`rl_redisplay`** : Force readline à réafficher le prompt et la ligne actuelle (Même exemple que pour rl_on_new_line).

```
// Utilisé dans le signal handler Ctrl-C
printf("\n");
rl_on_new_line();
rl_replace_line("", 0);
rl_redisplay();
```