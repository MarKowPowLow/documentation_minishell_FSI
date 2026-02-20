### La Routine Journalière Git

**Le postulat de départ :** On a 3 niveaux de branches (`main` ➔ `dev` ➔ `feature/...`). Le travail quotidien se fait **uniquement** sur les branches `feature/`.

---

### Phase 1 : Le Matin

_Objectif : Récupérer le travail que j'ai validé la veille et préparer ton atelier._

1. **Ouvre VS Code.**

2. **Bascule sur le Labo :** Clique en bas à gauche et sélectionne la branche `dev`.

3. **Mise à jour (Pull) :** Clique sur le bouton de synchronisation (Sync) ou tape `git pull origin dev` dans le terminal. _Tu as maintenant la toute dernière version du projet. (Si tu es encore sur ta branche à continuer ton projet, regarde la phase 2.5 et ignore le point 4.)_

4. **Crée ton atelier du jour :** Toujours depuis `dev`, crée une nouvelle branche avec un nom clair.

    - _Exemple :_ `feature/lexer_quotes` ou `fix/cd_segfault`...

5. **Commence à coder !**


---

### Phase 2 : La Journée

_Objectif : Avancer pas à pas et sauvegarder régulièrement._

Pendant que tu codes, n'attends pas la fin de la journée pour faire un énorme commit. Fais des petits commits logiques (des "checkpoints") au cas où tu casses tout ton code, tu pourras toujours revenir en arrière ^^.

1. Ton code compile et une petite étape est finie ?

2. **Stage (+)** les fichiers modifiés dans l'onglet Source Control.

3. **Commit** avec un message clair :

    - `feat: ajout de la reconnaissance des pipes` (Pour une nouveauté)

    - `fix: correction de la fuite mémoire dans cd` (Pour un bug)

    - `docs: ajout des commentaires dans l'executor` (Pour du texte)

4. _Répète cette boucle autant de fois que nécessaire dans la journée._


---

### Phase 2.5 : L'Alerte "Mise à jour du binôme"

_Scénario : Il est 14h. J'ai fini mon Parser, je l'ai validé et fusionné sur `dev`. Tu as besoin de mon code dans ta propre branche `feature/` actuelle pour continuer._

1. Clique en bas à gauche et retourne sur `dev`.

2. Fais un **Sync** (Pull) pour télécharger mon Parser.

3. Re-clique en bas à gauche et retourne sur ta branche `feature/ma_branche`.

4. Fais `Ctrl+Shift+P` (ou `Cmd+Shift+P`) ➔ **Git: Merge Branch...** ➔ Choisis `dev`.

5. _Mon binôme est maintenant dans ton atelier, tu peux continuer à travailler._


---

### Phase 3 : Le Soir

_Objectif : Envoyer ton travail propre dans le laboratoire commun (`dev`) et nettoyer ton PC._

1. **Envoie sur le Cloud (Push) :** Clique sur "Publish Branch" (ou "Sync") pour envoyer ta branche `feature/` sur GitHub.

2. **La Demande Officielle (Pull Request) :**

    - Va sur le site de GitHub.

    - Clique sur le gros bouton vert **"Compare & pull request"**.

    - ⚠️ **ATTENTION :** Change la branche de destination ! Par défaut GitHub veut fusionner sur `main`. Tu dois choisir **`base: dev`** ⬅️ **`compare: feature/...`**.

3. **La validation par l'autre :**  Normalement, **On ne valide jamais son propre PR.** C'est l'autre qui doit lire vite fait le code sur GitHub, et c'est LUI qui clique sur le bouton "Merge Pull Request".

4. **Le Nettoyage Cloud :** Clique sur le bouton violet **"Delete branch"** sur GitHub.

5. **Le Nettoyage Local (Sur ton PC) :**

    - Dans VS Code, retourne sur la branche `dev`.

    - Fais un **Sync** (Pull) pour récupérer ta propre fusion !

    - Supprime ta branche locale (`Git: Delete Branch` ➔ `feature/...`).
