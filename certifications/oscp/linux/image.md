# Image

I start off by a quick scan and see something running on port 80:

<figure><img src="../../../.gitbook/assets/image (272).png" alt=""><figcaption></figcaption></figure>

after uploading a random image i see the version number:

<figure><img src="../../../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

and i identify the payload that matches this version:

{% embed url="https://github.com/SudoIndividual/CVE-2023-34152/blob/main/CVE-2023-34152.py" %}

```sh
[Mar 13, 2025 - 21:35:34 (CET)] exegol-Offsec Image # python3 exploit.py                    
Usage: exploit.py Attacker_IP Attacker_Port
```

So its does not give us a reverse shell but creates a malicious png ifle that will give us a reverse shell:

<figure><img src="../../../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

So i upload the file and get a reverse shell:

<figure><img src="../../../.gitbook/assets/image (275).png" alt=""><figcaption></figcaption></figure>

I look for some nice suids and find some stuff:

<figure><img src="../../../.gitbook/assets/image (276).png" alt=""><figcaption></figcaption></figure>

nice:

<figure><img src="../../../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>
