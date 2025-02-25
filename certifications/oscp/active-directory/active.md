# 📦 Active

I start off by scanning open ports and quickly see that this is a AD, with domain active.htb, then i try to look for SMB shares:

<figure><img src="../../../.gitbook/assets/image (12) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then i download everything in that share:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I end up finding a groups.xml file:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/credential-access/unsecured-credentials/group-policy-preferences/gpp-password" %}

I take the hash of the password and run ggpp-decrypt on it:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>active.htb\SVC_TGS:GPPstillStandingStrong2k18</p></figcaption></figure>

i'm able to get other users with those credentials:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Using the credentials i'm able to kerberoast the Administrator hash:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and i'm able to crack the hash just like that:

```
hashcat -m13100 krb5hash.txt /usr/share/wordlists/rockyou.txt.gz
```

and connect to the admin machine with psexec:

```
psexec.py active.htb/Administrator:Ticketmaster1968@10.129.231.94
```

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
