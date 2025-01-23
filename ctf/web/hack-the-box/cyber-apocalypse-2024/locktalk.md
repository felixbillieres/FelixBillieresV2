# LockTalk

<figure><img src="../../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

So we start off with some sort of API with different endpoints. We also have the source code of the application&#x20;

If i try to execute the JWT token ticket request i get a message:

_Forbidden: Request forbidden by administrative rules._

so i go and check the source code to see how this is triggered and found how the JWT token was crafted:

```python
token = jwt.generate_jwt(claims, current_app.config.get('JWT_SECRET_KEY'), 'PS256', datetime.timedelta(minutes=60))
```

I tried to craft one myself but didn't manage to do some valid stuff so i started to look around again. Oh i forgot to mention that i could not request a token and when i try to curl it i get _**403 Forbidden**_ and i found the reason behind that:

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

This line instructs **HAProxy** to **deny the HTTP request** if the condition specified within the curly braces `{}` evaluates to **true**.

**`-i`**:

* Makes the matching **case-insensitive**. For example:
  * `/api/v1/get_ticket`, `/API/v1/get_ticket`, or `/Api/V1/Get_Ticket` would all match.

**Denied Requests:**

* `GET /api/v1/get_ticket`
* `GET /api/v1/get_ticket?id=123`
* `GET /API/V1/get_ticket`

**Allowed Requests:**

* `GET /api/v1/get_user_ticket`
* `GET /other_path`

So i tried some 403 bypass:

{% embed url="https://github.com/intrudir/BypassFuzzer" %}

And requesting [http://83.136.250.116:32821/%2fapi/v1/get\_ticket](http://83.136.250.116:32821/%2fapi/v1/get_ticket) gives us a valid ticket

<figure><img src="../../../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

We still can't ask for flag:

<figure><img src="../../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

So from what i see i need to have role = administrator to ask for flag so i took the jwt and tried to change my role manually:

<figure><img src="../../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (41).png" alt=""><figcaption><p>Nope</p></figcaption></figure>

And i looked at the json that we could request but nothing interesting poped up. So i went to the requirements&#x20;

<figure><img src="../../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

and looking at the version i found some articles that seemed interesting:

{% embed url="https://www.vicarius.io/vsociety/posts/authentication-bypass-in-python-jwt-cve-2022-39227" %}

<figure><img src="../../../../.gitbook/assets/image (43).png" alt=""><figcaption><p>nice</p></figcaption></figure>

I tried using the poc of the website but ran into to many errors that i didn't want to fix :monkey\_face: so i ofund another poc:

{% embed url="https://github.com/user0x1337/CVE-2022-39227" %}

```
python poC.py -j eyJhbGciOiJQUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3Mzc2MzI4MjEsImlhdCI6MTczNzYyOTIyMSwianRpIjoibE4yMkQwUWhaRmdzQi1sY1lBUU1aUSIsIm5iZiI6MTczNzYyOTIyMSwicm9sZSI6Imd1ZXN0IiwidXNlciI6Imd1ZXN0X3VzZXIifQ.J-6cMrt8a9jff7KDBZwfipCb74ZDCSX_uc8J4QSxMjAFHK9hp5s4C9U7ZB0NH0DrJ9DS8UG-IAO29DutAhBlKlzVDd4mDROIOtdwCY4sYLUBRwc9NaiwXbdwI52SCQQvYfyOk0LzMPu9mRAywW4xbuNQZZGWSTeV4CbAG_vw_CzQwPIHQRmRYgHecXlm_p1gnOW2aXUFahzghB1PnrSrz8inQ7Nk3QzvPa8-g8bFLrC1-It-r5a4ji2MtT-FlRPXdOWESdlvHLgSsFBRnhv8EU3WGx9bJFjAocTQN_9usaks4HnLaHxUoaCkYz2vBHJYFGtvvrVZn3Bq0680McszuA -i role=administrator
```

and i get a new token that i can use to ask for flag :)

<figure><img src="../../../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>
