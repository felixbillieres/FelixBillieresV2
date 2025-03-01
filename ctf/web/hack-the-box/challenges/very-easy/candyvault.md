# CandyVault

We begin by landing on a simple login page. Upon inspecting the page's source code, we notice that it’s part of a Flask web application and uses MongoDB as its backend database. This provides an interesting starting point for our exploration, as we can start analyzing the request behavior to look for vulnerabilities.

<figure><img src="../../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In the `app.py` file, we can see two routes that catch our attention. These routes handle the user login functionality, and by analyzing them, we gain insight into how the application processes login requests. Specifically, we observe the following behavior:

* The application handles HTTP POST requests to the login form.
* The request content type can be set as `application/json`, which is important because it hints at the way the data is processed and allows us to craft a custom request with specific payloads.

<figure><img src="../../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We verify that the application is built using Flask, which is a lightweight Python framework for web applications. Additionally, it utilizes MongoDB as its database, which is a NoSQL database. Given that MongoDB is used to store the user credentials, this opens the door to potential NoSQL injection attacks, particularly around how data is queried and validated.

<figure><img src="../../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

At this point, I decided to look up potential MongoDB login bypass techniques. I came across some relevant documentation on HackTricks, but I found a more structured approach in the PortSwigger Web Security Academy exercises. These resources provided insights into bypassing MongoDB authentication mechanisms, which proved to be highly useful for crafting the next step in the exploit.

In particular, one interesting method from the exercises involves manipulating the MongoDB query to bypass authentication. The query is generally structured like this:

<figure><img src="../../../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So now we just adapt the syntax to our need:

```
{ "email": { "$ne": 0 }, "password": { "$ne": 0 } }
```

Here, the `$ne` (not equal) operator is used to ensure that the query doesn’t match a value of `0`. In this case, it’s trying to bypass the check for valid credentials. Since this query checks that both the `email` and `password` fields are not equal to `0`, the goal is to exploit how the Flask application interacts with MongoDB.

the MongoDB json query ensures that we bypass the login form's usual authentication check, allowing us to gain unauthorized access.

Upon sending the modified request, we successfully bypass the login form's authentication mechanism.and the flag is revealed in the response, confirming that the exploit was successful.

<figure><img src="../../../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
