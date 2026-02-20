**`fork`** : Clone le processus.

```
pid_t pid = fork();
if (pid == 0)
    printf("Je suis l'enfant !\n");
else
    printf("Je suis le parent, mon enfant a le PID %d\n", pid);
```