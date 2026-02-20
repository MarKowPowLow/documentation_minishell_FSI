
## Pour une belle documentation, il y a le pdf de notre [[Github en équipe !|Dieu Eliot]]

[![Image de github](https://framerusercontent.com/images/jmGsErVaXE9HMnmsIjHDba05DSg.png?scale-down-to=1024&width=1600&height=900)

## Le Gitflow ! 

Le Gitflow, dans le monde professionnel, c'est une architecture de branche plus solide pour travailler en équipe, sans jamais casser la version finale.
(Par exemple, comme ce que Ludo nous avait raconté, que l'un des devs senior de sa boîte avait balancer tout sur la branche prod...)

---
### 1. Le concept des 3 niveaux

Au lieu d'avoir un seul univers, le `main`, on en crée généralement trois, avec des niveaux de sécurité différents :

- Le `main` : 
	- Ça sera notre code parfait, il devra compiler, ne pas avoir de leaks, passer la norme...
		- On n'écrit jamais de code directement dessus. 
		- On ne le mettra à jour qu'à la veille du rendu ou, si on valide des grosses étapes et qu'elles marchent à 100 % (Genre si on fini entièrement le [[_Parser|parser]]...).

- Le `dev` : 
	- Ça sera le coeur de notre projet. C'est ici qu'on va rassembler tout notre code petit à petit. 
	- Le code ici devra compiler, mais si on a quelques bugs ou des leaks, ça ne sera pas si grave.

- Un `feature/nom_de_la_feature` : 
	- C'est ici qu'on codera au quotidien, quand on se lance dans une tâche, la première chose à faire sera de créer une branche au nom de la feature (fonctionnalitée) qu'on veut faire (par exemple, le [[_Parser|parser]]).
	- Si le code ne compile pas ou qu'il explose, c'est pas si grave, il n'y aura que toi dans cette branche :D.

---
### 2. Le cycle de vie quotidien (notre workflow)

#### Étape 1 : On créer notre sous-branche (le matin généralement :D)

Continuons sur l'exemple du parser, on va devoir créer notre atelier de travail, mais à partir de notre branche dev et pas du main !

Pour ce faire :

- Dans VS Code, on clique sur le nom de la branche en bas à gauche.

- On choisi `dev` (pour avoir la dernière version de notre branche)

- On clique sur le bouton de synchronisation (les deux flèches en cercle) pour télécharger les dernières nouveautés que j'aurais potentiellement push la veille. 

- On re-clique en bas à gauche puis -> Create new branch -> On écrit `feature/parser`.
	- VS Code comprendra ça sans problème, comme une arborescence visuelle.

- Si jamais tu veux le faire par le terminal : `git checkout -b feature/parser`

---
#### Étape 2 : On sauvegarde, de temps en temps !

Par la suite, une fois qu'on aura coder dans la journée, on voudra sauvegarder, là, on reste sur du classique :

- Tu vas dans l'onglet des trois points, tu add les fichiers que tu veux commit (le + devant les fichiers ou devant la barre de change si tu veux tout commit).

- Tu mets un message explicite tout de même ! (Genre : `Mise à jour du Parser, la fonction principale est finie, mais il y a toujours des bugs... | Update Parser, main function done, but still have bugs...`)

--- 
#### Étape 3 : On fusionne ! (Bizarre...)

Enfin, le soir, si tu as fini ton Parser et qu'il fonctionne (Si il reste quelques bugs, on pourra toujours voir ça ensemble si tu n'as pas trouvé) :

- Tu fais un push de `feature/parser` vers Github !

- Là, on arrive sur une partie que tu as pas forcément vu, sur GitHub directement, il faudra que tu ailles dans l'onglet **Pull Request**.
	- Par défaut, GitHub voudra fusionner vers le `main`. N'oublie pas de changer le menu déroulant pour dire que tu veux fusionner `feature/parser` avec notre branche `dev` !

- Une fois la PR faite, je checkerais le code arrivant et le validera et la branche `dev` sera fusionnée avec. 

- Tu pourras supprimer ta branche maintenant que tu auras fini. 
	- Il faudra que tu bascules sur la branche dev en premier.

	- Puis, tu fais un `Ctrl+Shift+P` (sur VS Code) et tu écris `Git: Delete Branch`. Enfin, tu n'auras qu'à choisir la branche que tu veux delete.

	- Si jamais tu veux le faire par le terminal, tu n'auras qu'à taper `git branch -d nom_de_la_branche` (Si il refuse de la delete, c'est qu'elle n'a pas encore été fusionné ailleurs, git protège automatiquement).

	- C'est probable que ta branche existe toujours sur le cloud (Sur GitHub) vu que tu auras fait une Pull Request, si elle est validé, on pourra delete ta branche en même temps (Tkt, je m'en occuperais au besoin :D).

---
### 3. Comment on va gérer les mises à jour de l'autre ?

Imaginons, tu es sur la `feature/parser`, moi, en attenant, j'ai fini ma `feature/lexer` et je l'ai mis sur la branche `dev` et tu vas absolument avoir besoin de mon code pour tester le tiens. 

Comment importer mon code dans ton atelier sans tout casser ?
Il va falloir mettre à jour ta branche par rapport à `dev`.

- Reste sur ta branche `feature/parser` dans VS Code.

- Ouvre la palette de commandes `Ctrl+Shift+P` et tape `Git: Merge Branch...` et clique dessus.

- VS Code va te demander depuis quelle branche tu veux importer du code, choisis `dev` !

- Si il y a des conflits (On aura update le même fichier), le Merge Editor va s'ouvrir et tu auras le fichier concerné avec ce genre de lignes (VS Code mets des `+++` et `---` selon quelle branche est concernée de souvenir).

```
    <<<<<<< HEAD (Current Change)
    int i = 0; // Le code sur main
    =======
    int i = 1; // Ton code qui arrive
    >>>>>>> feature/parser
```

-  VS Code affiche des petits boutons au-dessus : "Accept Current Change", "Accept Incoming Change", ou "Accept Both".

- Choisis la bonne version (ou réécris le code manuellement).

- Sauvegarde, stage le fichier (`+`), et commit. Conflit résolu.

--- 
#### 4. La Pull Request (PR)

Vu qu'on fait des pull request, je vais t'expliqué un peu comment ça se passe plus clairement !

- GitHub va voir que tu as pushé `feature/parser` et va te proposer un bouton vert **"Compare & pull request"**. Clique dessus.

- **C'est là que tu vérifies les modifs :** *
    - Tu verras un onglet "Files changed".

    - **Rouge** : Ce qui a été supprimé.

    - **Vert** : Ce qui a été ajouté.

    - C'est le moment pour moi de relire ton code. Je peux mettre des commentaires directement sur les lignes pour te dire ce qui pourrait ne pas aller (genre "Attention, ici tu as oublié de free !").

    - Normalement, on devrait check chacun le code de l'autre.

- **Le Merge :**
    - Si tout est OK (et que GitHub dit "Able to merge", donc pas de conflits), on clique sur le gros bouton vert **Merge pull request**.

    - Cela fusionne ta branche dans `dev` sur le serveur.

- **Retour au local :**
    - De mon côté, il faudra que je fusionne ma branche avec la branche `dev` qui a été mise à jour.

---
### 4. Extensions VS Code cool

1. **Git Graph** (par mhutchie) : 

    - Elle ajoute un bouton dans ta barre d'état.

    - Quand tu cliques, tu vois un magnifique graphique "genre map de métro" avec des points de couleur.

    - Tu peux voir qui a fait quoi, quand les branches se sont séparées et quand elles se sont rejointes.

2. **GitLens** (par GitKraken) :

    - Quand tu cliques sur une ligne de code dans l'éditeur, il écrit en gris clair à côté : _qui a modifié cette ligne et quand_ (le "blame").

    - Très utile pour dire : "Bah Polo, pourquoi tu as supprimé mon `malloc` hier à 23h ?"

