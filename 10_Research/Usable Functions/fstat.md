**`fstat`** : Infos sur un fichier via son FD (pas son nom).

```
struct stat info;
int fd = open("main.c", O_RDONLY);
fstat(fd, &info); // On remplit la structure
printf("Taille du fichier : %lld\n", info.st_size);
```