---
description: >-
  You have found a portal of the recently arising tornado malware, it appears to
  have some protections implemented but a bet was made between your peers that
  they are not enough. Will you win this bet?
---

# TornadoService

So when we go and look at the web service we come accross a Tornado list with some Ips, a status set to active/inactiv and some features such as updating status or report ip:

<figure><img src="../../../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

After looking at the features i go and read the source code of the application and spot the main details that pop to my eye:

<figure><img src="../../../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (18) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (19) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (20) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (21) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (22) (1) (1).png" alt=""><figcaption></figcaption></figure>

Those are the informations that seemed important to me on the first quick look.

