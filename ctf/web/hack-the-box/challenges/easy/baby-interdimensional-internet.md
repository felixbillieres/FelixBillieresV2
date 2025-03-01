# baby interdimensional internet

So we start off with a simple page that retrieves random numbers when we refresh:

<figure><img src="../../../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

When looking at the source code we see that there is a hidden directory, /debug that we can maybe use?

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The /debug is a flask app and we can read the source code:

<figure><img src="../../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

First thing i see is i can send POST requests to the rick & morty landing page:

<figure><img src="../../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The next thing that stands out is the exec function:

<figure><img src="../../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The `exec` function allows you to dynamically execute Python code that is passed as a string. Here it executes the Python code contained in `recipe` within the context of `garage and` if an error occurs during execution, the exception is ignored using `try/except`.

Here we see that in order to inject in recipe, we need to post 2 variables:

<figure><img src="../../../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i capture a request after refreshing the landing page, i send it to the repeater and change the Method from GET to POST and input the following&#x20;

```sh
ingredient=leak&measurements=__import__('os').popen('cat /etc/passwd').read()
```

<figure><img src="../../../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And from there we have code execution and can enumerate the system to find the flag:

<figure><img src="../../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
