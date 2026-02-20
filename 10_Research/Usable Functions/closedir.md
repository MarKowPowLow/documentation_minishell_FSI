**`closedir`** : Ferme un dossier ouvert (voir `opendir`).

```
DIR *dir = opendir(".");
// ... on lit ...
closedir(dir);
```