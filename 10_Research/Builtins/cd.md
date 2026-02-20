#### `cd` (Le Voyageur)

- **Ce qu'il fait :** Change le dossier courant avec la fonction système `chdir()` et mets à jour notre l'environnement (`PWD` et `OLDPWD`).

- **Les pièges à prévoir :**

    - `cd` tout court (sans argument) doit te ramener au dossier de la variable `$HOME`. Si `$HOME` n'est pas définie (supprimée par l'utilisateur), il faut renvoyer une erreur `minishell_from_stark_industries: cd: HOME not set`.

    - Gérer l'erreur si le dossier n'existe pas ou si les droits manquent (`perror`) et code erreur `1`.

	- `cd -` retourne à `$OLDPWD`.

- **Algorithme :**
    
    1. Sauvegarder le chemin actuel : `old_path = getcwd()`.

    2. Faire `chdir(cible)`. Si échec, afficher erreur et return `1`.

    3. Modifier la variable `OLDPWD` dans l'env avec `old_path`.

    4. Récupérer le nouveau chemin : `new_path = getcwd()`.

    5. Modifier la variable `PWD` dans l'env avec `new_path`.

---
