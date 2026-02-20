#### `unset` (Le Destructeur)

- **Ce qu'il fait :** Supprime une variable de l'environnement.

- **Comportement attendu :**

    - Détache et libère la mémoire d'un nœud de ta liste `t_env`.

- **Les pièges à prévoir :**

    - C'est ici qu'on va tester la solidité de notre liste chaînée. On doit reconnecter le maillon précédent au maillon suivant, et bien faire un `free()` de la clé, de la valeur, et du nœud.

    - Si la variable n'existe pas, `unset` ne fait rien (et ne renvoie pas d'erreur (donc `0`, succès)).

    - Attention à la même syntaxe stricte que `export` pour le nom de la variable.

    - Attention à ne pas perdre le pointeur de tête si on `unset` le tout premier élément de la liste !

- **Algorithme :**

    1. Parcourir la liste `t_env`.

    2. Si `courant->next->key == argument` :

    3. `tmp = courant->next`

    4. `courant->next = tmp->next` (On refait la couture).

    5. `free(tmp->key)`, `free(tmp->value)`, `free(tmp)`.

---
