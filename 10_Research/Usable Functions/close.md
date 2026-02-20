**`close`** : Ferme un file descriptor.

```
int fd = open("test.txt", O_RDONLY);
// ... on s'en sert ...
close(fd); // Toujours fermer pour éviter les fuites !
```