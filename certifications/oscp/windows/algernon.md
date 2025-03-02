# Algernon

I start off by scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I start by enumerating ftp anonymous login and try directory bruteforcing on the http instances and come accross this smartermail instance on port 9998 ->

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I go back to the ftp anonymous download eveyrthing and look at some files  in the Logs folder and see some administrative logs:

command to download everything in FTP:

```
wget -r ftp://anonymous:password@192.168.51.65
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I thought i needed to find some creds in the logs but nothing seemed to pop out so i wen to look at some smartermail exploits ->

{% embed url="https://www.exploit-db.com/exploits/49216" %}

I change the parameters and run the exploit and end up getting a shell:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

ok nice haha not too complicated
