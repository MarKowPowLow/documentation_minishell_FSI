**`perror`** : Affiche l'erreur système actuelle sur STDERR.

```
int fd = open("inexistant", O_RDONLY);
if (fd == -1)
    perror("minishell_from_stark_industries"); // Affiche : "minishell_from_stark_industries: No such file or directory"
```