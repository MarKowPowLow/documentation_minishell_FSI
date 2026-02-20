#### Structure pour notre liste chaînée


```
typedef struct s_env {
    char            *key;   // Ex: "USER"
    char            *value; // Ex: "Jarvis" (Peut être NULL si export sans '=')
    struct s_env    *next;
} t_env;
```


