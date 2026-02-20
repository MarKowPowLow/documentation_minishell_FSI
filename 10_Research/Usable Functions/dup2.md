**`dup2`** : Duplique un FD vers un numéro précis (force la redirection).

```
int file = open("log.txt", O_WRONLY | O_CREAT, 0644);
// Remplace STDOUT (1) par 'file'. Tout printf ira dans le fichier.
dup2(file, STDOUT_FILENO); 
printf("Ceci est écrit dans le fichier log.txt\n");
```

