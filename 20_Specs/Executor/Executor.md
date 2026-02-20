
#### Notre récursivité dans l'Executor

```
int exec_node(t_ast_node *node)
{
    int status;

	status = 0;
    if (!node)
        return (0);
    if (node->type == NODE_AND)
    {
        status = exec_node(node->left);
        if (status == 0)
            status = exec_node(node->right);
        return (status);
    }
    else if (node->type == NODE_OR)
    {
        status = exec_node(node->left);
        if (status != 0)
            status = exec_node(node->right);
        return (status);
    }
    else if (node->type == NODE_PIPE)
    {
        return (exec_pipe_node(node));
    }
    else if (node->type == NODE_CMD)
    {
        return (exec_simple_cmd(node));
    }
    return (127);
}
```

---
#### L'exécution complèxe (`exec_pipe_node`)


```
int exec_pipe_node(t_ast_node *node)
{
    int     fd[2];
    pid_t   pid_left;
    pid_t   pid_right;
    int     status_left;
    int     status_right;

    if (pipe(fd) == -1)
        return (perror("pipe"), 1);
    pid_left = fork();
    if (pid_left == 0)
    {
        close(fd[0]);
        dup2(fd[1], STDOUT_FILENO);
        close(fd[1]);
        exit(exec_node(node->left)); 
    }
    pid_right = fork();
    if (pid_right == 0)
    {
        close(fd[1]);
        dup2(fd[0], STDIN_FILENO);
        close(fd[0]);
        exit(exec_node(node->right));
    }

    // 4. Le Parent (minishell) reprend le contrôle
    // IL DOIT FERMER SES PROPRES FDS DU PIPE !
    close(fd[0]);
    close(fd[1]);
    waitpid(pid_left, &status_left, 0);
    waitpid(pid_right, &status_right, 0);
    if (WIFEXITED(status_right))
        return (WEXITSTATUS(status_right));
    return (1);
}
```

---
#### L'Exécution Simple (`exec_simple_cmd`)

Le parcours de notre liste chaînée :

```
int apply_redirections(t_redir *redir_list)
{
    t_redir *current;
    int     fd;

	current = redir_list
    while (current)
    {
        if (current->type == REDIR_IN)
        {
            fd = open(current->file, O_RDONLY);
            if (fd == -1) return (perror("open"), 1);
            dup2(fd, STDIN_FILENO);
            close(fd);
        }
        else if (current->type == REDIR_OUT)
        {
            fd = open(current->file, O_WRONLY | O_CREAT | O_TRUNC, 0644);
            if (fd == -1) return (perror("open"), 1);
            dup2(fd, STDOUT_FILENO);
            close(fd);
        }
        // Pareil pour REDIR_APPEND (>>), mais avec O_APPEND au lieu de O_TRUNC
        ...
        current = current->next;
    }
    return (0);
}
```

---

Notre chasse au trésor :

```
char *find_executable(char *cmd, char **env)
{
    char **paths;
    char *half_path;
    char *full_path;
    int  i;
    
	i = 0;
	paths = ft_split(my_getenv("PATH", env), ':');
    if (strchr(cmd, '/'))
        return (strdup(cmd));
    while (paths && paths[i])
    {
	    half_path = ft_strjoin(paths[i], )
        full_path = ft_strjoin_three(paths[i], "/", cmd); 
        if (access(full_path, X_OK) == 0)
        {
            free_split(paths);
            return (full_path);
        }
        free(full_path);
        i++;
    }
    free_split(paths);
    return (NULL);
}
```

---

Et du coup, notre "squelette"...

```
int exec_simple_cmd(t_ast_node *node, char **env)
{
    pid_t pid;
    int   status;
    char  *cmd_path;

    if (!node->args || !node->args[0])
        return (apply_redirections(node->redirs));
    if (is_builtin(node->args[0]) == 1)
        return (exec_builtin(node));
    pid = fork();
    if (pid == 0) // --- ENFANT ---
    {
        if (apply_redirections(node->redirs) != 0)
            exit(1);
        cmd_path = find_executable(node->args[0], env);
        if (!cmd_path)
        {
            fprintf(stderr, "minishell_from_stark_industries: %s: command not found\n", node->args[0]);
            exit(127);
        }
        execve(cmd_path, node->args, env);
        perror("execve");
        exit(126);
    }
    // --- PARENT ---
    // On attend que la commande finisse et on renvoie son statut ($?)
    waitpid(pid, &status, 0);
    if (WIFEXITED(status))
        return (WEXITSTATUS(status));
    return (1);
}
```
