
#### Notre structure pour les tokens (Et notre enum de type)

```
typedef enum e_type {
    WORD,
    PIPE,
    REDIR_IN,
    REDIR_OUT,
    HEREDOC,
    APPEND,
    EOF
} t_type;

typedef struct s_token {
    t_type          type;
    char            *value;
    struct s_token  *prev;
    struct s_token  *next;
} t_token;
```


---

#### Notre structure pour nos modes (Et un peu de code et l'enum)

```
typedef enum e_state {
    STATE_GENERAL,
    STATE_IN_DQUOTE,
    STATE_IN_SQUOTE
} t_state;

//Dans la fn
t_state state;
int i;

i = 0;
state = STATE_GENERAL;
while (str[i])
{
    if (str[i] == '\'' && state == STATE_GENERAL)
        state = STATE_IN_SQUOTE;
    else if (str[i] == '\'' && state == STATE_IN_SQUOTE)
        state = STATE_GENERAL;
    else if (str[i] == '"' && state == STATE_GENERAL)
        state = STATE_IN_DQUOTE;
    else if (str[i] == '"' && state == STATE_IN_DQUOTE)
        state = STATE_GENERAL;
    if (state == STATE_GENERAL && str[i] == ' ')
    {
        create_new_token();
    }
    else
    {
        add_to_word(str[i]);
    }
    i++;
}
if (state != STATE_GENERAL)
	syntax_error(!?!); // Il faudra tout clean proprement !
```
