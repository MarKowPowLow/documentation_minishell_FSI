Que veut dire AST ? C'est un Arbre Syntaxique Abstrait (ASA) (Abstract Syntax Tree en Anglais).

On peut le voir comme un arbre où les noeuds internes sont marqués par des opérateurs et dont les feuilles (noeuds externes) représentent les opérandes de ces opérateurs.

Concrètement pour nous : 

- Les Noeuds internes (les branches) sont la grammaire (`|`, `&&`, `||`). Ce sont eux qui décident comment on enchaîne les actions.

- Les Noeuds externes (les feuilles) sont les commandes (`ls`, `grep`, `echo`...).

Pour faire un exemple, imaginons la commande :

`cmd1 && (cmd2 || cmd3) | cmd4`

Avec notre arbre, ça ressemblerait à ça : 

```
		  [AND] 
		 /     \ 
	[cmd1]      [PIPE] 
				/    \ 
	  [SUBSHELL]      [cmd4]
		   |
		 [OR] 
		 /  \ 
	[cmd2]  [cmd3]
```

On aura donc probablement une énumération pour tout les cas possibles (`| ; && ; || ; cmd`)

Pourquoi c'est utile ? Ça va nous imposé une hiérarchie d'exécution (les priorités des commandes).

Par exemple :

1. L'[[_Executor|Executor]] va regarder la racine (`&&` / AND) et se dire :

	 - C'est un noeud logique. Je ne peux pas toucher à la partie droite tant que la gauche n'est pas finie.
 
2. Exécution de `cmd1` :

	- `cmd1` s'exécute sur l'entrée et la sortie standard (pas de pipe ici).

	- L'[[_Executor|Executor]] attend son code de retour ([[$?]]).

		- Scénario A, la cmd échoue (`return != 0`).
			- Il s'arrête. La partie droite est ignorée vu qu'on a un && comme noeud logique.

		- Scénario B, la cmd réussit (`return == 0`)
			- Il se dit, ok on poursuit, j'appelle la fonction d'exécution sur le noeud de droite (`|`).

3. L'[[_Executor|Executor]] arrive sur le noeud de droite (`|` / PIPE)

	- Il se dit, c'est un tuyau, je dois connecter deux processus.

	- Action 1 -> Il crée le tube avec `pipe()`.

	- Action 2 -> Il fait un premier `fork` (Enfant Gauche).

		- Il dit à l'enfant, toi, tu prends tout le bloc SUBSHELL `(cmd2 || cmd3)` et tu écris ta sortie dans le pipe.

	- Action 3 -> Il fait un second `fork` (Enfant Droit).
		- Il dit à cet enfant, toi tu es `cmd4` et tu lis ton entrée depuis le pipe.

	- Action 4 -> Le Parent ferme les tuyaux et attend (`waitpid`) que les deux finissent.

4. L'enfant de gauche ( `(...)` / SUBSHELL)

	- L'enfant reçoit l'ordre d'exécuter le groupe.

	- Il regarde son contenu, c'est un noeud OR `||`.

	- Logique:

		- Il essaye d'exécuter `cmd2`.

		- Si `cmd2` rate -> Il exécute `cmd3`.

		- (Pour info, tout ce que `cmd2` ou `cmd3` affiche part directement dans le pipe vers `cmd4`, car l'Enfant Gauche a redirigé son STDOUT (sa sortie standard) avant).

5. L'enfant de droite (`cmd4`)

	- Il lance `execve("cmd4")`.

	- Il reçoit les données filtrées par le groupe précédent via le pipe.

	- Il affiche le résultat final à l'écran.

---
