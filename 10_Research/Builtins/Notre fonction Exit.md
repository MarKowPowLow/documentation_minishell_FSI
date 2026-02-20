#### `exit` (La Porte de Sortie)

- **Ce qu'il fait :** Quitte minishell avec un code de retour spécifique.

- **Comportement attendu :**

    - Affiche "exit" sur la sortie standard (ou erreur standard).

    - Ferme le shell avec le code demandé (modulo 256). Ex: `exit 42` ➔ code `42`. `exit 258` ➔ code `2` (car 258 % 256 = 2).

- **Les pièges à prévoir :**

    - **Le ménage :** Avant d'appeler la fonction `exit()`, on DOIT appeler notre "[[_Garbage Collector|Garbage Collector]]" (`free_all()`) pour libérer notre [[_Environnement|environnement]], l'[[AST]], etc...

    - **L'argument numérique :** `exit 42` doit quitter avec le code 42.

    - **Trop d'arguments :** `exit 1 2` doit afficher l'erreur `minishell_from_stark_industries: exit: too many arguments` et **ne pas** quitter.

    - **Argument non numérique :** `exit lol` doit afficher `minishell_from_stark_industries: exit: lol: 
	et quitter avec le code `2`.

- **Algorithme :**

	1. `printf("exit\n");`

    2. Si `args[1]` existe, vérifier qu'il est 100% numérique (gérer le `+` ou `-` au début).

        - Si non numérique : Libérer la mémoire (`free_all()`) ➔ `exit(2)`.

    3. Si numérique, vérifier `args[2]`.

        - S'il existe : Afficher erreur ➔ `return (1)`.

    4. Si tout est OK : `code = ft_atoi(args[1])`, libérer la mémoire (`free_all()`), ➔ `exit(code % 256)`.

---
