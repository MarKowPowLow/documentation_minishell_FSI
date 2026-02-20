**`pipe`** : Crée un tube (2 FDs).

```
int fd[2];
pipe(fd);
// fd[1] pour écrire, fd[0] pour lire.
write(fd[1], "hello", 5);
read(fd[0], buffer, 5);
```