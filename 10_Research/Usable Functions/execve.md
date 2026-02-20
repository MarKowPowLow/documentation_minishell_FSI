**`execve`** : Remplace le processus actuel par un nouveau programme.

```
char *args[] = {"/bin/ls", "-l", NULL}; // Tableau terminant par NULL
char *env[] = {NULL}; // Environnement vide ou copie de l'env
// Si ça marche, le code s'arrête ici net.
execve("/bin/ls", args, env);
perror("Si je m'affiche, c'est que execve a échoué");
```