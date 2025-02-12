# BitForge

so i start off by scanning ports:

<figure><img src="../../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

I add bitforge.lob to my /etc/hosts file and go visit the website:

<figure><img src="../../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

I click on login, see nothing and click on employee portal:

<figure><img src="../../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

Then i add the new vhost to the hosts file and see this portal:

<figure><img src="../../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

then i find this exploit that seems perfect:

<figure><img src="../../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

So i need to find a user, i remebered that with the nmap scan it found a /.git/ directory:

<figure><img src="../../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

maybe a username we can exploit:

<figure><img src="../../../.gitbook/assets/image (96).png" alt=""><figcaption><p>mcsam@bitforge.lab</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (97).png" alt=""><figcaption><p>1ce700a508aec3d5e4d4aa1b128a662f2c85f5ad</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

ok so this seems like the way to go so i do what git dumper does but by hand ->

```
wget -r -np -nH --cut-dirs=1 -R "index.html*" http://bitforge.lab/.git/
```

<figure><img src="../../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

Now we can connect to the mysql db and enumerate everything using the dbName
