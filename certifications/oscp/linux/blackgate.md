# BlackGate

I start my enumeration and see a Redis service running with a version number:

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

There was not much going on on the webpage so i started looking for known vulns:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and came accross this POC:

{% embed url="https://github.com/Ridter/redis-rce" %}

and i was able to exploit this poc and get a reverse shell:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then i used the following command to get an upgraded shell:

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

{% hint style="warning" %}
Machine broke and redis service froze when i tried enumerating for privesc; will come back later to root it
{% endhint %}
