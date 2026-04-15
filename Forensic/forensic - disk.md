
## Procédure de montage

### Prérequis

```bash
apt install ewf-tools ntfs-3g kpartx
```

### 1. Lire les métadonnées de l'image
Le fichier **.E01** est un format d'image disque utilisé principalement par le logiciel **EnCase Forensic** pour l'analyse médico-légale et la sécurisation des preuves numériques.  Il s'agit d'une **image de volume ou de disque** chiffré et compressée au format binaire, souvent désignée sous le nom de **Expert Witness Format (EWF)**, qui permet de stocker une copie exacte du contenu d'un support de stockage.
Les fichiers E01 sont conçus pour être **inviolables et non modifiables**, garantissant que la chaîne de preuve reste intacte lors des enquêtes.

```bash
cat PHILIP.E01.txt
# ou
ewfinfo PHILIP.E01
```

mount avec .bin
```
losetup -f --show -o $((2048*512)) dev0_lba0_104857600.bin
```

Mount avec E01
```bash
mkdir -p /mnt/ewf
ewfmount PHILIP.E01 /mnt/ewf
# → Crée /mnt/ewf/ewf1
```

Vérification :
```bash
ls -lh /mnt/ewf/
# -r--r--r-- 1 root root 50G  ewf1
```

### 3. Analyser la table des partitions

```bash
fdisk -l /mnt/ewf/ewf1
```

**Output :**

```
Disk /mnt/ewf/ewf1: 50 GiB, 53687091200 bytes, 104857600 sectors
Disklabel type: dos  |  Disk identifier: 0xaba25ff7

Device           Boot  Start       End     Sectors  Size  Id  Type
/mnt/ewf/ewf1p1  *      2048  104855551  104853504   50G   7  HPFS/NTFS/exFAT
```

> **Offset partition :** Start = 2048 secteurs × 512 bytes = **1 048 576 bytes**

### 4. Créer un loop device sur la partition

```bash
losetup -f --show -o 1048576 /mnt/ewf/ewf1
# → /dev/loop0
```

### 5. Monter la partition NTFS en lecture seule

```bash
mkdir -p /mnt/disk
ntfs-3g -o ro,noatime /dev/loop0 /mnt/disk
```

Vérification :

```bash
ls /mnt/disk
# $Recycle.Bin  Boot  Documents and Settings  inetpub
# ProgramData  Program Files  Program Files (x86)
# Users  Windows  pagefile.sys  swapfile.sys ...
```

---

## 🔓 Démontage propre

```bash
umount /mnt/disk
losetup -d /dev/loop0
umount /mnt/ewf
```

---

## Commandes d'investigation

### Fichiers supprimés (Sleuth Kit)
 `*` = fichiers supprimés  
 `-r` = récursif  
 `-o 2048` = offset (debut de la partition)

```bash
fls -r -o 2048 /mnt/ewf/ewf1 | grep "^\*"
# Récupérer un fichier par inode :
icat -o 2048 /mnt/ewf/ewf1 <inode> > /tmp/recovered_file
```

### Lister TOUS les fichiers (avec métadonnées)
Utile pour :
- timeline
- reconstruction d’arborescence
- recherche avancée

```
fls -r -m / -o 2048 /mnt/ewf/ewf1 > fls_full.txt
```

### Timeline MFT
reconstruction chronologique des activités liées aux fichiers sur un système NTFS, basée sur les métadonnées stockées dans la **Master File Table (MFT)**.  Cette table contient des entrées pour chaque fichier et dossier, incluant plusieurs horodatages critiques :

- **M** (Modification) : dernière modification du contenu du fichier. 
- **A** (Accès) : dernière lecture du fichier.
- **C** (Changement) : dernière modification des métadonnées (ex. permissions).
- **B** (Création) : date de création du fichier.

```bash
fls -m "/" -o 2048 /mnt/ewf/ewf1 > /tmp/body.txt
mactime -b /tmp/body.txt -d > /tmp/timeline.csv
```

### Event logs Windows

```bash
evtx_dump /mnt/disk/Windows/System32/winevt/Logs/Security.evtx
evtx_dump /mnt/disk/Windows/System32/winevt/Logs/System.evtx
```

