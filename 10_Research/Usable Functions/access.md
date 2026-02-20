**`access`** : Vérifie les permissions d'un fichier.

```
// F_OK = Existe ?, X_OK = Exécutable ?, R_OK = Lisible ?, W_OK = Écrivable ?
if (access("/bin/ls", X_OK) == 0)
    printf("Je peux exécuter ls !\n");
else
    printf("Je ne peux pas (ou il n'existe pas).\n");
```
