**`add_history`** : Ajoute une ligne à l'historique (flèche haut).

```
char *cmd = readline("minishell_from_stark_industries> ");
if (cmd && *cmd)
    add_history(cmd); // Maintenant, flèche haut rappellera cette commande
```
