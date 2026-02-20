### `Ctrl-C` : L'Interrupteur (SIGINT)

Nom technique : `SIGINT` (Signal Interrupt). Ce qu'il dit à l'ordi : _"Arrête tout de suite ce que tu fais !"_

- Comportement par défaut (Bash) : Il tue le processus en cours.

- Dans Minishell :

    - Quand tu es sur le prompt (rien ne tourne) : On ne doit **pas** tuer notre shell ! Il doit juste afficher une nouvelle ligne avec un nouveau prompt.

    - Quand une commande tourne (ex: `cat`) : On doit tuer la commande `cat`, mais notre Minishell doit survivre et nous redonner la main.

Il faut utiliser `rl_on_new_line`, `rl_replace_line` et `rl_redisplay` pour réafficher le prompt proprement.