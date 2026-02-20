**`getcwd`** : Récupère le dossier actuel (la commande `pwd`).

```
char buf[1024];
if (getcwd(buf, sizeof(buf)) != NULL)
    printf("PWD actuel : %s\n", buf);
```