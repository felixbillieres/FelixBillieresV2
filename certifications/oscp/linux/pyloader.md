# pyLoader

I start off by scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

We can see a version in the header that seems interesting:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

that fills every aspect of this box, let's try to get an RCE ->

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

ok so maybe we can't see the output, so i tried a reverse shell but it still did not work, so i took a step back and continued enumeration with default creds:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and with those creds i'm able to log in:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In the info section we find interesting informations:

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i tried exploiting an exploit on pyLoad 0.5.0

{% embed url="https://github.com/JacobEbben/CVE-2023-0297" %}

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

ah and we spawn a root shell, nice gg well play
