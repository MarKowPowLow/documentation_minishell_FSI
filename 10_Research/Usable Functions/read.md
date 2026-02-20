**`read`** : Lit depuis un FD.

```
char buf[100];
int lu = read(fd, buf, 99);
buf[lu] = '\0'; // Toujours terminer la string manuellement !
```