**`sigaction`** : Définit comment réagir à un signal (moderne).

```
struct sigaction sa;
sa.sa_handler = ma_fonction_handler;
sigemptyset(&sa.sa_mask);
sa.sa_flags = SA_RESTART;
sigaction(SIGINT, &sa, NULL); // Intercepte Ctrl-C
```