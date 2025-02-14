# vmdak

i start off by scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

The port 9443 says it's http but there is a SSL problem so we need to add the domain in etc hosts and request the website using https and then we see this:

<figure><img src="../../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

I look at some known exploits for this framework, click on admin dashboard and see this:

{% embed url="https://www.exploit-db.com/exploits/52017" %}

so i try it out:

<figure><img src="../../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

and i manage to get in:

<figure><img src="../../../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>

the box is very laggy and i have to wait 5min + for one request so i will come back later
