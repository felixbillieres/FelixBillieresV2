# Console

So when we launch the challenge and go on the webpage we are greeted with a php info page with something that stands out:

<figure><img src="../../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

after looking up what was php-console we can see this repository:

{% embed url="https://github.com/barbushin/php-console" %}

So from what i see i need to launch the tool in a chromium browser and it's not in the extension store so i clone the repo of the extension, go in the extension manager and load the unpacked extension and select the php console extension folder, update and we'll be good:

<figure><img src="../../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (25).png" alt=""><figcaption><p>Now we can use it </p></figcaption></figure>

So let's go back on our phpinfo page and refresh it, the extension should prompt us for some sort of password:

<figure><img src="../../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

I enter test as... a test and nothing seems to change and then i decide to go and see the cookies of our page and see a php-console cookie that was not here before:

<figure><img src="../../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

So i go and decode the base64 cookie to reveal the interesting content:

<figure><img src="../../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

So i mess around for some time trying to see how the token is crafted then i try and re input a random password (helloworld) and see that the cookie changes slightly:

<figure><img src="../../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

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
