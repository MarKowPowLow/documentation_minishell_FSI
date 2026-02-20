# **Le Lexer** (ou Analyseur Lexical).


### 1. Qu'est-ce qu'un Lexer en général ?

Imaginons qu'on doit analyser cette phrase en français : _"Le chat mange."_

Tu ne vas pas lire les lettres une à une, `L`, puis `e`, puis , puis `c`, etc... Tu vas regrouper les lettres pour former des mots, et on va donné une "étiquette" à chaque mot :

- `[Article : Le]`

- `[Nom : chat]`

- `[Verbe : mange]`


**Le Lexer fait exactement ça pour notre code.**

Il prend une longue chaîne de caractères brute (un `char *` en C) et la découpe en petits paquets logiques qu'on appelle des **tokens** (jetons).

**Pourquoi on a besoin de ça ?**

Parce que manipuler des chaînes de caractères avec des indices `i` et `j` partout, c'est l'enfer (Ou pas, regarde ta map de so_long :D). C'est beaucoup plus facile de dire à la suite de ton programme : _"Hé, j'ai reçu un jeton de type VERBE, suivi d'un jeton de type NOM"_, plutôt que _"J'ai reçu les lettres L, e, espace..."_.

---

### 2. Un Lexer pour notre Minishell

Le Bash a une grammaire très particulière. Le travail de notre Lexer va être de lire la ligne tapée par l'utilisateur et d'appliquer les règles de découpage du Shell.

Voici les **3 grandes règles** de notre Lexer :

#### Règle A : Les espaces séparent les mots

Si notre Lexer voit un espace (ou une tabulation), il sait que le jeton actuel est terminé et qu'un nouveau commence.

- Entrée : `ls -l`

- Sortie : `[MOT : "ls"]` ➔ `[MOT : "-l"]`


#### Règle B : Les symboles spéciaux se séparent tout seuls

Dans Bash, les métacaractères (`|`, `<`, `>`, `<<`, `>>`) n'ont pas besoin d'espaces autour d'eux pour être reconnus. Notre Lexer doit être capable de les isoler.

