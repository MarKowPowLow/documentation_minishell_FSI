#### `export` (Le Créateur)

- **Ce qu'il fait :** Ajoute ou modifie une ou plusieures variables dans notre environnement.

- **Comportement attendu :**

    - Syntaxe : `export CLE=VALEUR` ou `export CLE` (sans valeur).

    - Si `export` est tapé seul : affiche tout l'environnement **trié par ordre alphabétique** (ASCII) précédé de `declare -x` .

- **Les pièges à prévoir :**

    - **La syntaxe stricte :** Une clé de variable ne peut contenir que des lettres, des chiffres et des underscores (`_`), et ne **peut pas** commencer par un chiffre. `export 1VAR=bonjour` doit renvoyer une erreur (`not a valid identifier` et renvoie juste `1`).

    - **Cas sans valeur :** `export VAR` (sans `=`) crée la variable mais elle n'a pas de valeur. Elle existe quand même !

    - **`export` tout court :** S'il n'y a pas d'arguments, il doit afficher tout l'environnement **trié par ordre alphabétique** avec le préfixe `declare -x`. (Ex: `declare -x USER="Jarvis"`). C'est souvent la fonction la plus longue à coder des builtins !

- **Algorithme (Ajout) :**

    1. Vérifier la validité de la chaîne avant le `=`.

    2. Chercher si la clé existe déjà dans la liste chaînée.

    3. Si OUI : `free` l'ancienne valeur, attribuer la nouvelle.

    4. Si NON : Créer un nouveau maillon `t_env` et l'ajouter à la fin de la liste.

---
