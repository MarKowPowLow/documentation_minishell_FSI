---

kanban-plugin: board

---

## 🔴 Todo

- [ ] Define the main structures in `minishell.h` (`t_token`, `t_ast_node`, `t_env`).
	|
	Créer les structures principales dans `minishell.h` (`t_token`, `t_ast_node`, `t_env`).
- [ ] Implement the **main loop** `while(1)` with `readline` and `add_history`.
	|
	Faire la boucle principale `while(1)` avec `readline` et `add_history`.
- [ ] Code the function to convert `char **envp` into a `t_env` linked list.
	|
	Coder la fonction qui convertit `char **envp` en liste chaînée `t_env`.
- [ ] History management: Ensure empty lines are not added to the history.
	|
	Gérer l'historique : ne pas ajouter une ligne vide à l'historique.
- [ ] Implement the State Machine to handle quotes (`STATE_DEFAULT`, `STATE_SQUOTE`, `STATE_DQUOTE`).
	|
	Mettre en place la machine à états (State Machine) pour gérer les quotes (`STATE_DEFAULT`, `STATE_SQUOTE`, `STATE_DQUOTE`).
- [ ] Function to ignore whitespaces (unless they are inside quotes).
	|
	Fonction pour ignorer les espaces (sauf s'ils sont dans des quotes).
- [ ] Function to identify operators (`|`, `<`, `>`, `<<`, `>>`).
	|
	Fonction pour identifier les opérateurs (`|`, `<`, `>`, `<<`, `>>`).
- [ ] Split the string into words and create the `t_token` doubly linked list.
	|
	Découper la chaîne en mots et créer la liste doublement chaînée `t_token`.
- [ ] Handle syntax errors: "Unclosed quotes".
	|
	Gérer l'erreur de syntaxe : "Quotes non fermées".
- [ ] Function to check global syntax (e.g., a `|` at the start of the line, or `||`).
	|
	Fonction pour vérifier la syntaxe globale (ex: un `|` au début de la ligne, ou deux `|` de suite).
- [ ] Extract redirections: Identify `< file` and store them in the command structure.
	|
	Extraire les redirections : identifier `< fichier` et les ranger dans la structure de la commande.
- [ ] Handle errors: Redirection without a file (e.g., `ls >` ).
	|
	Gérer l'erreur : redirection sans fichier derrière (ex: `ls >` ).
- [ ] Extract arguments: Convert remaining tokens into a `char **args` array.
	|
	Extraire les arguments : convertir les tokens restants en un tableau `char **args`.
- [ ] Build "Pipe" nodes: Split the binary tree into left and right branches.
	|
	Construire les nœuds "Pipe" : séparer la gauche et la droite dans l'arbre binaire.
- [ ] Function to locate `$VARIABLE` within arguments.
	|
	Fonction pour trouver les `$VARIABLE` dans les arguments.
- [ ] Replace `$VARIABLE` with its value by searching the `t_env` list.
	|
	Remplacer `$VARIABLE` par sa valeur en cherchant dans `t_env`.
- [ ] Handle the special case **`$?`** (replace with the last command's exit code).
	|
	Gérer le cas spécial `$?` (qui doit être remplacé par le code d'erreur de la dernière commande).
- [ ] If `$VARIABLE` does not exist, replace it with an empty string.
	|
	Si `$VARIABLE` n'existe pas, la remplacer par une chaîne vide.
- [ ] Quote Removal: Strip quotes (e.g., transform `"hello"` into `hello`) just before execution.
	|
	Supprimer les guillemets (Quote Removal) : transformer `"bonjour"` en `bonjour` juste avant l'exécution.
- [ ] Execute a "solo" builtin (no pipe) within the parent process.
	|
	Exécuter un builtin "solo" (sans pipe) dans le parent.
- [ ] Function to find a command's path (split `$PATH` and check with `access()`).
	|
	Fonction pour trouver le chemin d'une commande (découper la variable `$PATH` avec `split` et tester avec `access()`).
- [ ] Launch a simple external command using `fork()` and `execve()`.
	|
	Lancer une commande simple externe avec `fork()` et `execve()`.
- [ ] Implement simple redirection logic (`>` and `<`) with `open()` and `dup2()`.
	|
	Implémenter la logique des redirections simples (`>` et `<`) avec `open()` et `dup2()`.
- [ ] Implement Append mode (`>>`).
	|
	Implémenter le mode Append (`>>`).
- [ ] Implement Heredoc (`<<`): Read from stdin using `readline` until the delimiter, then write to a pipe or temporary file.
	|
	Implémenter le Heredoc (`<<`) : lire l'entrée standard avec `readline` jusqu'au mot-clé, écrire dans un pipe ou fichier temporaire.
- [ ] Launch a Pipe (`|`): Create pipes, handle `fork()`, set up `dup2()`, and properly close all FDs!
	|
	Lancer un Pipe (`|`) : créer les tuyaux, faire les `fork()`, brancher les `dup2()`, et bien fermer les `fd` !
- [ ] Wait for children: Use `waitpid()` and correctly retrieve the exit status for `$?`.
	|
	Attendre les enfants : utiliser `waitpid()` et récupérer correctement le code de retour pour `$?`.
- [ ] **`echo`** (with perfect management of `-n` and multiple `-nnnn`).
	|
	`echo` (avec gestion parfaite du `-n` et des `-nnnn`).
- [ ] **`cd`** (update `PWD` and `OLDPWD` variables in the environment).
	|
	`cd` (qui met à jour les variables `PWD` et `OLDPWD` dans l'environnement).
- [ ] **`pwd`** (display current working directory).
	|
	`pwd` (affichage du dossier courant).
- [ ] **`export`** (no arguments: display env sorted alphabetically).
	||
	**`export`** (with arguments: add/modify one or more variables, validate names).
	|
	`export` (sans argument : afficher l'env triée par ordre alphabétique).
	||
	`export` (avec arguments : ajouter ou modifier une/plusieurs variables, vérifier la validité du nom).
- [ ] **`unset`** (remove one or more variables from the linked list).
	|
	`unset` (supprimer une ou plusieurs variables de la liste chaînée).
- [ ] **`env`** (display the `t_env` list).
	|
	`env` (afficher la liste `t_env`).
- [ ] **`exit`** (clean exit, handle numeric arguments and exit codes).
	|
	`exit` (quitter le shell proprement, gérer les arguments numériques et le code de retour).
- [ ] `Ctrl-C` (SIGINT) in the prompt: Display a new line with a fresh, empty prompt.
	|
	`Ctrl-C` (SIGINT) dans le prompt : afficher une nouvelle ligne avec un nouveau prompt vide.
- [ ] `Ctrl-D` (EOF) in the prompt: Quit Minishell (equivalent to `exit`).
	|
	`Ctrl-D` (EOF) dans le prompt : quitter Minishell (équivalent à `exit`).
- [ ] `Ctrl-\` (SIGQUIT) in the prompt: Do nothing.
	|
	`Ctrl-\` (SIGQUIT) dans le prompt : ne rien faire.
- [ ] Signals during blocking processes (e.g., while `cat` is running): `Ctrl-C` and `Ctrl-\` must stop the child process, not the shell.
	|
	Signaux pendant un processus bloquant (ex: quand `cat` tourne) : `Ctrl-C` et `Ctrl-\` doivent arrêter le `cat`, pas le shell.
- [ ] Signals during a Heredoc (`<<`): Properly close the input prompt.
	|
	Signaux pendant un Heredoc (`<<`) : fermer proprement le prompt de saisie.
- [ ] Leak Hunting: Implement a `free_all()` function (clear AST, tokens, environment, etc.).
	|
	Traquer les leaks : faire une fonction `free_all()` (libérer l'AST, les tokens, l'environnement, etc.).
- [ ] Valgrind testing (manage suppression files for `readline` internal leaks).
	|
	Tester avec Valgrind (et gérer les suppressions pour `readline`).


## 🟡 In Progress



## 🟢 Done

- [ ] Makefile with a logo Stark Industries :D #MarKow
- [ ] Create the **folder structure** (`src`, `headers`, `libft`).
	#MarKow




%% kanban:settings
```
{"kanban-plugin":"board","list-collapse":[false,false,false],"tag-sort":[{"tag":"#MarKow"},{"tag":"#Marvin"},{"tag":"#to_check"},{"tag":"#to_ask"},{"tag":"#to_test"},{"tag":"#urgent"}],"tag-colors":[],"move-tags":true,"tag-action":"kanban"}
```
%%
