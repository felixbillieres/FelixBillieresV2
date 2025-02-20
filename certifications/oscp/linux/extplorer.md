---
description: Community Rating:Intermediate
---

# Extplorer

First we start by scanning the ip for open ports:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Visiting the webpage we see a wordpress website that is not initialized:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

when i enter dummy credentials i get this error:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then i run a dirb:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

&#x20;and discover this directory:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

with default admin:admin credentials we can log in and i quickly see the upload feature since there are some php pages maybe i can upload a reverse shell:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i upload a reverse shell and visit the file to run it:

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><a href="http://192.168.54.16/reverse.php">http://192.168.54.16/reverse.php</a></p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So now i elevate my shell:

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Then i navigate to the home directory:

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we need to find dora's password or root's password to continue so i enumerate for passwords and end up in the filemanager config files:

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So there isn't the root password but something catches my eye:

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

hidden files with .ht\*

and in the same directory we find the following files:

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1).png" alt=""><figcaption><p>.htusers.php</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
john dorahash --wordlist=/usr/share/wordlists/rockyou.txt.gz
```

This gives us a password to switch user to dora ->

<figure><img src="../../../.gitbook/assets/image (15) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then i run linpeas as dora and find the following information:

<figure><img src="../../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

Being in the `disk` group grants direct access to raw storage devices (`/dev/sdX`, `/dev/nvmeX`). This allows an attacker to:

* **Read all data** on the disk, bypassing permissions.
* **Modify or delete system files**, leading to corruption or privilege escalation.
* **Overwrite boot sectors** to install rootkits or backdoors.

{% embed url="https://www.hackingarticles.in/disk-group-privilege-escalation/" %}

So i start of by looking at what partition i need to exploit:

<figure><img src="../../../.gitbook/assets/image (17) (1).png" alt=""><figcaption><p>/dev/mapper/ubuntu--vg-ubuntu--lv</p></figcaption></figure>

and just like that i am able to retrieve root flag:

<figure><img src="../../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

* The `disk` group allows direct **block-level access** to the storage device.
* `debugfs` doesn’t rely on Linux file permissions; instead, it directly reads filesystem metadata and contents.
* Even though `/root/proof.txt` is owned by `root`, `dora` can **read it at a lower level**, completely bypassing normal restrictions.
