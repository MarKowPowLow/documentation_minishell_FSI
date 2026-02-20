### Pour l'architecture

Dans notre code, on aura une fonction qui nous servira d'aiguilleur dans notre Executor qui ressemblera à ça :

```
typedef struct s_builtin
{
    char    *cmd_name;
    int     (*func)(char **args, t_env **env_list);
} t_builtin;

Nos fonctions auront toujours le même patern :
int builtin_echo(char **args, t_env **env);
int builtin_cd(char **args, t_env **env);
int builtin_pwd(char **args, t_env **env);
// ... etc

int exec_builtin(char **args, t_env **env)
{
    int i = 0;
    
    t_builtin dispatch_table[] = {
        {"echo", &builtin_echo},
        {"cd", &builtin_cd},
        {"pwd", &builtin_pwd},
        {"export", &builtin_export},
        {"unset", &builtin_unset},
        {"env", &builtin_env},
        {"exit", &builtin_exit},
        {NULL, NULL} 
    };
    while (dispatch_table[i].cmd_name != NULL)
    {
        if (ft_strcmp(args[0], dispatch_table[i].cmd_name) == 0)
        {
            return (dispatch_table[i].func(args, env));
        }
        i++;
    }
    return (-1);
}

int is_builtin(char *cmd)
{
	int i;
	char *builtins[] = {"echo", "cd", "pwd", "export", "unset", "env", "exit", NULL};
	
	i = 0;
	if (!cmd)
		return (0);
	while (builtins[i])
	{
			if (ft_strcmp(cmd, builtins[i]) == 0)
				return (1);
			i++;
	}
	return (0);
}
```