- Entrée : `ls>fichier` _(Remarque : il n'y a pas obligatoirement des espaces !)_

- Sortie : `[MOT : "ls"]` ➔ `[REDIRECTION_DROITE : ">"]` ➔ `[MOT : "fichier"]`


#### Règle C : Les Quotes (Guillemets) collent les mots

Si notre Lexer entre dans une quote (`'` ou `"`), **il ignore la Règle A**. Les espaces deviennent de simples lettres normales jusqu'à ce qu'il trouve la quote de fermeture.

- Entrée : `echo "bonjour le monde"`

- Sortie : `[MOT : "echo"]` ➔ `[MOT : "bonjour le monde"]` _(un seul token !)_


---

### 3. L'Exemple visuel complet

Imaginons que l'utilisateur tape cette commande (pas trop compliqué :p) :

`cat < file.txt | grep "erreur 404" > log`

Voici comment notre Lexer va transformer cette string en une liste chaînée de tokens :

| **Nœud** | **Type du Token** | **Valeur stockée** |
| -------- | ----------------- | ------------------ |
| 1        | `TOKEN_WORD`      | `"cat"`            |
| 2        | `TOKEN_LESS`      | `"<"`              |
| 3        | `TOKEN_WORD`      | `"file.txt"`       |
| 4        | `TOKEN_PIPE`      | `\|`               |
| 5        | `TOKEN_WORD`      | `"grep"`           |
| 6        | `TOKEN_WORD`      | `"erreur 404"`     |
| 7        | `TOKEN_GREAT`     | `">"`              |
| 8        | `TOKEN_WORD`      | `"log"`            |

---

### 4. À quoi ça ressemble ?

Tu l'auras bien compris avec le tableau juste avant, on aura besoin d'une structure pour stocker nos mots, le type de notre mot (et aussi le mot suivant, vu que ça sera une liste chaînée) et pour avoir le type de notre mot, quoi de plus simple que de tout lister dans une énumération ? :

```
typedef enum e_type {
    WORD,
    PIPE,      // |
    REDIR_IN,  // <
    REDIR_OUT, // >
    HEREDOC,   // <<
    APPEND     // >>
} t_type;

typedef struct s_token {
    t_type          type;   // Quelle est son type ?
    char            *value; // Quel est le texte exact ?
    struct s_token  *next;  // Pointeur vers le jeton suivant
} t_token;
```

Une fois que notre Lexer a fait ce travail, notre ligne de commande sera propre. On l'envoi ensuite au [[_Parser|Parser]], qui va regarder cette liste et dire : _"Ah, il y a un PIPE au milieu, je vais donc créer un [[AST]] avec une branche gauche et une branche droite !"_.

Par contre, comme le disait un certain papi magicien :

Cependant ! 

Ça, c'est pour les lignes "simples", il faudra aussi pouvoir gérer les quotes, `" "` et `' '`, pour ce faire, on va avoir besoin d'une "Machine à états" (Spoiler, c'est pas compliqué).

---

La machine à états (ou _Finite State Machine_ en anglais), c'est une façon élégante de donner une "mémoire" à ton Lexer pendant qu'il lit la ligne caractère par caractère. Pour éviter d'avoir un milliard de if/else, ça sera nécessaire.


### 1. Les "Modes" du Lexer

Imagine qu'un Lexer, c'est un personnage de jeu vidéo. Pendant qu'il avance sur la ligne de texte, il peut changer "d'arme" (d'état) en fonction de ce qu'il ramasse.

Chaque armes (une épée, un arc ou un bouclier :O)  lui donne des règles différentes pour réagir aux espaces et aux symboles.

Dans notre minishell, on aura besoin que de **3 états principaux** (vu que c'est limité à ceux-là dans le sujet) :

- 🟢 **L'état `GENERAL` (Mode par défaut) :**

    - _Règle :_ Si je vois un espace, je coupe le mot. Si je vois un `|`, je crée un token spécial.

- 🟡 **L'état `IN_DQUOTE` (Mode "Guillemets Doubles `"`") :**

    - _Règle :_ J'ignore les espaces et les `|`, ce sont de simples lettres. Par contre, le `$` fonctionne toujours (pour les variables).

- 🔴 **L'état `IN_SQUOTE` (Mode 'Guillemets Simples `'`') :**

    - _Règle :_ Je suis aveugle. Tout est du texte brut. Les espaces, les `|`, et même les `$` perdent leur pouvoir.


---

### 2. Comment on change d'état ?

Le principe de la machine à états est basé sur des "interrupteurs" (Comme ton ).

1. Tu commences toujours en `GENERAL`.

2. Tu lis la phrase lettre par lettre.

3. **Si tu rencontres une quote (`"`) :**

    - Es-tu en `GENERAL` ? ➔ Tu passes en `IN_DQUOTE`.

    - Es-tu DÉJÀ en `IN_DQUOTE` ? ➔ Tu repasses en `GENERAL` (tu as trouvé la fin de la quote).

    - Es-tu en `IN_SQUOTE` ? ➔ Tu l'ignores, c'est juste du texte.

### 3. L'Exemple pas-à-pas

Prenons une ligne un peu compliquée : `echo "hello | wc" '!'`

Voici ce qui se passe dans la machine à états :

| **Caractère(s) lu(s)**  | **État Actuel** | **Action du Lexer**                                    | **Nouvel État** |
| ----------------------- | --------------- | ------------------------------------------------------ | --------------- |
| `e`, `c`, `h`, `o`      | 🟢 `GENERAL`    | Ajoute au mot courant (`echo`).                        | 🟢 `GENERAL`    |
| (espace)                | 🟢 `GENERAL`    | **Coupe le mot** ➔ Token `[echo]` créé.                | 🟢 `GENERAL`    |
| `"`                     | 🟢 `GENERAL`    | **Bascule l'interrupteur !**                           | 🟡 `IN_DQUOTE`  |
| `h`, `e`, `l`, `l`, `o` | 🟡 `IN_DQUOTE`  | Ajoute au mot (`hello`).                               | 🟡 `IN_DQUOTE`  |
| (espace)                | 🟡 `IN_DQUOTE`  | Ajoute au mot (`hello` ). (On ne coupe pas !)          | 🟡 `IN_DQUOTE`  |
| `\|`                    | 🟡 `IN_DQUOTE`  | Ajoute au mot (`hello \|`). (Ce n'est pas un pipe ici) | 🟡 `IN_DQUOTE`  |
| (espace)                | 🟡 `IN_DQUOTE`  | Ajoute au mot (`hello \|` ).                           | 🟡 `IN_DQUOTE`  |
| `w`, `c`                | 🟡 `IN_DQUOTE`  | Ajoute au mot (`hello \| wc`).                         | 🟡 `IN_DQUOTE`  |
| `"`                     | 🟡 `IN_DQUOTE`  | **Bascule l'interrupteur (Fin).**                      | 🟢 `GENERAL`    |
| (espace)                | 🟢 `GENERAL`    | **Coupe le mot** ➔ Token `[hello \| wc]` créé.         | 🟢 `GENERAL`    |
| `'`                     | 🟢 `GENERAL`    | **Bascule l'interrupteur !**                           | 🔴 `IN_SQUOTE`  |
| `!`                     | 🔴 `IN_SQUOTE`  | Ajoute au mot (`!`).                                   | 🔴 `IN_SQUOTE`  |
| `'`                     | 🔴 `IN_SQUOTE`  | **Bascule l'interrupteur (Fin).**                      | 🟢 `GENERAL`    |
| `\0` (Fin de chaîne)    | 🟢 `GENERAL`    | **Coupe le mot final** ➔ Token `[!]` créé.             | 🟢 `GENERAL`    |

(Note : Il faudra probablement retirer les quotes `" "` et `' '` dans nos tokens (ou les skip), vu qu'ils ne font pas partis des mots.)

---

### 4. À quoi ça ressemble ?

Pour pouvoir utilisé notre machine à états simplements, on aura besoin d'une énumération de nos trois états, c'est aussi simple que ça ^^. Après, dans notre code, on vérifiera simplement notre état actuel ! :

```
typedef enum e_state {
    STATE_GENERAL, // Base
    STATE_IN_DQUOTE, // ""
    STATE_IN_SQUOTE // ''
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
        create_new_token(...);
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

 **Le cas d'Erreur (Syntax Error) :**
 
À la fin de ta boucle `while`, si l'état n'est **PAS** repassé sur `STATE_GENERAL` (par exemple, si on est toujours en `STATE_IN_DQUOTE`), ça veut dire que l'utilisateur a oublié de fermer ses guillemets ! On peut alors directement renvoyer une erreur de syntaxe.

---

