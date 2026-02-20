#### `echo` (L'Afficheur)

- **Ce qu'il fait :** Affiche ses arguments séparés par un espace, suivi d'un retour à la ligne (`\n`).

- **L'option à gérer :** `-n` (ne pas afficher le `\n` final).

- **Les pièges à prévoir :**

    - Gérer les `-n` multiples combinés : `echo -n -n -nnnnn Salut` doit fonctionner comme un seul `-n`.
		`echo -nnnnn salut` ➔ Est un `-n` valide !

    - `echo -n -nmaison` ➔ Affiche `-nmaison` (dès qu'il y a autre chose qu'un `n` après le `-`, ce n'est plus une option).

    - Pas besoin de se prendre la tête avec les espaces, notre lexer fera le tri ! (`echo a b`). On recevra juste le tableau `["echo", "a", "b", NULL]`. Il y aura juste à faire une boucle d'affichage avec un espace entre chaque.

- **Algorithme :**
    
    1. Boucler sur les arguments (à partir de `args[1]`).

    2. Tant que l'argument est exactement `-n` (ou `-nnn`), avancer l'index et activer un flag `newline = 0`.

    3. Afficher les arguments restants avec un espace entre eux.

    4. Si `newline == 1`, faire un `write(1, "\n", 1)`.

---
