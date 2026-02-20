**`lstat`** : Comme `stat`, mais si c'est un lien symbolique, donne les infos du lien (pas de la cible).

```
struct stat info;
lstat("mon_lien_symbolique", &info);
if (S_ISLNK(info.st_mode))
    printf("C'est bien un lien !\n");
```