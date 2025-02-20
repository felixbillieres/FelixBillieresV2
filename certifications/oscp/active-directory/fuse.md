# 📦 Fuse

<figure><img src="../../../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

First i scan for open ports:

<figure><img src="../../../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

I try enumerating various services like smb, ldap and rpc but no good so i hop on the webserver after adding fabricorp.local to hosts file

<figure><img src="../../../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

When clicking on view we see some possible usernames so i add all of them in a file and it will be helpful later:

<figure><img src="../../../.gitbook/assets/image (190).png" alt=""><figcaption></figcaption></figure>

We can quickly check if the usernames are valid with kerbrute:

```
./kerbrute userenum -d fabricorp.local --dc 10.129.2.5 ../users
```

<figure><img src="../../../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

Now we need to craft a password for those usernames:

```
cewl -d 7 -m 8 --with-numbers -w cewl.out http://fuse.fabricorp.local/papercut/logs/html/index.htm
```

and now we combine our usernames and passwords and spray:

```
nxc smb 10.129.2.5 -u ../users -p cewl.out
```

<figure><img src="../../../.gitbook/assets/image (192).png" alt=""><figcaption><p>tlavel:Fabricorp01 // bhult:Fabricorp01</p></figcaption></figure>

Now that we know their old password we can change them with a tool called smbpasswd:

Will come back later i have problems changing the SMB passwd:

```
machine 10.129.2.5 rejected the password change: Error was : The transport connection is now disconnected..
&
Failed to find entry for user bhult@10.129.2.5.
```
