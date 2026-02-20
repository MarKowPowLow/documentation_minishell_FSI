Fonctions autorisées :

- [[access]]
- [[add_history]]
- [[chdir]]
- [[close]]
- [[closedir]]
- [[dup]]
- [[dup2]]
- [[execve]]
- [[exit|exit]]
- [[fork]]
- [[free]]
- [[fstat]]
- [[getcwd]]
- [[getenv]]
- [[ioctl]]
- [[isatty]]
- [[kill]]
- [[lstat]]
- [[malloc]]
- [[open]]
- [[opendir]]
- [[perror]]
- [[pipe]]
- [[printf]]
- [[read]]
- [[readdir]]
- [[10_Research/Usable Functions/readline|readline]]
- [[rl_clear_history]]
- [[rl_on_new_line]]
- [[rl_redisplay]]
- [[rl_replace_line]]
- [[sigaction]]
- [[sigaddset]]
- [[sigemptyset]]
- [[signal]]
- [[stat]]
- [[strerror]]
- [[tcgetattr]]
- [[tcsetattr]]
- [[tgetent]]
- [[tgetflag]]
- [[tgetnum]]
- [[tgetstr]]
- [[tgoto]]
- [[tputs]]
- [[ttyname]]
- [[ttyslot]]
- [[unlink]]
- [[wait]]
- [[wait3]]
- [[wait4]]
- [[waitpid]]
- [[write]]

---

#### Headers des fonctions : 

- **`<stdio.h>`** : printf, perror
- **`<stdlib.h>`** : malloc, free, exit, getenv
- **`<unistd.h>`** : write, access, read, close, fork, getcwd, chdir, unlink, execve, dup, dup2, pipe, isatty, ttyname, ttyslot
- **`<fcntl.h>`** : open
- **`<sys/wait.h>`** : wait, waitpid, wait3, wait4
- **`<signal.h>`** : signal, sigaction, sigemptyset, sigaddset, kill
- **`<sys/stat.h>`** : stat, lstat, fstat
- **`<dirent.h>`** : opendir, readdir, closedir
- **`<string.h>`** : strerror
- **`<sys/ioctl.h>`** : ioctl
- **`<termios.h>`** : tcsetattr, tcgetattr
- **`<curses.h>` / `<term.h>`** : tgetent, tgetflag, tgetnum, tgetstr, tgoto, tputs
- **`<readline/readline.h>`** : readline, rl_on_new_line, rl_replace_line, rl_redisplay
- **`<readline/history.h>`** : add_history, rl_clear_history

---
### 1. La gestion de l'Input (Le début de la boucle)

C'est ce qui permet d'afficher le prompt `minishell_from_stark_industries>$` et de récupérer ce que l'utilisateur tape.

- **`readline(const char *prompt)`** : Affiche le `prompt` et attend que l'utilisateur tape une ligne. Elle gère l'édition de ligne (flèches gauche/droite) toute seule. Retourne la string allouée.

- **`add_history(const char *line)`** : Ajoute la ligne tapée à l'historique (flèche du haut).

- **`rl_on_new_line`**, **`rl_replace_line`**, **`rl_redisplay`**, **`rl_clear_history`** :

    - Ces fonctions servent principalement à **gérer le `Ctrl-C` proprement**.

    - Si l'utilisateur fait `Ctrl-C` pendant qu'il tape, on doit dire à readline : "On passe à la ligne suivante, vide le buffer actuel et réaffiche le prompt propre".

Exemple :

```
// readline : Affiche "minishell_from_stark_industries> " et attend une entrée
char *input = readline("minishell_from_stark_industries> ");
if (input)
{
    // add_history : Ajoute la ligne à l'historique (flèche haut)
    if (*input)
        add_history(input);
    printf("Tu as écrit : %s\n", input);
    free(input); // Toujours free le retour de readline
}

// rl_clear_history : À appeler à la toute fin du programme pour nettoyer
rl_clear_history();
```

