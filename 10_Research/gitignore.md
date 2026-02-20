# Le `.gitignore`

### 1. À quoi ça sert ?

Quand tu fais `git add .` ou que tu cliques sur le `+` dans VS Code, Git veut prendre  tous les fichiers de ton dossier. Mais tu ne dois **absolument pas** envoyer les fichiers compilés (`.o`, l'exécutable...) sur GitHub. D'une part, ça pollue le dépôt, et d'autre part, les sujets ne le demandent pas généralement :D.

Le `.gitignore` est un fichier texte caché à la racine de ton projet qui dit à Git : _"Ignore ces fichiers, fais comme s'ils n'existaient pas."_

### 2. Pour notre minishell

Crée un fichier nommé exactement `.gitignore` à la racine de ton projet et colle ceci :

```
# Fichiers objets (compilés)
*.o
# Si c'est dans des dossiers ou sous-dossiers...
*/*.o
*/*/*.o
...

# L'exécutable final
minishell_from_stark_industries

# Fichiers de configuration locaux de VS Code (optionnel)
.vscode/

# Fichier de suppression Valgrind (qu'on a vu dans la partie Debug)
*.supp
```

Dès que ce fichier est sauvegardé, tous tes `.o` deviendront grisés dans VS Code et Git arrêtera de te proposer de les commit !