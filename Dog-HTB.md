# DOG machine - HTB LABS

We have 2 port open
- 22
- 80 

on the port 80 we have CMS backdrop with 2 pages:
- Login
- About

About page contains only images & text, so nothing to exploit. But Login contains 2 input (email & password) so we can login after get some creds

if you Fuzz directory we can discover "/.git", so go to upload all source code:


Wget all files on .git:
```
wget -I .git -r http://10.10.11.58/.git
```

get the path of Master Hash:
```
cat HEAD
```

Get .zip of source code:
```
git archive --format zip --output "source.zip" <MASTER HASH>
```

After all it, we can explore some files and in settings.php at the racine we can view 

```
$database = 'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop';
```

After get the password, now go found the user in "cat 10.10.11.58/10.10.11.58/.git/dog/files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/update.settings.json 
```
{
    "_config_name": "update.settings",
    "_config_static": true,
    "update_cron": 1,
    "update_disabled_extensions": 0,
    "update_interval_days": 0,
    "update_url": "",
    "update_not_implemented_url": "https://github.com/backdrop-ops/backdropcms.org/issues/22",
    "update_max_attempts": 2,
    "update_timeout": 30,
    "update_emails": [
        "tiffany@dog.htb"
    ],
    "update_threshold": "all",
    "update_requirement_type": 0,
    "update_status": [],
    "update_projects": []
}
```

In the Website we can get the version of the CMS BackDrop: 1.27.1 and this version contains critical Authenticated RCE:
"https://www.exploit-db.com/exploits/52021"

to launch the RCE go execute with python3 and the code generate file zip with:
- shell.info
- shell.php

Now upload your TAR file in upload page on th website and go in this url:
"http://10.10.11.58/modules/shell/shell.php"

Now in the web shell check the /home/user and we can see this user:
- johncusack

try to connect on ssh with this user and the same password of tiffany email
```
ssh johncusack@10.10.11.58
```

USER.TXT IS FLAGGED !



