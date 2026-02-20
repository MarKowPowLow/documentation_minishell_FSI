#### `pwd` (Le Localisateur)

- **Ce qu'il fait :** Affiche le chemin absolu du dossier courant (avec un `\n`).

- **Les pièges à prévoir :**

    - Si `getcwd()` échoue, chercher la valeur de `$PWD` dans ton environnement et l'afficher à la place.

    - _Cas extrême :_ Si un autre terminal supprime le dossier dans lequel on se trouve, `getcwd()` peut renvoyer NULL. Dans ce cas, notre minishell va lire la variable `$PWD` de son environnement pour s'en sortir.

    - _Sécurité :_ Si `getcwd()` échoue, chercher la valeur de `$PWD` dans l'environnement et l'afficher à la place.

- **Algorithme :**
    
    1. `char *path = getcwd(NULL, 0);`
        
    2. Si `path` est valide, l'afficher. Sinon, lire l'env.

---
