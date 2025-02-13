# Pelican

i start off by scanning open ports ->

<figure><img src="../../../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

so there are plenty of interesting stuff here, i start off by looking at SMB shares but no permissions:

<figure><img src="../../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

So i look at the HTTP pages:

<figure><img src="../../../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

What catches my eye is Exhibitor for zookeeper v1.0, i looked at Zookeeper 3.4.6 exploits since it whats in nmap and i found that:

<figure><img src="../../../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

and i get a shell:

<figure><img src="../../../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

So i'm allowd to gcore as sudo:

<figure><img src="../../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

i find this article:

{% embed url="https://wiki.sentnl.io/security/hacking-demos/getting-passwords-of-logged-in-users" %}

So i tried seeing if root user had a password in his session by dumping his session with gcore using this document but it did not work so i launched pspy to see maybe other PIDs of apps that could be interesting -> but we only find wierd outputs

So 30min+ passed here, i was trying to see every PID that i could dump with gcore and found a root one that was interesting:

<figure><img src="../../../.gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>

so i dumped the process with gcore:

```
sudo /usr/bin/gcore 494
strings core.494
```

and retrieved the password:

<figure><img src="../../../.gitbook/assets/image (124).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>
