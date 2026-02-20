### L'Expander, à quoi ça correspond ? 

Le concept principal de notre Expander est d'utiliser une `$VAR`. Quand notre terminal verra un $, il va comprendre qu'il ne doit pas lire littéralement ce qui suit, mais qu'il doit aller chercher la valeur de ce qui suit dans notre Environnement (Tu peux voir l'environnement comme un carnet d'adresse géant)

---

#### 1. Le fonctionnement dans minishell ? 

Imaginons qu'on a nous envoi `echo Bonjour $USER`. Notre `USER` actuellement serait Jarvis.

- L'utilisateur va écrire `echo Bonjour $USER`

- Notre minishell va lire le `$` et se dire, lui c'est pas un oiseau, on va devoir trouver un truc qui s'appelle `$USER` dans notre carnet d'adresse.

- Notre minishell va remplacer le `$USER` par l'adresse dont nous disposons pour cette variable.

- Notre commande finale deviendra `echo Bonjour Jarvis`.

---

#### 2. D'où viennent ces variables ?

Elles viennent de l'environnement (le `char **envp` qu'on reçois dans le main, ou la copie qu'on aura fait pour notre minishell dans la liste chaînée).

Ça ressemble à un truc comme ça : 

```
USER=Jarvis
HOME=/home/Jarvis
PATH=/bin:/usr/bin
PWD=/home/Jarvis/minishell_from_stark_industries
```

Quand on va chercher notre `$USER`, notre code devra :

- Parcourir la liste.

- Comparer ce qu'il y a avant le = avec la variable recherché (Actuellement `USER`).

- Renvoyer ce qu'il y a après le =, `Jarvis`.

---

#### 3. Il y aura des cas spéciaux !

- Si la variable n'existe pas :
	- Si on tape `echo $ARTHURUS` et que `ARTHURUS` n'est pas dans l'environnement.

		- Notre minishell remplacera `$ARTHURUS` par rien du tout (une chaîne vide).

		- Résultat, notre `echo` n'affichera rien, juste un `\n`.

- Si on nous envoi `$?` :
	- Ce n'est pas une variable liée à l'environnement ! Elle est gérée en interne par notre minishell. 

		- C'est le résultat de la dernièe commande exécutée.

		- `0` = Succès.

		- `1` à `127`... = Erreur (Si on reçoit des signaux, ça peut aller plus loin)

Exemple :

```
ls fichier_inexistant
echo $?  # Affiche 1 (car ls a échoué)
```

- Les quotes !

	- Sans quotes, on doit remplacer ! 
		- Avec notre exemple précédent, `echo $USER` -> `Jarvis`

	- Double quotes (`""`), on remplace aussi !
		- Toujours avec le précédent, `echo "Bonjour $USER"` -> `Bonjour Jarvis` 

	- Simple quotes (`''`), on ne touche à rien du tout !
		- `echo 'Bonjour $USER'` -> `Bonjour $USER` (Le texte s'affiche littéralement).

---

#### 4. Comment faire ça ? (Ça sera notre Expander !)

Notre expander reçoit la chaîne envoyer par notre [[_Lexer|Lexer]]. 

Il va :

- Lire la chaîne caractère par caractère.

- Il tombe sur un `$`.

- Il regarde ce qui suit (ex. : `$USER`)
	- Il s'arrêtera sur le premier caractère qui n'est pas alphanumérique.

- Il cherche ce nom dans notre variable d'environnement.

- Il crée une nouvelle chaîne où on a remplacé `$USER` par `Jarvis`.

- Il continue de lire la suite de la chaîne.

---
