**`wait4`** : Attend un enfant spécifique (PID) et récupère ses stats (RAM/CPU) (Je ne pense pas que sa servira pour minishell)

```
struct rusage usage;
int status;
// Comme waitpid, mais avec le 4ème argument pour les stats
wait4(pid_enfant, &status, 0, &usage);
printf("Mémoire max utilisée : %ld\n", usage.ru_maxrss);
```