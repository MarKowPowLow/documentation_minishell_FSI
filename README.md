# Documentation pour le projet 42 Minishell.

Ce répertoire contient une documentation issue de recherches sur Minishell (à la base, pour nous donner, à mon collègue et moi, un fil rouge à suivre dans le cas où on serait un peu perdus ^^).

Cette documentation a été faite grâce à un logiciel nommé Obsidian. Ce logiciel, entre autres choses, est utile pour :

- Lier toutes nos fiches techniques entre elles (Comme un mini-Wikipédia) grâce à un système de liens très simple. Si on parle de l'Executor, on peut cliquer pour atterrir directement sur la fiche liée à l'Executor.

- Visualiser notre architecture (Enfin, pas sûr que ça soit utile pour le graph xD).

- Gérer notre avancement grâce à un tableau Kanban pour suivre nos tâches (To Do, In Progress, Done) directement dans notre documentation.

- Rester compatible avec GitHub, vu que c'est du pur Markdown, l'intégralité de cette documentation reste parfaitement lisible sur GitHub !


---

### Installation d'Obsidian

Si vous souhaitez ouvrir ce vault (notre dossier où Obsidian stocke nos notes), il vous suffit d'installer Obsidian, disponible sur le site [Obsidian.md](https://obsidian.md/).

Téléchargez la version pour votre OS et installez-là !

Au lancement, cliquez sur **Open folder as vault** (Ouvrir un dossier comme coffre) et sélectionnez le dossier racine du dépôt Git.


---

### Activer le plugin "Kanban"

Par défaut, Obsidian ne dispose pas d'un mode Kanban, c'est un plugin communautaire mis à disposition par **mgmeyers**.

Pour pouvoir utiliser Kanban vous devrez :

- Ouvrir les **Paramètres** (La roue crantée en bas à gauche).

- Dans le menu de gauche, allez sur **Community plugins** (Plugins communautaires).

- Désactivez le **Restricted Mode** (Mode Restreint) -> Cliquez sur le bouton **Turn on community plugins**.

- Cliquez par la suite sur **Browse** (Parcourir).

- Recherchez **Kanban** dans la barre de recherche.

- Cliquez sur le plugin **Kanban** (par mgmeyers) et l'installer.

- Une fois installé, il suffit de l'activer (**Enable**).


Si vous souhaitez utiliser le plugin de votre côté pour créer d'autres tableaux, faites un `Ctrl + P` pour ouvrir la palette de commandes et tapez `Kanban`. Par la suite, vous n'aurez qu'à choisir `Kanban: Create new board` puis lui donner un nom.


---

### Structure de la documentation

```
Minishell-Vault/
├── .obsidian/          <-- Le dossier d'Obsidian (caché sur Mac/Linux)
├── 00_Management/      <-- Dashboard, Routine, Journal de bugs et Tags.
├── 10_Research/        <-- Documentation théorique (Lexer, Parser...)
├── 20_Specs/           <-- Spécifications techniques (Structures C, Prototypes...)
├── 30_Resources/       <-- Les PDF utiles et liens utilisés
├── README.md           <-- Ce que vous lisez actuellement
├── README_EN.md        <-- Le même ! Mais en Anglais !
└── Roadmap             <-- La Roadmap
```
