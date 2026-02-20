**`open`** : Ouvre un fichier.

```
// O_CREAT : Créer si existe pas. O_TRUNC : Vider si existe.
int fd = open("out.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
```