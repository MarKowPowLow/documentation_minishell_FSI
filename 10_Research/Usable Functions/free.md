**`free`** : Libère la mémoire.

```
char *str = malloc(10);
// ...
free(str); // Indispensable pour éviter les leaks
```