
#### Les structures de redirection et de nœud (Et les enums)

```
typedef enum e_node_type {
    NODE_CMD,
    NODE_PIPE,
    NODE_AND,
    NODE_OR
} t_node_type;

typedef enum e_redir_type {
    REDIR_IN,       // <
    REDIR_OUT,      // >
    REDIR_APPEND,   // >>
    REDIR_HEREDOC   // <<
} t_redir_type;

typedef struct s_redir {
    t_redir_type    type;
    char            *file;
    struct s_redir  *next;
} t_redir;

typedef struct s_cmd_node {
    char            **args;
    t_redir         *redirs;
} t_cmd_node;

typedef struct s_ast_node {
    t_node_type         type;
    // Si c'est PIPE, AND, OR
    struct s_ast_node   *left;
    struct s_ast_node   *right;
    // Si c'est une CMD
    t_cmd_data          cmd;
} t_ast_node;
```

---

#### Pour parcourir notre sous-liste de tokens ! 

```
t_cmd_node *parse_cmd(t_token **sub_list)
{
	t_redir_type type;
	char *file;
    t_cmd_node  *cmd;

    cmd = malloc(sizeof(t_cmd_node))
    cmd->redirs = NULL;
	// On boucle tant que notre sub_list existe et que c'est pas un opérateur de contrôle (&& & |)
    while (*sub_list && !is_control_operator(*sub_list))
    {
        if (is_redirection(*sub_list))
        {
            type = get_redir_type(*sub_list);
            if (!(*sub_list)->next)
	            return (error !?!) // Ça voudrait dire qu'il y a un chevron sans rien derrière !
            *sub_list = (*sub_list)->next; 
            file = (*sub_list)->value;
            add_redir_to_list(&(cmd->redirs), type, file);
        }
        else
        {
            add_arg_to_array(&(cmd->args), (*sub_list)->value);
        }
        if ((*sub_list)->next)
	        *sub_list = (*sub_list)->next;
    }
    return (cmd);
}
```

---

#### Pour le heredoc ?!?

```
void handle_heredoc(char *limiter, int expand_variables)
{
    int fd = open("/tmp/.minishell_from_stark_industries_heredoc", O_CREAT | O_WRONLY | O_TRUNC, 0644);
    char *line;

    while (1)
    {
        line = readline("> ");
        if (!line) 
        {
            print_warning("Heredoc terminé par EOF (Ctrl-D)");
            break;
        }
        if (strcmp(line, limiter) == 0)
        {
            free(line);
            break;
        }
        if (expand_variables == TRUE) (Si on nous a donné un EOF ou un "EOF")
            line = expand_string(line, env);
            
        write(fd, line, strlen(line));
        write(fd, "\n", 1);
        free(line);
    }
    close(fd);
}
```

---

#### Le code erreur #?

Côté Executor : 


```
int status;
waitpid(pid, &status, 0);

if (WIFEXITED(status))
    g_status = WEXITSTATUS(status);
else if (WIFSIGNALED(status))
    g_status = 128 + WTERMSIG(status);
```


Côté Expander :

```
if (nom_de_la_variable == '?')
{
    char *status_str;
    
    status_str = ft_itoa(g_status);
    ligne_finale = remplacer_str(ligne_originale, "$?", status_str);
    free(status_str);
}
```
