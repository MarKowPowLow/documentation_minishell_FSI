**`wait3`** : Attend n'importe quel enfant qui meurt et récupère ses stats (RAM/CPU) (Je ne pense pas que sa servira pour minishell)

```
struct rusage usage;
int status;
// Le 2ème arg est pour les options (0 = bloquant)
// Remplira 'usage' avec les infos de consommation
wait3(&status, 0, &usage); 
printf("Temps CPU utilisé : %ld sec\n", usage.ru_utime.tv_sec);
```