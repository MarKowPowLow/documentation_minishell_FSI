### `Ctrl-\` : Le Destructeur (SIGQUIT)

Nom technique : `SIGQUIT` (Signal Quit). Ce qu'il dit à l'ordi : _"Quitte violemment et laisse une trace (Core Dump)."_

C'est une version plus "sale" de Ctrl-C.

- Comportement par défaut : Il tue le processus et génère souvent un fichier "core" (une copie de la mémoire au moment du crash pour débugger).

- Dans Minishell :

    - Sur le prompt : Il ne doit rien faire du tout. On l'ignore totalement (On affiche juste le prompt vide ou rien).

    - Quand une commande tourne (ex: `cat`) : Il doit tuer la commande. Souvent, ça affiche un message "Quit (core dumped)" dans le terminal (à voir les messages d'erreurs plus tard).

Dans le main, on devra sûrement ignorer ce signal, puis par la suite, si on arrive dans un enfant (quand on gère des commandes), il faudra le réactiver.