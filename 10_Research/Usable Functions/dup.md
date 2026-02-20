**`dup`** : Duplique un FD vers le premier numéro libre.

```
// fd_copy sera une copie de STDOUT (1), probablement 3
int fd_copy = dup(STDOUT_FILENO); 
write(fd_copy, "Coucou via la copie\n", 20);
```
