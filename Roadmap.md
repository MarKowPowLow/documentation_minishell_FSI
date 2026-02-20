
1. Main Loop

	- [[10_Research/Usable Functions/readline|Readline]] (Récupère la string)
	- [[_Lexer|Lexer]] (Découpe la string en jetons/tokens)

2. Tokeniser

	- [[_Lexer|Lexer]] (t_list de strings)
		- -> [[_Parser|Parser]] (Construction de l'[[AST]])
	- [[_Parser|Parser]] (t_ast done)
		- -> [[_Expander|Expander]] (Remplacer $VAR, gérer les quotes...)
	- [[_Expander|Expander]] ([[AST]] finally clean)
		- -> [[_Executor|Executor]]

3. Exécution

	- Branche A : [[_Builtins|Builtins]] ([[echo]], [[cd]], [[pwd]], [[export]], [[unset]], [[env]], [[Notre fonction Exit|exit]])
	- Branche B : [[_Executor#C. Les Nœuds Commandes (Les Feuilles) Nos Ouvriers|Les commandes externes !]]

4. Systèmes Transverses

	- [[_Environnement|Environnement]] (Copie de l'env)
	- [[_Signaux|Signaux]] (Gérer les [[Ctrl-C]], [[Ctrl-Antislash|Ctrl-\]], [[Ctrl-D]])
	- [[_Garbage Collector|Garbage Collector]] (Pour éviter les leaks, fermer les fds...)
	- [[_Debugs|Debugs]] (Pour tester notre code, maybe un print de notre [[AST]] dedans ?)

5. Les [[_Fonctions autorisées|Fonctions autorisées]] (À lire au début, explique un peu comment tout marche.)