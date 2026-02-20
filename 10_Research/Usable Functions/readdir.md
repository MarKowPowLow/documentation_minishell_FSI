**`readdir`** : Lit le prochain fichier dans un dossier ouvert.

```
struct dirent *entry;
while ((entry = readdir(d)) != NULL)
    printf("Fichier : %s\n", entry->d_name);
```