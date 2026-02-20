**`stat`** : Récupère les infos d'un fichier par son chemin.

```
struct stat s;
stat("/bin/ls", &s);
// s.st_mode contient les permissions
```