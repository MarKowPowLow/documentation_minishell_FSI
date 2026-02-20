**`rl_replace_line`** : Change le contenu de la ligne que l'utilisateur est en train de taper (mais n'a pas encore validé).

```
// Vide la ligne en cours (utile après un Ctrl-C)
rl_replace_line("", 0); 
```