# Nickel

<figure><img src="../../../.gitbook/assets/image (157).png" alt=""><figcaption></figcaption></figure>

i look at ftp anonymous ande rpcclient but i did not find anything so i look at the http server:

<figure><img src="../../../.gitbook/assets/image (158).png" alt=""><figcaption></figcaption></figure>

ok so for some reason i found what i was looking&#x20;

The source code of this code had hardcoded endpoints and IPs so i started looking at it, curling them and trying to see what i could do:

<figure><img src="../../../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

i saw a redirect from the hardcoded ip to the target ip so i tried curling the service and enpoint with our target ip and somehow made it work

<figure><img src="../../../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

and found this:

<figure><img src="../../../.gitbook/assets/image (161).png" alt=""><figcaption><p>ariah:Tm93aXNlU2xvb3BUaGVvcnkxMzkK</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (162).png" alt=""><figcaption><p>NowiseSloopTheory139</p></figcaption></figure>

and i manage to get a shell just like that:

<figure><img src="../../../.gitbook/assets/image (163).png" alt=""><figcaption></figcaption></figure>

