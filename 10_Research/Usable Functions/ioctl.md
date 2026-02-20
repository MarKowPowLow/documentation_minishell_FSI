**`ioctl`** : Contrôle les périphériques (souvent utilisé pour la taille de fenêtre).

```
struct winsize w;
// TIOCGWINSZ = Terminal IOCtl Get WINdow SiZe
ioctl(STDOUT_FILENO, TIOCGWINSZ, &w);
printf("Lignes: %d, Colonnes: %d\n", w.ws_row, w.ws_col);
```