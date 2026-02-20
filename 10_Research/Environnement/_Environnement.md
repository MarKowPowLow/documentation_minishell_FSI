# L'Environnement

### 1. Pourquoi on ne garde pas le `envp` du `main` ?

Quand on déclare notre `main(int argc, char **argv, char **envp)`, le système nous donnes `envp`, un tableau de chaînes de caractères (ex: `USER=Jarvis`, `PATH=/bin...`).

Ce tableau a une taille fixe allouée par le système. Si l'utilisateur tape `export NOUVELLE_VAR=123`, on ne peut pas juste ajouter une ligne à la fin de ce `envp`, il n'y a pas de place ! Si on essai, c'est le `segfault` direct.

Dès la première ligne de notre `main`, on **DEVRA** copier entièrement ce `envp` dans notre propre structure de données dynamique, puis oublier le `envp` d'origine.

### 2. La Liste Chaînée

Pour manipuler l'environnement (ajouter, modifier, supprimer des variables), un tableau en `char **` est un enfer (il faudrait faire des `malloc`/`free` géants à chaque modification). 
Cependant ! La **liste chaînée** est parfaite pour ça.

```
typedef struct s_env {
    char            *key;   // Ex: "USER"
    char            *value; // Ex: "Jarvis" (Peut être NULL si export sans '=')
    struct s_env    *next;
} t_env;
```

Séparer la `key` et la `value` dès le début sera plus intéressant, ça évitera des problèmes pour nos mallocs plus tard, mais aussi pour l'[[_Expander|Expander]] (`$USER`) et pour les [[_Builtins|builtins]] `export`/`unset`.

### 3. Les variables "spéciales" à gérer au démarrage

Quand notre minishell démarre, il doit faire le ménage dans l'environnement qu'il vient de copier :

1. **`SHLVL` (Shell Level) :** C'est le niveau d'imbrication de notre shell. On doit trouver cette variable, prendre sa valeur (ex: `1`), faire `+1`, et la remettre à jour (`SHLVL=2`).

2. **`PWD` :** S'il n'existe pas, on doit le créer en utilisant `getcwd()`.

3. **`OLDPWD` :** Le comportement standard de Bash au démarrage est souvent de le supprimer (ou de le vider) pour que le premier `cd -` échoue.

---
