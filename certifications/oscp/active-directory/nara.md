# Nara

## Rapport de Pentest pour Nara

### Informations de base

* **Nom de la box**: Nara
* **Adresse IP**: 192.168.169.30
* **Système d'exploitation**: Linux
* **Date**: 3/25/2025

### Phases de Pentest

#### 1. Reconnaissance

* Scan Nmap initial: `nmap -sC -sV -oA initial 192.168.169.30`
* Scan complet: `nmap -p- --min-rate 10000 -oA full 192.168.169.30`
* Vérification des versions des services découverts

<figure><img src="../../../.gitbook/assets/image (307).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (308).png" alt=""><figcaption></figcaption></figure>

#### 2. Énumération

* Énumération des utilisateurs, groupes, et autres informations pertinentes
* Recherche de vulnérabilités connues pour les services identifiés

I have access to a share:

<figure><img src="../../../.gitbook/assets/image (309).png" alt=""><figcaption></figcaption></figure>

And i am able to get a file called important.txt:

<figure><img src="../../../.gitbook/assets/image (310).png" alt=""><figcaption></figcaption></figure>

Ok so nothing too crazy, but i have write permissions on the share so i'll try exploiting that:











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

### Services Détectés

_Pour ajouter des détails sur les services spécifiques, utilisez la page des Templates d'Exploitation._
