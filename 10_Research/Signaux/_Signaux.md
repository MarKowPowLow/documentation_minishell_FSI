# Les Signaux Unix

### 1. Qu'est-ce qu'un Signal ?

Imagine que ton programme (le processus) est un étudiant en train de passer un examen très concentré, avec un casque anti-bruit.

Il lit son code ligne par ligne (l'exécution normale).

Soudain, le surveillant (le Système d'Exploitation / l'OS) vient lui taper sur l'épaule. **C'est ça, un signal.**

C'est une **notification asynchrone**. Ça veut dire qu'elle peut arriver _n'importe quand_, interrompant brutalement ce que faisait le programme.

Quand le programme reçoit la tape sur l'épaule, il met son travail en pause, exécute une action spécifique liée au signal, puis (s'il survit) reprend exactement là où il en était.

---

### 2. Le Top 5 des Signaux

Il existe une trentaine de signaux sous Unix. Voici ceux qu'on croisera le plus souvent :

|**Nom**|**ID**|**Qui l'envoie ?**|**Que signifie-t-il ?**|**Action par défaut**|
|---|---|---|---|---|
|**`SIGINT`**|2|Clavier (`Ctrl-C`)|"Interruption ! Arrête ce que tu fais."|Termine le processus.|
|**`SIGQUIT`**|3|Clavier (`Ctrl-\`)|"Quitte et fais un rapport (Core Dump)."|Termine + Fichier core.|
|**`SIGKILL`**|9|Système (`kill -9`)|**L'Assassin.** "Meurs immédiatement."|**Termine. Impossible à bloquer.**|
|**`SIGSEGV`**|11|Système (Erreur RAM)|"Erreur de Segmentation." Tu as touché une mémoire interdite.|Crash (Core Dump).|
|**`SIGTERM`**|15|Système (`kill`)|"Terminaison." Demande polie de quitter.|Termine le processus proprement.|

**Le saviez-vous ?** On ne peut **PAS** intercepter ou ignorer `SIGKILL` (9). C'est la sécurité de l'OS pour pouvoir toujours tuer un programme qui a planté.

---

### 3. Les 3 choix du développeur

Face à un signal interceptable (comme `SIGINT`), le programme a 3 choix de réaction. C'est ce qu'on appelle la **disposition** du signal.

1. **Le comportement par défaut (`SIG_DFL`) :** Tu laisses l'OS gérer. S'il reçoit `Ctrl-C`, il meurt.

2. **L'ignorance (`SIG_IGN`) :** Tu dis à l'OS "Si je reçois ça, jette-le à la poubelle, je m'en fiche." (C'est ce qu'on fait pour `Ctrl-\` dans notre minishell).

3. **Le Handler :** Tu dis à l'OS "Si je reçois ça, ne me tue pas, exécute plutôt CETTE fonction C que j'ai écrite."


---

### 4. `signal()` vs `sigaction()` : Le Dududududu DUEL

En C, il y a deux fonctions pour changer le comportement d'un signal. Mais, il vaut mieux utiliser `sigaction`.

- ❌ **`signal()` :**

    - _Avantage :_ Très facile à écrire en une ligne : `signal(SIGINT, ma_fonction);`

    - _Inconvénient :_ Elle n'est pas "fiable" sur tous les systèmes d'exploitation. Parfois, après avoir reçu le signal une fois, elle se remet par défaut. Elle gère mal le fait de recevoir plusieurs signaux en même temps.

- ✅ **`sigaction()` :**

    - _Avantage :_ Permet de bloquer d'autres signaux pendant qu'on gère le premier (le "masque").

    - _Inconvénient :_ C'est une structure entière à remplir, ça prend 5 lignes.


---

### 5. À quoi ça ressemble ?

On va avoir notre fonction personnalisée qui va initialiser notre signal, quand on recevra un SIGINT, notre fonction écrira directement ce qu'il doit écrire (Ou dans un cas réel, fermera tout ce qu'il faut fermer ?) !

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

---

### 6. Les variables globales

C'est LA grande difficulté des signaux.

Puisque le signal peut te couper au milieu de n'importe quelle fonction (même pendant un `malloc` !), ta fonction "Handler" ne peut prendre aucun argument à part le numéro du signal.

**Comment communiquer entre le Handler et le reste de notre programme (ex: pour dire que `$?` vaut maintenant 130 après un `Ctrl-C`) ?**

Pour cela, on nous autorise une variable globale et elle nous servira seulement pour les signaux, le code erreur ou le code du signal reçu.

On va juste utiliser un `int signal_status = 0;`

---
---

Si on veut coder "proprement" (même si ça nous servira pas sur minishell), il faut comprendre pourquoi la norme POSIX (la Bible d'Unix) recommande l'utilisation d'un **`volatile sig_atomic_t`** à la place de notre `int`.

Voici l'explication simple de ces deux mots barbares :p.

### 1. `sig_atomic_t`

En informatique, une opération **"atomique"**, c'est une action qui est **impossible à couper en deux**. Elle se fait en un seul "tic" du processeur.

- **Le problème avec un `int` normal :** Sur certains vieux systèmes ou certaines architectures, écrire dans un gros nombre entier peut prendre 2 ou 3 cycles d'horloge au processeur.

    - _Imagine :_ Ton programme est en train d'écrire la première moitié du nombre dans la RAM... BAM ! Un signal `Ctrl-C` arrive, interrompt l'écriture, et le Handler modifie la variable. Quand le programme reprend, il écrit la deuxième moitié de l'ancien nombre. Résultat : ta variable contient un mix des deux, c'est corrompu (un _torn read/write_).

- **La solution `sig_atomic_t` :** C'est un type spécial (souvent juste un `int` déguisé par le système) qui te **garantit** que la lecture et l'écriture se feront en 1 seul coup. Le signal ne pourra jamais t'interrompre "au milieu" de l'écriture.

### 2. `volatile`

Celui-là c'est pour éviter les bugs incompréhensibles.

Ton compilateur (`gcc`) est très intelligent... Quand il compile ton code, il essaie de l'accélérer. Si tu as une boucle comme ça :

```
int g_status = 0;

while (g_status == 0) {
    // attendre...
}
```

`gcc` se dit : _"Cet idiot n'a mis aucun code qui modifie `g_status` dans sa boucle. Pour aller plus vite, je vais copier `g_status` dans la mémoire ultra-rapide du processeur (un registre), et je ne lirai plus jamais la vraie RAM."_

- **Le drame :** Tu fais `Ctrl-C`. Ton Handler de signal modifie `g_status = 130` dans la RAM. Mais ta boucle infinie continue de tourner, car elle regarde la vieille copie `0` dans son registre ! Ton programme a "freeze".

- **La solution `volatile` :** Ce mot-clé dit à `gcc` : _"Attention ! Cette variable peut être modifiée par un fantôme de l'extérieur (le signal) à tout moment. N'essaie pas de l'optimiser. Tu DOIS aller lire sa vraie valeur dans la RAM à chaque fois."_


---

### La réalité pour notre minishell

Avec les flags de compilation basiques (`-Wall -Wextra -Werror`), `gcc` ne fait pas d'optimisations assez agressives pour que le bug du `volatile` apparaisse (en tous cas, pas assez souvent). Et les processeurs modernes écrivent les `int` de façon atomique.

Donc :

- **Si on met `int g_status = 0;` :** Ça marchera.

- **Si on met `volatile sig_atomic_t g_status = 0;` :** On aura un code blindé, conforme aux normes industrielles.

_(Pour l'utiliser, il faut juste `#include <signal.h>`)._

---
