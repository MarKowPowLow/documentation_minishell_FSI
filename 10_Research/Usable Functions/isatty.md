**`isatty`** : Vérifie si un FD est un terminal (et pas un fichier ou un pipe).

```
if (isatty(STDIN_FILENO))
    printf("Mode interactif (clavier)\n");
else
    printf("Mode script (pipe ou redirection)\n");
```