**`chdir`** : Change le dossier courant (la commande `cd`).

```
if (chdir("/tmp") == 0)
    printf("Je suis maintenant dans /tmp\n");
else
    perror("Erreur cd");
```

