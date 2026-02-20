#### `env` (L'Afficheur d'Environnement)

- **Ce qu'il fait :** Affiche la liste des variables d'environnement.

- **Comportement attendu :**

    - Affiche uniquement les variables qui ont un `=` (donc une valeur associée, même vide).

- **Les pièges à prévoir :**

    - Contrairement à `export` sans argument, `env` ne trie pas la liste.

    - `env` n'affiche **que** les variables qui ont un `=` (une valeur). Celles créées par `export VAR` sans valeur ne doivent pas apparaître ici.

    - Le sujet demande `env` sans options ni arguments. S'il y a des arguments, on peut juste renvoyer une erreur ou l'ignorer selon comment on va calquer le bash d'origine.

- **Algorithme :**

    1. Parcourir la liste chaînée.

    2. Si `noeud->value != NULL`, faire `printf("%s=%s\n", noeud->key, noeud->value)`.

---