### Registry — Hash dump

```bash
impacket-secretsdump \
  -sam /mnt/disk/Windows/System32/config/SAM \
  -system /mnt/disk/Windows/System32/config/SYSTEM \
  LOCAL
```

## Application

```
/mnt/disk/Users/USER/AppData/Roaming/
```

### Logs IIS

```bash
ls /mnt/disk/inetpub/logs/LogFiles/
```

### Prefetch (programmes exécutés)
Les fichiers **.pf** sont des **fichiers Prefetch** utilisés par Windows pour améliorer les temps de démarrage des applications.  En forensic, ils constituent un artefact clé pour prouver l'exécution de programmes, même supprimés. 

Ils sont stockés dans `C:\Windows\Prefetch`

```bash
ls /mnt/disk/Windows/Prefetch/
```

## Browser Forensics (Windows)

Les données des navigateurs sont généralement stockées dans :

```
/mnt/disk/Users/<USER>/AppData/Local/  
/mnt/disk/Users/<USER>/AppData/Roaming/
```
---

## Navigateurs principaux

###  Microsoft Edge (Chromium)
```
/mnt/disk/Users/<USER>/AppData/Local/Microsoft/Edge/User Data/Default/
```

---
###  Google Chrome
```
/mnt/disk/Users/<USER>/AppData/Local/Google/Chrome/User Data/Default/
```

---
## Mozilla Firefox
```
/mnt/disk/Users/<USER>/AppData/Roaming/Mozilla/Firefox/Profiles/
```
---
## Fichiers importants à extraire

## Historique de navigation

- `History` (Chrome / Edge)
- `places.sqlite` (Firefox)
```
cp "<path>/History" /tmp/history.db
```

---
## Cookies

- `Cookies` (Chrome / Edge)
- inclus :
    - sessions
    - tokens d’authentification

---
## Téléchargements

Dans `History` (table `downloads`)

---

## Mots de passe

- `Login Data` (chiffré)  
-  nécessite DPAPI / master key

---
## Cache

Cache/  
Code Cache/

 Peut contenir :

- fichiers téléchargés
- fragments HTML/JS

---
## Sessions / onglets ouverts

Current Session  
Current Tabs

---

##  Analyse SQLite (Chromium)

###  Historique
```
sqlite3 history.db "  
SELECT   
  datetime(last_visit_time/1000000-11644473600,'unixepoch') AS visit_time,  
  url,  
  title,  
  visit_count  
FROM urls  
ORDER BY last_visit_time DESC  
LIMIT 50;"
```

## Firefox
```
sqlite3 places.sqlite "  
SELECT  
datetime(visit_date/1000000,'unixepoch') AS visit_time,  
url,  
title,  
visit_count  
FROM moz_places  
JOIN moz_historyvisits  
ON moz_places.id = moz_historyvisits.place_id  
ORDER BY visit_date DESC  
LIMIT 50;"
```

---
### Téléchargements
```
sqlite3 history.db "  
SELECT   
  datetime(start_time/1000000-11644473600,'unixepoch'),  
  target_path,  
  tab_url  
FROM downloads;"
```

```bash
for profil in jbg01wnx.default w0bjz8mn.default-release-1 zrvdobv7.default-release; do echo "=== $profil ===" cp "/mnt/disk/Users/PHIL/AppData/Roaming/Mozilla/Firefox/Profiles/$profil/places.sqlite" /tmp/ff_tmp.db 2>/dev/null sqlite3 /tmp/ff_tmp.db "SELECT datetime(h.visit_date/1000000,'unixepoch'), p.url FROM moz_historyvisits h JOIN moz_places p ON h.place_id = p.id WHERE p.url LIKE '%q=%' OR p.url LIKE '%search%' ORDER BY h.visit_date DESC LIMIT 30;" 2>/dev/null done
```

Credentials:
```
python3 firefox_decrypt.py --no-interactive --choice 1 \
  "/mnt/disk/Users/PHIL/AppData/Roaming/Mozilla/Firefox/"
```

