# 📦 Search

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So after scanning for open ports we see some interesting stuff, let's dig in the web services:

There is an about us page with some possible users of the domain:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Maybe that's cheating but this box had a "cleartext creds" tag, so i was extra cautious and found this picture in a carousel:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

While zooming we can see something that's maybe a password:

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And we can easily guess the policy for usernames and get a valid set if credentials:

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We are able to kerberoast the user web\_svc:

```
nxc ldap 10.129.229.57 -u users.txt -p 'IsolationIsKey?' --kerberoasting kerb.out
```

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

cracking this hash gets us `@3ONEmillionbaby`

I spray this password and see that it's the password of Edgar.Jacobs:

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This set of creds unlock a lot of shares so i download them all:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
nxc smb 10.129.229.57 -u Edgar.Jacobs -p '@3ONEmillionbaby' -M spider_plus -o DOWNLOAD_FLAG=True
```

After going through a lot of files we find something interesting:

<figure><img src="../../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

Doing a strings on the document reveals plenty of other files:

<figure><img src="../../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

There i was stuck so i asked for help and got a hint to look at `sheetProtection` tag exploitation and found this tag in sheets2.xml, so after looking up some documentation i saw that i needed to remove this tag and it'll remove the protection of the file ->

Will continue later, got work to do.
