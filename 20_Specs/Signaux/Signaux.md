Bon, pas sûr que le I'm alive bitches soit un code de Bash, mais en changeant notre fonction pour un clean (vu que sigint est sensé kill le processus en cour), ça sera ptêtre mieux :D :

```
#include <signal.h>
#include <stdio.h>

// 1. On crée notre fonction personnalisée (Le "Handler")
void handle_sigint(int sig)
{
    write(1, "\nSignal reçu, I'm alive, bitches !\n", 31);
}

void init_signals(void)
{
    struct sigaction sa;

    // 2. On indique quelle fonction appeler
    sa.sa_handler = handle_sigint;
    
    // 3. On vide le "masque" (on ne bloque pas d'autres signaux pendant l'exécution)
    sigemptyset(&sa.sa_mask);
    
    // 4. Options (0 = standard)
    sa.sa_flags = 0;

    // 5. On applique la structure au signal SIGINT
    sigaction(SIGINT, &sa, NULL);
    
    // Exemple pour ignorer complètement SIGQUIT (Ctrl-\)
    signal(SIGQUIT, SIG_IGN); // On peut utiliser signal pour juste l'ignorer.
}
```