**Gestion du `Ctrl-C` (Signal Handler) :** C'est là qu'on utilise les fonctions bizarres `rl_`.

```
void handle_sigint(int sig)
{
    printf("\n"); // Saut de ligne
    rl_on_new_line(); // Dit à readline qu'on est sur une nouvelle ligne
    rl_replace_line("", 0); // Vide la ligne actuelle
    rl_redisplay(); // Réaffiche le prompt propre
}
```

---
### 2. Les Processus

Pour lancer une commande (`ls`), on ne peut pas le faire dans notre programme principal (sinon notre minishell devient `ls` et s'arrête). On doit le cloner (comme dans Pipex).

- **`fork()`** : Clone le processus actuel.

    - Retourne `0` dans l'enfant (le futur `ls`).

    - Retourne le PID de l'enfant dans le parent (notre minishell).

- **`execve(path, args, env)`** : Transforme le processus actuel en un nouveau programme. Si ça marche, le code après `execve` ne s'exécute jamais.

- **`exit(code)`** : Termine le processus proprement avec un code de retour (0 = succès, >0 = erreur).

- **`wait` / `waitpid`** : Le parent attend que l'enfant ait fini.

    - Indispensable pour récupérer le code de retour (`$?`) via les macros `WEXITSTATUS`, etc.

    - `wait3` / `wait4` sont des versions plus vieilles qui donnent aussi des infos sur l'utilisation des ressources (CPU, RAM).

- **`kill(pid, sig)`** : Envoie un signal à un processus. Si des processus sont bloqués, on s'en servira peut-être pour les down ? Je sais pas.

Exemple :

```
pid_t pid = fork(); // On se dédouble ici

if (pid == 0) // On est dans l'ENFANT (à ne pas sortir de son contexte)
{
    char *args[] = {"/bin/ls", "-l", NULL};
    char *env[] = {NULL}; // Ou l'environnement réel

    // execve : Remplace le processus par "ls". Si ça marche, le code s'arrête.
    execve(args[0], args, env);

    // Si on arrive ici, c'est que execve a échoué
    perror("Exec failed");
    exit(1); // On tue l'enfant avec une erreur
}
else // On est dans le PARENT
{
    int status;
    // waitpid : On attend que l'enfant (pid) finisse
    waitpid(pid, &status, 0);

    // Vérifier si l'enfant a fini normalement
    if (WIFEXITED(status))
        printf("L'enfant a fini avec le code : %d\n", WEXITSTATUS(status));
}
```

- `kill(pid, SIGKILL)` : Tuerait le processus `pid`.

---
### 3. Fichiers & Pipes (Redirections `|`, `>`, `<`)

- **`pipe(fd[2])`** : Crée un tube. `fd[1]` pour écrire, `fd[0]` pour lire.

- **`open`, `close`, `read`, `write`** : Des inconnues au bataillon :D.

    - _Note :_ Dans minishell, On va beaucoup utiliser `close` pour éviter les fuites de File Descriptors (FD leaks) (Pipex et notre garbage cleaner aussi).

- **`dup`, `dup2(old, new)`** : Pour faire des redirections.

    - `dup2(fd_fichier, STDOUT_FILENO)` permet de dire "Maintenant, quand on écrit dans la console, ça part dans le fichier".

- **`unlink(path)`** : Supprime un fichier.

    - _Usage vital :_ Pour le **Heredoc** (`<<`). Tu crées un fichier temporaire, tu écris dedans, tu le donnes à la commande, puis tu l'effaces avec `unlink`.

Exemple :

```
// pipe : Crée un tunnel. fd[0] lecture, fd[1] écriture.
int fd[2];
if (pipe(fd) == -1)
    return (perror("Pipe error"));

// open : Ouvre un fichier
int file_fd = open("out.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);

// dup2 : Redirection. "La sortie standard (1) devient le fichier file_fd"
dup2(file_fd, STDOUT_FILENO);

// write : Écrit sur le FD 1 (qui est maintenant le fichier)
write(1, "Coucou", 6);

// close : Toujours fermer ce qu'on n'utilise plus !
close(fd[0]);
close(fd[1]);
close(file_fd);

// unlink : Supprime un fichier
unlink("out.txt");
```

---
### 4. Navigation & Système de fichiers

Pour recoder `cd`, `pwd` et l'exécution via `PATH`.

- **`getcwd`** : (Get Current Working Directory). C'est la commande `pwd`.

- **`chdir(path)`** : (Change Directory). C'est le moteur de la commande `cd`.

- **`access(path, mode)`** : Vérifie si un fichier existe et si on a le droit de l'exécuter.

    - Sert à vérifier si `/bin/ls` existe avant d'essayer de l'exécuter.

- **`opendir`, `readdir`, `closedir`** : Pour ouvrir et lire un dossier.

    - Sert pour le bonus **wildcards** (`*.c`) pour lister les fichiers (Si on te dit `*/*/*.c`, faudra tout prendre dans tout les dossiers, sous-dossiers et sous-sous-dossier qui ont une terminaison en .c).

- **`stat`, `lstat`, `fstat`** : Récupère les infos d'un fichier (taille, type, permissions).

    - Sert à vérifier si un chemin est un dossier pour renvoyer l'erreur "Is a directory".

```
// getcwd : Récupère le dossier actuel (pwd)
char buffer[1024];
if (getcwd(buffer, sizeof(buffer)))
    printf("Dossier actuel : %s\n", buffer);

// chdir : Change de dossier (cd)
if (chdir("..") == 0)
    printf("On est remonté d'un cran\n");

// access : Vérifie si un fichier est exécutable (X_OK) ou existe (F_OK)
if (access("/bin/ls", X_OK) == 0)
    printf("On peut exécuter ls !\n");

// opendir / readdir : Lister les fichiers (pour Wildcards *)
DIR *dir = opendir(".");
struct dirent *entry;
if (dir)
{
    while ((entry = readdir(dir)) != NULL)
        printf("Fichier trouvé : %s\n", entry->d_name);
    closedir(dir);
}

// stat : Infos sur un fichier
struct stat file_info;
if (stat("main.c", &file_info) == 0)
    printf("Taille du fichier : %lld octets\n", file_info.st_size);
```

---
### 5. Les Signaux ([[Ctrl-C]], [[Ctrl-Antislash]], [[Ctrl-D]])

La partie la plus "système" du projet.

- **`signal`** : L'ancienne façon de gérer les signaux. Moins précise.

- **`sigaction`** : La "vraie" façon. Tu définis une structure qui dit "Si je reçois SIGINT ([[Ctrl-C]]), lance telle fonction".

- **`sigemptyset`, `sigaddset`** : Sert à configurer les masques de signaux pour `sigaction` (pour dire quels signaux bloquer pendant l'exécution du handler).

Exemple :

La méthode moderne pour intercepter `Ctrl-C`.

```
struct sigaction sa;

// On vide le set de signaux
sigemptyset(&sa.sa_mask);
// On ajoute SIGQUIT (Ctrl-\) au masque pour le bloquer pendant le handler
sigaddset(&sa.sa_mask, SIGQUIT);

sa.sa_handler = handle_sigint; // La fonction définie plus haut
sa.sa_flags = SA_RESTART; // Redémarre les appels système (comme read) si interrompus

// Applique la configuration pour SIGINT
sigaction(SIGINT, &sa, NULL);
```

---
### 6. Terminal & TTY (Configuration avancée)

Parfois, minishell aura besoin de parler au terminal lui-même, pas juste d'écrire du texte.

- **`isatty(fd)`** : Vérifie si un FD (ex: entrée standard 0) est relié à un terminal ou à un fichier/pipe. Utile pour savoir si on doit afficher le prompt ou non.

- **`ttyname`, `ttyslot`** : Récupère le nom du terminal (ex: `/dev/ttys001`).

- **`ioctl`** : "Input/Output Control". C'est un couteau suisse pour configurer le périphérique.

- **`tcsetattr`, `tcgetattr`** : **Très important.**

    - Permet de changer le mode du terminal.

    - Exemple clé : Cacher les caractères `^C` qui s'affichent quand tu fais [[Ctrl-C]]. Tu modifies les attributs du terminal pour désactiver l'écho des signaux de contrôle.

Exemple :

C'est souvent utilisé pour masquer le `^C` qui s'affiche ou gérer le mode non-canonique (lire caractère par caractère (immédiatemment)) (à l'inverse, le mode canonique, il va lire ligne par ligne, par exemple quand tu fais enter).

```
// isatty : Vérifie si on est dans un vrai terminal (pas un pipe)
if (isatty(STDIN_FILENO))
    printf("Je suis dans un terminal interactif\n");

// ttyname : Le nom du fichier terminal
printf("Mon TTY est : %s\n", ttyname(STDIN_FILENO));

// tcgetattr / tcsetattr : Modifier les attributs
struct termios term;

// 1. On récupère la config actuelle
tcgetattr(STDIN_FILENO, &term);

// 2. On modifie : On enlève le flag ECHOCTL (pour ne pas afficher ^C)
term.c_lflag &= ~ECHOCTL;

// 3. On applique la nouvelle config
tcsetattr(STDIN_FILENO, TCSANOW, &term);

// ioctl : Exemple pour obtenir la taille de la fenêtre
struct winsize w;
ioctl(STDOUT_FILENO, TIOCGWINSZ, &w);
printf("Lignes : %d, Colonnes : %d\n", w.ws_row, w.ws_col);
```

---
### 7. Termcaps

Toutes les fonctions commençant par `t` (`tgetent`, `tgetstr`, `tgoto`, `tputs`...) servent à manipuler le curseur (déplacer le curseur à un endroit précis, effacer l'écran, mettre en couleur via la base de données termcap).

- Dans minishell, elles ne sont pas obligatoires sauf si tu veux faire des trucs stylés visuellement ou gérer l'édition de ligne sans `readline` (ce qui est suicidaire).

Exemple :

Si on doit les utiliser, il faut link la lib `-ltermcap` ou `-lncurses`.

```
char buf[1024];
char *area = NULL; // ou un buffer malloc

// tgetent : Charge la base de données pour le terminal actuel (getenv("TERM"))
tgetent(buf, getenv("TERM"));

// tgetstr : Récupère la commande "cm" (cursor move)
char *cm = tgetstr("cm", &area);

// tgoto : Prépare la string pour aller en (10, 5)
char *move_code = tgoto(cm, 10, 5);

// tputs : Envoie la commande au terminal (déplace le curseur)
tputs(move_code, 1, putchar);
```

---
### 8. Utilitaires & Erreurs

- **`malloc`, `free`** : Jamais utilisé depuis le début :P.

- **`printf`** : Autorisé ! Incroyable, plus besoin de `ft_putstr`.

- **`strerror(errno)`** : Transforme le numéro d'erreur global `errno` en phrase lisible (ex: "Permission denied").

- **`perror(msg)`** : Affiche directement l'erreur sur la sortie d'erreur (STDERR). Ex: `perror("minishell_from_stark_industries")` affichera `minishell_from_stark_industries: Permission denied`.

- **`getenv(name)`** : Récupère une variable d'environnement.

    - _Attention :_ Dans minishell, on nous demandera de gérer nos propres variables d'environnement dans une liste, donc on utilisera pas `getenv` tant que ça, sauf pour initialiser au début.

Exemple :

```
// strerror : Transforme un code d'erreur (errno) en texte
// Exemple si open() échoue, errno vaut peut-être EACCES
printf("Erreur : %s\n", strerror(EACCES)); // Affiche "Permission denied"

// perror : Le raccourci ultime
// Si open échoue :
perror("minishell_from_stark_industries"); // Affiche "minishell_from_stark_industries: Permission denied" sur STDERR

// getenv : Chercher une variable
char *path = getenv("PATH");
if (path)
    printf("PATH est : %s\n", path);
```

---
