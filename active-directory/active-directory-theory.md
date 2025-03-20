# 🧠 Active Directory: Interview Prep Handbook

{% hint style="info" %}
All this documentation was made possible thanks to this Website that goes way deeper in everything listed below, I tried to make it shorter and question-based but for the technical guys here you can go and check out : [https://zer1t0.gitlab.io/posts/attacking\_ad/](https://zer1t0.gitlab.io/posts/attacking_ad/)
{% endhint %}

{% hint style="warning" %}
Disclaimer: English is not my native language, so there may be grammar mistakes in this document. Additionally, as I'm not a cybersecurity professional, there might be technical inaccuracies present. If you notice any mistakes or have suggestions for improvement, please don't hesitate to reach out. I'm committed to promptly fixing any errors to enhance the quality of the document.
{% endhint %}

#### Game Plan, 4 week AD rush

**Week 1: Understanding the Basics**

* Milestone 1 (End of Week 1):
  * Have a general understanding of the following sections:
    * Domains
    * Trusts
    * Users
    * Groups
    * Computers
    * Authentication
    * Authorization
  * Be able to explain the basic concepts of each section.

**Week 2: Deepening Knowledge**

* Milestone 2 (End of Week 2):
  * Have a detailed understanding of the following sections:
    * Trust types
    * Trust key
    * User properties
    * Important groups
    * Windows computers connection
    * NTLM
    * ACLs
  * Be able to describe the processes and mechanisms involved in each section.

**Week 3: Mastering Advanced Concepts**

* Milestone 3 (End of Week 3):
  * Have mastery over the following sections:
    * Trust transitivity
    * How to query the database?
    * Kerberos
    * Kerberos Delegation
    * Powershell remoting
    * SSH tunneling
  * Be able to explain advanced concepts and apply them to specific scenarios.

**Week 4: Review and Consolidation**

* Milestone 4 (End of Week 4):
  * Review all articles and sections to consolidate knowledge.
  * Practice exercises or simulations to test your understanding.
  * Be capable of explaining each article in detail and answering specific questions on each topic.

***

**What is an Active Directory?**

* It's a system that allows to manage a set of Computers & Users connected in the same network from a central server

### Domain

**What is a Domain?**

* A domain is a set of computers connected to the same AD database that is managed by the same central server called **Domain Controller**

**What is a Domain Name?**

* every domain has a DNS name, it can be the same as their website (contoso.com) or and internal domain (contoso.local)

```
#current user domain 
$env:USERDNSDOMAIN
#current computer domain
(Get-WmiObject Win32_ComputerSystem).Domain
```

**What is a NetBIOS name?**

* It facilitates communication between computers on a local area network (LAN). Every domain can have it's NetBIOS name. In the logs, you can identify users with `CONTOSO\Administrator`, where the first part is the NetBIOS name and the second one is the username.

**What is a Forest?**

* A forest is a tree of domains. The name of the forest is the name of the root domain (ex: contoso.com) and it has one or more subdomains:

```
              contoso.local
                    |
            .-------'--------.
            |                |
            |                |
     it.contoso.local hr.contoso.local
            | 
            |
            |
  webs.it.contoso.local
```

* each domain has its own database and its own Domain Controllers. Users from a domain can access ressources in another domain but not in another forest

**What are Functional Modes?**

* A functional mode is the version of a domain/forest. The modes are named based on the minimum Windows Server operative system required to work with them. Then if, for example, you find a domain/forest with Windows2012 mode, you can know that all the Domain Controllers are at least Windows Server 2012.

```
 #Get the mode of the forest/domain
(Get-ADForest).ForestMode
#OR
(Get-ADDomain).DomainMode
```

### **Trust**

**What are Trusts?**

* Trust is what links users and allow them to access to other domains in the same forest. [A trust is a connection from a domain to another](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc731335\(v=ws.10\)). Not a physical network connection, but a kind of authentication/authorization connection.

**What is trust direction?**

* A trust direction is a one sided trust that allows the trusted users to access the trusted domains resources

```
 (trusting)         trusts        (trusted)
  Domain A  -------------------->  Domain B
       outgoing               incoming
       outbound               inbound
                    access
            <--------------------
```

**Difference between inbound and outbound connection:**

* If the trust is directed towards your current domain, it's an incoming (also called inbound) connection. If it goes from your domain to the other, it's an outcoming (also called outbound) connection.

```
#Trust of your current domain
nltest /domain_trusts
```

**What is Trust transitivity?**

* A [trust can be transitive or nontransitive](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc754612\(v=ws.10\)), a non-transitive trust indicates that the trust is not extended beyond the immediate relationship. In other words, just because entity A trusts entity B and entity B trusts entity C, it does not mean that entity A automatically trusts entity C whereas transitive trust means that trust is passed from one entity to another through a chain of trust relationships. Entity A trusts entity B, and entity B trusts entity C, then entity A also implicitly trusts entity C

transitive trust:

```
       (trusting)   trusts   (trusted)  (trusting)   trusts   (trusted)
  Domain A  ------------------->  Domain B --------------------> Domain C
                    access                          access
            <-------------------           <--------------------
```

* In a domain, to be able to access all the resources, you need all the parents and children to be connected by a bidirectional transitive trust:

```
              contoso.local
               ^  v   v  ^  
          .----'  |   |  '----.
          |  .----'   '----.  |
          ^  v             v  ^
     it.contoso.local hr.contoso.local
          ^  v 
          |  |
          ^  v
  webs.it.contoso.local
```

**What are the different Trust types?**

* **Parent-Child**: The default trusts created between a parent domain and its child.
* **Forest**: A trust to share resources between forests. This way any domain of the forest can access to any domain on the other forest (if the direction and transitivity of the trust allow it). If a forest trust is misconfigured, then it can allow to [take control of the other forest](http://www.harmj0y.net/blog/redteaming/not-a-security-boundary-breaking-forest-trusts/).
* **External**: A trust to connect to a specific domain that is in a non trusted forest.
* **Realm**: A special trust to connect Active Directory and a non-Windows domain.
* **Shortcut**: When two domains within the forest communicate often but are not directly connected, you can avoid jumping over many trusts by creating a direct shortcut trust.

**Explain Trust Keys:**

When you communicate through trust, there is a communication between the domain controller of your domain and the domain controller of the target domain and the domain controllers needs to share a key to keep the communications secure.

### Users

**Explain User Identifiers;**

* User identifiers are what we use to distinguish users. it can be the username (stored in the **SamAccountName** attribute) or the **SID** (a combination of the domain SID plus the user RID (Relative Identifier), which is the last number that appears in the user SID)

**What is User Secrets**?

*   For the Domain Controller to be able to authenticate the user, the database stores the sensitive data but not in plaintext. The following secrets are saved:

    * NT hash (and LM hash for the older accounts)
      * The hashes are stored both in Windows local [SAM](https://en.wikipedia.org/wiki/Security_Account_Manager) and Active Directory NTDS databases
    * Kerberos keys

    you need administrator privileges (or equivalent) to [dump the domain database](https://zer1t0.gitlab.io/posts/attacking_ad/#domain-database-dumping) with a dcsync attack or grabbing the `C:\Windows\NTDS\ntds.dit` file from the Domain Controller.

**What can you do with those hashes?**

*   LM hashes are not used a lot because often obsolete. NT hashes can be used to perform [Pass-The-Hash](https://zer1t0.gitlab.io/posts/attacking_ad/#pass-the-hash) or [Overpass-the-Hash](https://zer1t0.gitlab.io/posts/attacking_ad/#pass-the-key) attacks in order to impersonate users in remote machines.

    Additionally, you can try to crack the LM and NT hashes with [hashcat](https://hashcat.net/) to recover the original password. If you are lucky and the LM hash is present, this should be quickly.

**What can you do with kerberos keys?**

* The keys can be used to ask for a Kerberos ticket that represents the user in Kerberos authentication. It can be used in a [Pass-The-Key](https://zer1t0.gitlab.io/posts/attacking_ad/#pass-the-key) attack to retrieve a ticket for the impersonated user.

**What is UserAccountControl?**

* It's a proprety in the user class tha contains a **series of flags** that are very relevant for the security and the domain and used in many attacks (flags: [https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties))

**What are some Important Users and how to list them?**

```
net user /domain
#or
Get-ADUser -Filter * | select SamAccountName
```

* `Administrator` user is the most privileged account of the domain, `krbtgt` account is very important too. Its secrets (NT hash and Kerberos keys) are used to encrypt the tickets (specifically the TGTs) used by Kerberos that allows to authenticate users. If you are able to compromise the `krbtgt` account, you will be able of creating [Golden Tickets](https://en.hackndo.com/kerberos-silver-golden-tickets/).

**What are Computer accounts and difference with user accounts?**

* The difference between user accounts and computers accounts is that the firsts are stored as instances of [User class](https://docs.microsoft.com/en-us/windows/win32/adschema/c-user) in the database whereas the others are stored as instances of [Computer class](https://docs.microsoft.com/en-us/windows/win32/adschema/c-computer) (which is a subclass of User class). Moreover the computer accounts names are the computer hostname finished with a dollar sign `$`.

```powershell
PS C:\> Get-ADObject -LDAPFilter "objectClass=User" -Properties SamAccountName | select SamAccountName

SamAccountName
--------------
Administrator
Guest
DC01$
krbtgt
anakin
WS01-10$
WS02-7$
DC02$
han
POKEMON$
```

**What are Trust accounts?**

* It's good to know that you can have a normal user that finishes with a $, but when a trust is established, an associated user object is created in each domain to store the trust key. The name of the user is the NetBIOS name of the other domain, finished in $
* For example, in case of the trust between the domains FOO and BAR, the FOO domain would store the trust key in the BAR$ user, and the BAR domain would store it in the FOO$ user.

### Groups

**What are groups?**

* Rather than managing each of the users file permissions, you put them in groups to handle the policy of all the users in a same group. the groups are stored in the domain database. And, in the same way, they can be identified by the `SamAccountName or the SID`

```
Get-ADGroup -Filter * | select SamAccountName
```

**Explain Administrative groups:**

* There are many [default groups](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#default-security-groups) defined for different roles in the domain/forest. As attacker, one of the most juicy groups is the [Domain Admins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-domainadmins) group, that gives administrator privileges to its members in the domain

```
Get-ADGroup "Domain Admins" -Properties members,memberof
```

**What are other interesting groups:**

* The [Enterprise Admins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-entadmins) group, which provides administrator privileges in all the forest. It's a group that only exists in the root domain of the forest, but is added by default to the [Administrators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#administrators) group of the all the domains in the forest. There is also the the `Domain Admins` group that is added to the `Administrators` group of the domain, as well as the `Administrators` groups of the domain computers.

```
                        .------------------------.
                        |     contoso.local      |
       .-------------------------------------------------------------.
       |                                                             |
       |                   .----------------.                        |  
       |               .-->| Administrators |<-.   .->Administrators |
       |               |   '----------------'  |   |     ____        | 
       |               |    .---------------.  |   |    |    |       |
       |               |    | Domain Admins |>-'---'    |____|       |
       |               |    '---------------'           /::::/       |
       |               |   .-------------------.                     |
       |               '--<| Enterprise Admins |                     |
       |                   '-------------------'                     |
       |                             v v                             |
       '-----------------------------|-|-----------------------------'  
                           |         | |      |                         
                           |         | |      |                         
                 .---------'         | |      '-----------.             
                 |                   v v                  |             
.----------------------------------. | | .----------------------------------.
|        it.contoso.local          | | | |        hr.contoso.local          |
|----------------------------------| | | |----------------------------------|
|                                  | v v |                                  |
|        .----------------.        | | | |        .----------------.        |
|     .->| Administrators |<---------' '--------->| Administrators |<-.     |
|     |  '----------------'        |     |        '----------------'  |     |
|     |  .---------------.         |     |        .---------------.   |     |
|     '-<| Domain Admins |         |     |        | Domain Admins |>--'     |
|        '---------------'         |     |        '---------------'         |
|                |                 |     |                |                 |
|        .-------'---------.       |     |        .-------'---------.       |
|        |                 |       |     |        |                 |       |
|        v                 v       |     |        v                 v       |
| Administrators    Administrators |     | Administrators    Administrators |
|       ____              ____     |     |      ____              ____      |
|      |    |            |    |    |     |     |    |            |    |     |
|      |____|            |____|    |     |     |____|            |____|     |
|      /::::/            /::::/    |     |     /::::/            /::::/     |
'----------------------------------'     '----------------------------------'
```

* other [important groups](https://adsecurity.org/?p=3700) to be taken into account

**What are different Group Scopes?**

* There are 3 different groups based on their scope:
  * **Universal** groups can include members from any domain within a forest or trusted forests and grant permissions within the same forest or trusted forests. The Enterprise Admins group is an example of a Universal group.
  * **Global** groups can include members only from the same domain and grant permissions within domains of the same forest or trusting domains or forests. The Domain Admins group is an example of a Global group.
  * **DomainLocal** groups can include members from the domain or any trusted domain and grant permissions only within their domains. The Administrators group is an example of a DomainLocal group.

### Computers <a href="#computers" id="computers"></a>

**Different types of Computers within a domain:**

* **Domain Controllers**: The central servers that manage the domain. They are Windows Server machines.
* **Workstations**: The personal computers used by people every day. These machines are usually Windows 10 or 7 machines.
* **Servers**: The computers that offers services such as webs, files or databases. They are usually Linux or Windows Server machines.

**What is a domain controller?**

* The domain controller is the **central server of a domain**, that is running the [Active Directory Domain Service](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) (AD DS). It keeps the domain DB stored in ths ntds.dit file.

**What is Domain database dumping?**

* When we become domain admin, in order to read sensitive files like krbtgt user creds or create Golden tickets, we can dump the DB of the DC
* One way of doing this is [dumping the NTDS.dit file](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/ntds.dit-enumeration#no-credentials-ntdsutil) locally with [ntdsutil](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc753343\(v=ws.11\)) or [vssadmin](https://docs.microsoft.com/en-gb/windows-server/administration/windows-commands/vssadmin) or remote with the [impacket secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) script.

```
secretsdump.py 'contoso.local/Administrator@192.168.100.2' -just-dc-user krbtgt
```

**How to discover Windows computers in a domain or network?**

* ([NetBIOS name service](https://zer1t0.gitlab.io/posts/attacking_ad/#netbios-name-service) listens in the port 137) We can scan the network and perform a NetBIOS scan by using a tool like [nbtscan](http://www.unixwiz.net/tools/nbtscan.html) or nmap [nbtstat](https://nmap.org/nsedoc/scripts/nbstat.html) script.
* ([SMB](https://zer1t0.gitlab.io/posts/attacking_ad/#smb) listens in the port 445) We can take advantage of the NTLM authentication negotiation to retrieve the machine name. You can perform an scan with [ntlm-info](https://github.com/Zer1t0/ntlm-info) or nmap [smb-os-discovery](https://nmap.org/nsedoc/scripts/smb-os-discovery.html) script.

**How to connect with RPC/SMB**

* If you have port 445 open, you can use tools such as [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) and the impacket examples [psexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py), [wmiexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py) ([wmiexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py) will also need the port 135)
* You can also perform Pass-the-hash with the NT or LM hash directly with impacket tools:

```
psexec.py contoso.local/Anakin@192.168.100.10 -hashes :cdeae556dc28c24b5b7b14e9df5b6e21
```

* With the PsExec tool, [inject the NT hash in the Windows session with mimikatz](https://stealthbits.com/blog/passing-the-hash-with-mimikatz/).

With those techniques we connect via NTLM auth, but by default Kerberos is used and we would need to provide a kerberos tickets with those tools

* in Windows we will need to [inject the ticket in the session](https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a#using-ticket-in-windows) by using [mimikatz](https://github.com/gentilkiwi/mimikatz) or [Rubeus](https://github.com/GhostPack/Rubeus).

Windows and linux use different ticket formay that we can convert using [ticket\_converter](https://github.com/Zer1t0/ticket_converter) or [cerbero](https://github.com/Zer1t0/cerbero#convert).

**How to use Powershell Remoting to connect**

to connect to a Windows machine and get a Powershell session in the remote machine

* From windows we need to use [CmdLets and parameters](https://docs.microsoft.com/en-us/powershell/scripting/learn/ps101/08-powershell-remoting?view=powershell-7.1)
  * if we have a Kerberos ticket or a NT hash, we will need to inject them by using [Rubeus](https://github.com/GhostPack/Rubeus) or [mimikatz](https://github.com/gentilkiwi/mimikatz).

```
.\Rubeus.exe asktgt /user:Administrator /rc4:b73fdfe10e87b4ca5c0d957f81de6863 /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v1.6.1

[*] Action: Ask TGT

[*] Using rc4_hmac hash: b73fdfe10e87b4ca5c0d957f81de6863
[*] Building AS-REQ (w/ preauth) for: 'contoso.local\Administrator'
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFQjCCBT6gAwIBBaEDAgEWooIETzCCBEthggRHMIIEQ6ADAgEFoQ8bDUNPTlRPU08uTE9DQUyiIjAg
      oAMCAQKhGTAXGwZrcmJ0Z3QbDWNvbnRvc28ubG9jYWyjggQFMIIEAaADAgESoQMCAQKiggPzBIID7xK3
      <!--stripped-->
      ERgPMjAyMTA1MDgwMjQzMjZapxEYDzIwMjEwNTE0MTY0MzI2WqgPGw1DT05UT1NPLkxPQ0FMqSIwIKAD
      AgECoRkwFxsGa3JidGd0Gw1jb250b3NvLmxvY2Fs
[+] Ticket successfully imported!

  ServiceName           :  krbtgt/contoso.local
  ServiceRealm          :  CONTOSO.LOCAL
  UserName              :  Administrator
  UserRealm             :  CONTOSO.LOCAL
  StartTime             :  07/05/2021 18:43:26
  EndTime               :  08/05/2021 04:43:26
  RenewTill             :  14/05/2021 18:43:26
  Flags                 :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType               :  rc4_hmac
  Base64(key)           :  95a1NmgYXwOmiyCa3qlplA==

PS C:\> Enter-PSSession -ComputerName dc01
[dc01]: PS C:\Users\Administrator\Documents> whoami
contoso\administrator
[dc01]: PS C:\Users\Administrator\Documents> hostname
dc01
[dc01]:
```

* From a Linux machine you can use [evil-winrm](https://github.com/Hackplayers/evil-winrm).

**How to connect with RDP**

* From windows we can use [mstsc](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/mstsc)
* From Linux we can use [rdesktop](http://www.rdesktop.org/), [freerdp](https://www.freerdp.com/) or [remmina](https://remmina.org/)
  * Unlike RPC/SMB, RDP transmits the password in plain text and allows the remote computer to cache the user's credentials. it enables the user to log in to the remote system without having to enter their password again at each connection thus simulating the experience of being physically logged into their own machine. This makes it impossible to use pass-the-hash attacks
  * But there is a mode called Restricted Admin, when enabled, you don't send plain text so it's then possible to perform pass the hash connections.
    * From Linux, you can [use freerdp to perform a Pass-The-Hash with RDP](https://www.kali.org/blog/passing-hash-remote-desktop/)
    * And from Windows you can [inject a NT hash or Kerberos ticket with mimikatz or Rubeus and then use `mstsc.exe /restrictedadmin`](https://shellz.club/pass-the-hash-with-rdp-in-2019/)

**Windows computers credentials**

**Explain LSASS credentials**

* On windows machine -> common place to find creds -> LSASS Process (lsass.exe) in charge of user auth
* when user logs in -> credentials cached in lsass process in order to use SSO -> credentials caches by some [SSPs](https://zer1t0.gitlab.io/posts/attacking_ad/#windows-ssps) (Security Support Providers) each one uses different auth methods, example:
  * The [Kerberos SSP](https://zer1t0.gitlab.io/posts/attacking_ad/#kerberos-ssp)
  * The [Digest SSP](https://zer1t0.gitlab.io/posts/attacking_ad/#digest-ssp)
  * The [NTLMSSP](https://zer1t0.gitlab.io/posts/attacking_ad/#ntlm-ssp)
* We can extract LSASS credentials using [mimikatz](https://github.com/gentilkiwi/mimikatz). We can launch mimikatz directly in the target machine, or [dumping the LSASS memory](https://www.deepinstinct.com/2021/01/24/lsass-memory-dumps-are-stealthier-than-ever-before/) with some tool like [procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump), [comsvcs.dll](https://lolbas-project.github.io/lolbas/Libraries/Comsvcs/) or [werfault.exe](https://www.deepinstinct.com/2021/02/16/lsass-memory-dumps-are-stealthier-than-ever-before-part-2/) and then process the generated memory dump with mimikatz or [pypikatz](https://github.com/skelsec/pypykatz). can also to use [lsassy](https://github.com/Hackndo/lsassy) to read a dump remotely avoiding to have to download the entire memory dump, that can take several megabytes.

**Explain Registry credentials**

* Other location to find credentials is the registry. One of the places where sensible credentials are stored is in the [LSA secrets](https://passcape.com/index.php?section=docsys\&cmd=details\&id=23).
* In the LSA secrets you can find:
  * **Domain Computer Account:** To be part of the domain, the computer must have a user account in the domain with available username and password. They are stored in the LSA secrets
  * **Service users passwords:** If you want to run some services as the user, the computer needs to store the password but it does not store the user
  * **Auto-logon password:** If windows [auto-logon is enabled](https://keithga.wordpress.com/2013/12/19/sysinternals-autologon-and-securely-encrypting-passwords/), the password can be stored in the LSA secrets. The other alternative is that it is saved in `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon` registry key under the key `DefaulUserName`.
  * **DPAPI master keys:** The [data protection API](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection) (DPAPI) is used to allow users encrypt sensible data without need to worry about cryptographic keys. If you are able to retrieve the master keys, then you [can decrypt users data](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/dpapi-extracting-passwords).

**How to dump registry credentials:**

To get the credentials from the SECURITY and SAM hives, we can read them from memory by using mimikatz.

execute `token::elevate` to acquire a `SYSTEM` session to read the credentials.

`privilege::debug` if required to enable the SeDebugPrivilege.

other commands:

* `lsadump::secrets`: Get the LSA secrets.
* `lsadump::cache`: Retrieve the cached domain logons.
* `lsadump::sam`: Fetch the local account credentials.

We can also save a copy of the hive files with [reg save](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/reg-save) command, move them to our machine, and finally to get the content with [impacket secretsdump](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) script or mimikatz.

**Linux computers**

**How to discover linux machines on a network?:**

Since Linux computers don't have any characteristic port opened by default, however many Linux machines are used as servers that are remotely administrate. In order to administrate Linux computers, usually the [SSH](https://zer1t0.gitlab.io/posts/attacking_ad/#SSH) protocol is used. The SSH server service listens in the port 22

Moreover, older Linux machines may have [Telnet](https://en.wikipedia.org/wiki/Telnet) enabled on port 23.

**where to find linux computer credentials**

Linux machines usually have a Kerberos client that is configured with the domain computer account. You can find the credentials in the keytab (`/etc/krb5.keytab)`or specified in the [Kerberos configuration file](https://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf_files/krb5_conf.html) in `/etc/krb5.conf.`

You can use the keys stored to [ask for a Kerberos ticket](https://www.tarlogic.com/en/blog/how-to-attack-kerberos/) an impersonate the user.

{% embed url="https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-active-directory#ccache-ticket-reuse-from-keytab" %}

**What can you say about linux kerberos tickets?**

* in order to be identified within the system, the linux machines has kerberos client configured within the computer account. You can find the creds usaully in the `/etc/krb5.keytab` OR `/etc/krb.conf` file and you can display it using `klist` command
* the next step would be to impersonate the user in the domain since, when authentificated, a kerberos ticket is retrieved under the `/tmp` directory in files with the format `krb5cc_%{uid}` . it is also possible that tickets are stored in the [Linux kernel keys](https://man7.org/linux/man-pages/man7/keyrings.7.html) instead of files, but you can grab them and convert to files by using [tickey](https://github.com/TarlogicSecurity/tickey).

**Where are stored the ssh private keys?**

* usually they are stored in the .ssh directory in a file called `id_rsa`
* In case the private key doesn't require a passphrase for using it, then you may can use it to connect to another machines in the domain with this syntax:

```
$ ssh -i id_ed25519_foo_key foo@db.contoso.local
```

{% embed url="https://robertholdsworthsecurity.medium.com/how-to-crack-an-ssh-private-key-passphrase-ab7dd1583178" %}

* a place worth looking is the bash history located in the `.bash_history` file of the user directory to find some credentials or command history

### Services <a href="#services" id="services"></a>

**What is the difference between active directory service and computer service?**

A windows or linux machine service could be understood as a background process running like a task (ex: database) but it is not obliged for a service to be listening on a port:

it could simply be a service that for example checks for updates on the system

on the other hand, active directory services are identifieres that indicates what remote services are or can be available (listening on a port) on a machine. Not all the remote services are registered in the domain database, however, the registration is required for those services that need to authenticate domain users through Kerberos.

* each active directory service stores those informations:
  * The **user** that runs the computer service.
  * The [service class](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc772815\(v=ws.10\)#service-principal-names), that indicates what kind of service is, for example web servers are registered like www class.
  * The **machine** where the service is hosted.

Since all services dont require to be executed and are stored in AD database, it's good to know that old services can lead to account takeover using kerberoast

### Database <a href="#database" id="database"></a>

So we know the basics of the database of the domain and some objects stored in it like users, groups etc..

**Can you go more in depth on classes and maybe subclasses?**

We know that there is the [User class](https://docs.microsoft.com/en-us/windows/win32/adschema/c-user), the [Computer class](https://docs.microsoft.com/en-us/windows/win32/adschema/c-computer) or the [Group class](https://docs.microsoft.com/en-us/windows/win32/adschema/c-group).

But a class can have a **subclasse** that allows to inherit propreties. For example, the Computer class is a subclass of User class, therefore the computer objects can have the same properties of the user objects ,like `SAMAccountName`, and some new custom properties,

All the classes are subclasses of the [Top](https://docs.microsoft.com/en-us/windows/win32/adschema/c-top) class & many of the most relevant classes when performing a pentest, like User and Group, are attached to [Security-Principal](https://docs.microsoft.com/en-us/windows/win32/adschema/c-securityprincipal) auxiliary class, the class that defines the `SAMAccountName` and `SID` properties.

**What do you know about properties of a class?**

Usually, properties are stored as a string value like _name_ or something else. There are some properties that contain sensitive data that are marked as [confidential properties](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/mark-attribute-as-confidential)

for example, the [UserPassword](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/f3adda9f-89e1-4340-a3f2-1f0a6249f1f8) and [UnicodePwd](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-ada3/71e64720-be27-463f-9cc5-117f4bc849e1) propreties cannot be read, only written. When a [password change](https://docs.microsoft.com/en-US/troubleshoot/windows/win32/change-windows-active-directory-user-password) is required, these properties can be written in order to modify the user password.

Only authorized users can retrieve the confidential properties.

There are some properties that require to meet certain conditions before being written.

**Define principals in active directory?**

In Active Directory, a **principal is a security entity**. The most common **principals are users, groups and computers**.

{% embed url="https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-principals" %}

**What are 3 kinds of SIDs in active directory?**

**Domain SID:** used to identify the domain, as well as the base for SIDs of the domain principals.

**Principal SID:** used to identify principals. It is compose by the domain SID and a principal RID

[**Well-known SIDs**](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/security-identifiers-in-windows) that identify abstract entities for special situations. Some of them are:

* **Authenticated Users**
* **Principal Self**
* **Administrator**
* **Domain Admins**

**What are Distinguished Names?**

The distinguished name is like a path that indicates the object position in the database hierarchy

It's used to identify objects in the db and reference objects (ex. the members of a group are referenced by its `DistinguishedName`.

It's composed by:

* Domain Component (DC) -> for `it.domain.com` the DC part will be `DC=it,DC=domain,DC=com`.
* [Organizational Unit](https://en.wikipedia.org/wiki/Active_Directory#Organizational_units) (OU)
* Common Name (CN) -> in `CN=Administrator,CN=Users,DC=contoso,DC=local`, the `CN=Users` identifies the Users container.

**What does the term Partitions refer to in AD databases?**

The databases are divided by partitions:

* **Domain**: Stores the domain objects.
* **Configuration**: Stores configuration of the domain, such as the `HOST` service alias or `Well-known` SIDs that we have seen before.
* **Schema**: Stores the definition of the classes and properties used by the database.
* **Domain DNS Zones**: Stores the DNS records of the domain and subdomains.
* **Forest DNS Zones**: Stores the DNS records of the rest of the forest, including parent domains.

**How would you define Global Catalog?**

To speed up searches through objects in other domains, some domain controllers have a read-only partitions with a subset of objects of other domains. These Domains Controllers can be called [Global Catalogs](https://docs.microsoft.com/pt-pt/previous-versions/windows/server/cc737410\(v=ws.10\)#domain-controller-and-global-catalog-server-structure)

**Explain querying with LDAP & ADWS**

To be able to interract with the domain controller database and it's data, we wan use different protocoles/services ->

**LDAP** allows to filter objects, interact and edit them in the db via a query syntax and you can go more in depth and retrieve propreties from the objects you interract with like the name (can check out more on [LDAP wiki](https://ldapwiki.com/)). We can pretty much interract with anything that is not highly sensitive. When enumerating active directory, ldap is a go to, you can use [ldapsearch](https://linux.die.net/man/1/ldapsearch) and [ldapmodify](https://linux.die.net/man/1/ldapmodify) tools (on linux). since we can modify objects we could imagine adding a user.

On the other hand there is **ADWS** (Active Directory Web Services), it also manipulates domain objects but it's based on [SOAP](https://en.wikipedia.org/wiki/SOAP) messages. It works very well with LDAP and is compatible with LDAP filters. So much that when used, the DC internally uses LDAP reqs to retrieve results

**Other protocols worth to know**

* The [DNS](https://en.wikipedia.org/wiki/Domain_Name_System) protocol
* The [SAMR](https://docs.microsoft.com/en-us/openspecs/windows_protocols/MS-SAMR/4df07fab-1bbc-452f-8e92-7853a3c7e380) (Security Account Manager Remote) protocol
* The [DRSR](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47) (Directory Replication Service Remote) protocol
* The [Kerberos](https://www.tarlogic.com/en/blog/how-kerberos-works/) authentication protocol
* The [Netlogon](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-nrpc/ff8f970f-3e37-40f7-bd4b-af7336e4792f) protocol

## Security

the 3 main topics in the security realm of Active Directory are:

**Address resolution; Authentication and Authentication**

### Address resolution <a href="#address-resolution" id="address-resolution"></a>

**What are some dangers related to misconfigured address resolution?**

The main danger is if a user or program can be tricked into connecting to an erroneous machine. Some common attacks are :

* Person-in-The-Middle (PitM) where the attacker interecpts the communication of the user and maybe interract with it if not encrypted
* [NTLM Relay](https://en.hackndo.com/ntlm-relay/) attack, using NTLM authentification from a victim and redirect it to a malicious server
* [NTLM crack](https://0xdf.gitlab.io/2019/01/13/getting-net-ntlm-hases-from-windows.html) attack where the attacker just tries to crack the NTLM hash

**What are the 3 types of addresses that need to be resolved?**

**MAC address ->** unique identifier of each computer in the world. Sends messages via the ethernet protocol (can be altered with spoofing)

**IP address ->** used by the IP protocol on the internet layer. Allows communication via different computers in a same network (can read more about IP address attribution with [DHCP](https://zer1t0.gitlab.io/posts/attacking_ad/#dhcp) or static IPs)

**Hostnames ->** Just like websites and DNS, Hostnames help us memorize and interract with computers with human-friendly names like `computerOfFelix` rather than just an IP that is hard to remember

**What are some vital processes for computers when it comes to finding the correct address to comunicate?**

**Hostname-IP resolution**

* to map a hostname to it's ip, computers can ask ask central server for hostname resolution which is used by DNS or they can send broadcast request asking all the computers to identify themselves (used by netBIOS, LLMNR, mDNS)

**IP-MAC resolution**

* Once we find an IP, we can broadcast to all to tell the network card of the computer to identify itself by asking for the MAC address related to the IP using the ARP protocol

**IP configuration**

* Either it's done manually or dynamically using DHCP, we need to configure the IP of a computer. The danger if done dynamically is that the computer is not configured and asks blindly to the DHCP server using broadcast and can be tricked if an attacker has a fake DNS server

**What is ARP?**

[ARP](https://en.wikipedia.org/wiki/Address_Resolution_Protocol) (Address Resolution Protocol) allows to map the relation between the IP address of a computer and its MAC (Media Access Control) address.

the client machine sends t broadcast ARP request to the local network, asking for the one that has the target IP address. Then the computer with that IP should respond identifying its MAC. Finally the client sends the application packets to that Ethernet address.

```
                                                   .---.
                                                  /   /|
                                                 .---. |
                                       .-------> |   | '
                                       |         |   |/ 
                                       |         '---'  
   .---.                               |
  /   /|                               |           .---.
 .---. |    1) Who is 192.168.1.5?     |          /   /|
 |   | ' >-------->>-------------------.-------> .---. |
 |   |/                                |         |   | '
 '---'   <---------.                   |         |   |/ 
                   |                   |         '---'  
                   |                   |
                   |                   |           .---.
                   |                   |          /   /|
                   |                   '-------> .---. |
                   |                             |   | '
                   '-<<------------------------< |   |/ 
                     2)  I am 192.168.1.5        '---'  
                      (MAC 01:02:03:04:05:06)
```

**Explain ARP Spoofing**

An attacker could try to respond to all ARP requests to impersonate other computers. To add a layer of security, rather than sending ARP requests every time they communicate, computers keep the old responses in their ARP cache to reduce the number of requests needed.

**Explain how DHCP works**

DHCP helps configure dynamic IP addresses to the computers of a network. When new to a network, the foreign computer will look for the DHCP server to get a config that allows them to interract with the network, it works in 4 steps:

-> Broadcast from the client to the server to find DHCP server

-> Server responds to request with valid IP

-> client receives IP and sends message to server to request it

-> Server confirms client can use the IP and config params like IP renewal time

```
client        server
    |             |
    |  discovery  |
    | ----------> |
    |             |
    |    offer    |
    | <---------- |
    |             |
    |   request   |
    | ----------> |
    |             |
    | acknowledge |
    | <---------- |
    |             |
```

**What is a Rogue DHCP server**

A rogue DHCP server is a device on a network that, without permission, gives out IP addresses and network settings to devices, which can cause network problems and security risks. Therefore the perfect tool for an attacker could create a rogue DHCP server in order to set a custom configuration in the clients and redirect them to fake computers or domains controlled by the attacker

**Explain DHCP Starvation attack:**

It's a DOS attack where a fake client requests all the available IPs the DHCP server has to offer and make it impossible for legitimate users to obtain an IP:

```
$ dhcpstarv -i enp7s0
08:03:09 11/30/20: got address 192.168.100.7 for 00:16:36:99:be:21 from 192.168.100.2
08:03:09 11/30/20: got address 192.168.100.8 for 00:16:36:25:1f:1d from 192.168.100.2
08:03:09 11/30/20: got address 192.168.100.9 for 00:16:36:c7:79:f2 from 192.168.100.2
```

### **DNS**

**What is DNS?**

DNS is a protocol that resolves DNS names of a computer to it's address (port 53). (it can also be used with mapping an IP to its name or resolving the aliases for a name)

```
    client                     DNS server
    .---.   A elFelixio.oscp?     .---.
   /   /| ------------------>  /   /|
  .---. |                     .---. |
  |   | '   185.199.111.153   |   | '
  |   |/  <------------------ |   |/ 
  '---'                       '---'
```

**What are DNS zones?**

DNS is hierarchical, each zone keeps record for it's domain and the subdomains attached to it. Let's take 2 zones for example:

_Zone contoso.com_

```
contoso.com
mail.contoso.com
www.contoso.com
```

_Zone internal.contoso.com_

```
internal.contoso.com
it.internal.contoso.com
admin.internal.contoso.com
hr.internal.contoso.com
```

Those are 2 independent zones, it makes it easier to keep track. The DNS server will comunicate with those zones in order to privide information to other zones

{% hint style="info" %}
With the `www.contoso.com` IP address, the DNS server needs to communicate with contoso authoritative DNS server, that manages the contoso.com zone, in order to retrieve this information.
{% endhint %}

**Explain DNS exfiltration:**

Let's say we have a server that is isolated and has no internet access, but it can perform DNS queries. If the local DNS performs requests to other DNS servers on the internet, it can be abused: let's say i have the DNS server `malicious.com` , every query made to be or other subdomains will reach my server. An attacker could query a subdomain like `felix.malicious.com` and technically, the query should go through the parent domain, thus, retrivieng informations. (attacker can use a tool like [iodine](https://github.com/yarrick/iodine) or [dnscat2](https://github.com/iagox86/dnscat2))

**How can DNS zone transfer be dangerous?**

We know that zone transfer replicates all DNS server records. If misconfigured, anyone could perform zone transfers

If just one DC allows to perform the zone transfer whereas the rest of DCs refuse the zone transfer, the misconfigured DCcould lead to anyone being able to perform zone transfers, thus recolecting all the DNS information without require any credentials.

**What to do if zone transfers aren't allowed?**

Since DNS records are stored in the AD database, we can read it using LDAP as any domain user to then dump all the DNS records, we can use [adidnsdump](https://github.com/dirkjanm/adidnsdump) to do so and it will save the output un _records.csv_ file

**What is ADIDNS?**

{% embed url="https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/adidns-spoofing" %}

**ADIDNS** is the Active Directory's implementation of DNS. In this setup, the role of the DNS server is assumed by the Domain Controller (DC). This is because the Active Directory database contains the DNS names of the computers within the domain as well as other DNS records. This integration allows for seamless name resolution and management of DNS entries within the Active Directory environment.

**What are DNS dynamic updates?**

Dynamic updates allows clients to create/modify/delete DNS records. Since any user can create a DNS record, the user that created the record becomes the owner of this record, thus, only authorized users of a certain record can update of modify/delete the record

**How Could be abuse Dynamic updates?**

{% embed url="https://www.netspi.com/blog/technical-blog/network-penetration-testing/exploiting-adidns/" %}

{% embed url="https://www.netspi.com/blog/technical-blog/network-penetration-testing/adidns-revisited/" %}

Since DNS records are stored in the Active Directory database, they can be created/modify/deleted by using LDAP.

Registering a wildcard record (\*) could be interesting since it's used to specify a default IP address that is used to resolve those queries that doesn't match any other record. It could lead to [perform PitM attacks](https://blog.netspi.com/exploiting-adidns/) if it is used to point to a computer controlled by us.

**What is NETBIOS?**

[NetBIOS](https://en.wikipedia.org/wiki/NetBIOS) (Network Basic Input/Output System) helps applications in the same LAN (Local Area Network) communicate between them. Howerever it was not able to communicate them on different networks so this led to the creation of NBT protocol (NetBIOS over TCP/IP) to make NetBIOS work over TCP and UDP protocols and allow applications that used NetBIOS to communicate over internet.

**What are the 3 services used by NETBIOS?**

* **NetBIOS Datagram Service ->** used as transport layer for application protocols that requires a connectionless communication.
* **NetBIOS Session Service ->** used as transport for connected-oriented communications
* **NetBIOS Name Service ->** This one is very interesting for pentesting, it allows to:
  * Resolve NetBIOS name to an IP address
  * Known the status of a NetBIOS node
  * Register/Release a NetBIOS name

**What is the NBNS protocol and how does it work?**

The NBNS was implemented as [WINS](https://web.archive.org/web/20031010135027/http://www.neohapsis.com:80/resources/wins.htm#sec4sub4sub4) (Windows Internet Name Service). In a network, each Windows computer has a WINS database that stores the available network resources, netbios and domain name.

So in order to resolve a NetBIOS name we can query using the WINS server or If this is not possible, then the query can be sent to the IP broadcast address, waiting for the answer from the target computer (but dangerous since anyone can respond, this is used by [responder.py](https://github.com/lgandx/Responder))

**What is LLMNR?**

LLMNR is a descentralized application protocol that allows to resolve hostnames in the same local network. A common vuln with LLMNR is it is used to resolve names in local link by sending A DNS querie that anyone could repond to, responder.py also abuses this feature

**What is mDNS?**

[mDNS](https://en.wikipedia.org/wiki/Multicast_DNS) (multicast DNS) is a descentralized application protocol, similar to LLMNR that allows to resolve names in local networks

**What is WPAD?**

The [WPAD](https://en.wikipedia.org/wiki/Web_Proxy_Auto-Discovery_Protocol) (Web Proxy Auto-Discovery) is a protocol for browsers to get dynamically a file that indicates the proxies they should use.

**What is the difference between those protocols?**

While they all perform name resolution, the primary difference lies in their order of preference for resolving names:

1. **DNS (Domain Name System)**
2. **mDNS (Multicast DNS)**
3. **LLMNR (Link-Local Multicast Name Resolution)**
4. **NBNS (NetBIOS Name Service)**

Each protocol has a specific context and method for resolving names, which dictates its usage priority in network communications.

### Authentication <a href="#authentication" id="authentication"></a>

In Active Directory there are two network authentication protocols available: [NTLM](https://zer1t0.gitlab.io/posts/attacking_ad/#ntlm) and [Kerberos](https://zer1t0.gitlab.io/posts/attacking_ad/#kerberos).

Kerberos is the preferred option to auth domain users but only NTLM can be used to auth local computer users

**What are Windows SSPs?**

SSPs (Security Support Provider), are implemented as dynamic link libraries (DLLs) that are loaded into processes that require security services.

In an Active Directory environment, SSPs are crucial for:

* **User Authentication**: Ensuring that users can securely log in to the network and access resources based on their permissions.
* **Service Authentication**: Verifying the identity of services and applications to prevent unauthorized access and ensure secure communication.
* **Secure Communication**: Protecting data transmitted over the network through encryption and integrity checks.
* **Interoperability**: Supporting different protocols and ensuring compatibility across various systems and applications within the network.

**Give me some names of different SSPs**

* **Kerberos SSP**
  * The [Kerberos SSP](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-kerberos) (kerberos.dll) manages the Kerberos authentication.
* **NTLM SSP**
  * The [NTLMSSP](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm) (msv1\_0.dll) manages NTLM authentication.
* **Negotiate SSP**
  * The [Negotiate](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-negotiate) SSP (secur32.dll) is an intermediary SSP that manages the [SPNEGO](https://zer1t0.gitlab.io/posts/attacking_ad/#spnego) negotiation and delegates the authentication to Kerberos SSP or NTLM SSP

```
                                             Kerberos
                                         .-------------------------.
                                         |      kerberos.dll       |
                                         |-------------------------|
                                         .---           .----      |
                   Negotiate       .---> | GSS-API ---> | Kerberos |
                 .-------------.   |     '---           '----      |
                 | secur32.dll |   |     |                         |
                 |-------------|   |     '-------------------------'
 .---------.     .---          |   |
 |  user   |---->| GSS-API ----|>--|
 | program |     '---          |   |         NTLM
 '---------'     |             |   |     .-------------------------.
                 '-------------'   |     |       msv1_0.dll        |
                                   |     |-------------------------|
                                   |     .---           .----      |
                                   '---> | GSS-API ---> | NTLM     |
                                         '---           '----      |
                                         |                         |
                                         '-------------------------'

```

* **Digest SSP**
  * The [Digest](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-digest-ssp) (wdigest.dll) implements the Digest Access protocol. This is the SSP that caches the plaintext password in old operating systems that can be retrieved by mimikatz.
* **Secure Channel SSP**
  * The [Secure Channel](https://docs.microsoft.com/en-us/windows/win32/secauthn/secure-channel) (schannel.dll) provide encrypted communications.
* **Cred SSP**
  * The [CredSSP](https://docs.microsoft.com/en-us/windows/win32/secauthn/credential-security-support-provider) (credssp.dll) creates a TLS channel, authenticates the client through negotiate SSP, and finally allows the client to send the user full credentials to the server. It is used by [RDP](https://zer1t0.gitlab.io/posts/attacking_ad/#rdp).

**What is SPNEGO?**

SPNEGO, or Simple and Protected GSSAPI Negotiation Mechanism, is a standard protocol used for negotiating security mechanisms, it ensures that the most appropriate and secure authentication protocol is used for each connection. Its ability to seamlessly switch between protocols like Kerberos and NTLM

**What is NTLM?**

[NTLM](http://davenport.sourceforge.net/ntlm.html) (NT LAN Manager) is an authentication protocol that can be used by Windows services in order to verify the identity of the client. It's good to know that NTLM is not an isolated protocol that generates network traffic, but must be used embebed in an application protocol, such as [SMB](https://zer1t0.gitlab.io/posts/attacking_ad/#smb), [LDAP](https://zer1t0.gitlab.io/posts/attacking_ad/#ldap) or [HTTP](https://zer1t0.gitlab.io/posts/attacking_ad/#http).

**What are the 3 stages of NTLM connection?**

NEGOTIATE, CHALLENGE and AUTHENTICATE.

```
client               server
  |                    |
  |                    |
  |     NEGOTIATE      |
  | -----------------> |
  |                    |
  |     CHALLENGE      |
  | <----------------- |
  |                    |
  |    AUTHENTICATE    |
  | -----------------> |
  |                    |
  |    application     |
  |      messages      |
  | -----------------> |
  | <----------------- |
  | -----------------> |
  |                    |
```

**How does NTLM work more in detail?**

* Firstly, the client sends a NEGOTIATE message to the server. It indicates security options, like the NTLM version to use.
* The server generates a challenge by calling the NTLM SSP, and sends it to the client within a CHALLENGE message. Also confirms the negotiated options and sends information about its computer name and version and domain name.
* The client receives the challenge and calculates a response by using the client key (NT hash). If it is required, it also creates a session key and encrypts it with a key, known as session base key, derivated from NT hash and sends the response and session key back to the server.
* Finally, the server verifies that the challenge response is correct and sends a session key

```
                         client               server
                           |                    |
 AcquireCredentialsHandle  |                    |
           |               |                    |
           v               |                    |
 InitializeSecurityContext |                    |
           |               |     NEGOTIATE      |
           '-------------> | -----------------> | ----------.
                           |     - flags        |           |
                           |                    |           v
                           |                    | AcceptSecurityContext
                           |                    |           |
                           |                    |       challenge
                           |     CHALLENGE      |           |
           .-------------- | <----------------- | <---------'
           |               |   - flags          |
       challenge           |   - challenge      |
           |               |   - server info    |
           v               |                    |
 InitializeSecurityContext |                    |
       |       |           |                    |
    session  response      |                    |
      key      |           |    AUTHENTICATE    |
       '-------'---------> | -----------------> | ------.--------.
                           |   - response       |       |        |
                           |   - session key    |       |        |
                           |     (encrypted)    |   response  session
                           |   - attributes     |       |       key
                           |     + client info  |       |        |
                           |     + flags        |       v        v
                           |   - MIC            | AcceptSecurityContext
                           |                    |           |
                           |                    |           v
                           |                    |           OK
                           |                    |
```

**What is the difference between NTLMv1 and NTLMv2?**

In [NTLMv1](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/464551a8-9fc4-428e-b3d3-bc5bfb2e73a5), the NTLM response (NTLMv1 hash) to the server challenge is calculated by using the NT hash to encrypt the server challenge with the DES algorithm. The session key is also encrypted with the NT hash directly.

However, in [NTLMv2](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/5e550938-91d4-459f-b67d-75d70009e3f3) more data is taken into to protect the integrity of the AUTHENTICATE message

NTLMv2 concatenates all the additional data and applies an HMAC to calculate the NTLM response, known as NTLMv2 hash.

**How does NTLM interract in Active Directory?**

It's good to remember that the NT hash is stored in the Active Directory [database](https://zer1t0.gitlab.io/posts/attacking_ad/#database), located in the [Domain Controllers](https://zer1t0.gitlab.io/posts/attacking_ad/#domain-controllers)

in order to verify the AUTHENTICATE message for a domain account, the target machine will send a request to the DC asking it to verify the client response to the challenge. The DC verifies this response and returns the necessary information to the machine

```
client            server                          DC
    |                 |                              |
    |                 |                              |
    |    NEGOTIATE    |                              |
    | --------------> |                              |
    |                 |                              |
    |    CHALLENGE    |                              |
    | <-------------- |                              |
    |                 |                              |
    |   AUTHENTICATE  |  NetrLogonSamLogonWithFlags  |
    | --------------> | ---------------------------> |
    |                 |                              |
    |                 |        ValidationInfo        |
    |                 | <--------------------------- |
    |                 |                              |
    |   application   |                              |
    |    messages     |                              |
    | --------------> |                              |
    |                 |                              |
    | <-------------- |                              |
    |                 |                              |
    | --------------> |                              |
    |                 |                              |
```

It can also be used for machines in other domains, it must ask to the DC to verify the AUTHENTICATE message, and the DC in turn must send the AUTHENTICATE message to the DC of the user account domain (by using a [trust](https://zer1t0.gitlab.io/posts/attacking_ad/#trusts))

```
  client            server                          DC                      DC
 (it.foo.com)     (foo.com)                      (foo.com)         (it.foo.com)
    |                 |                              |                       |
    |                 |                              |                       |
    |    NEGOTIATE    |                              |                       |
    | --------------> |                              |                       |
    |                 |                              |                       |
    |    CHALLENGE    |                              |                       |
    | <-------------- |                              |                       |
    |                 |                              |                       |
    |   AUTHENTICATE  |  NetrLogonSamLogonWithFlags  |  NetrLogonSamLogonEx  |
    | --------------> | ---------------------------> | --------------------> |
    |                 |                              |                       |
    |                 |      ValidationInfo          |    ValidationInfo     |
    |                 | <--------------------------- | <-------------------- |
    |                 |                              |                       |
    |   application   |                              |                       |
    |    messages     |                              |                       |
    | --------------> |                              |                       |
    |                 |                              |                       |
    | <-------------- |                              |                       |
    |                 |                              |                       |
    | --------------> |                              |                       |
    |                 |                              |                       |
```

**How do we force NTLM authentication over Kerberos?**

A way to force NTLM authentication over Kerberos is to connect to the target machine by using the IP address instead of the hostname, since Kerberos requires the hostname to identify the machine services.

For example the command `dir \\dc01\C$` will use Kerberos to authenticate against the remote [share](https://zer1t0.gitlab.io/posts/attacking_ad/#shares) while `dir \\192.168.100.2\C$` will use NTLM.

**How does NTLM brute-force work?**

We now know that NTLM is used for authenticating, so we could use plenty of tools ([hydra](https://github.com/vanhauser-thc/thc-hydra), [nmap](https://nmap.org/nsedoc/scripts/smb-brute.html), [cme](https://github.com/byt3bl33d3r/CrackMapExec/wiki/SMB-Command-Reference#using-usernamepassword-lists), or [Invoke-Bruteforce.ps1](https://github.com/samratashok/nishang/blob/master/Scan/Invoke-BruteForce.ps1)) to test for a working password:

```
$ cme smb 192.168.100.10 -u elfelixio -p passwords.txt 
SMB         192.168.100.10  445    WS01-10          [*] Windows 10.0 Build 19041 x64 (name:WS01-10) (domain:contoso.local) (signing:False) (SMBv1:False)
SMB         192.168.100.10  445    WS01-10          [-] contoso.local\anakin:1234 STATUS_LOGON_FAILURE 
SMB         192.168.100.10  445    WS01-10          [-] contoso.local\anakin:Vader! STATUS_LOGON_FAILURE 
SMB         192.168.100.10  445    WS01-10          [+] contoso.local\anakin:Vader1234! (Pwn3d!)
```

We also need to remember that in active directory, authentication works by verifying credentials with the DC so this can lead to account block and will lead to a lot of logs

**How does pass the hash woks?**

Since NTLM calculates the NTLM hash and the session key based on the NT hash, we can use this hash to athenticate and impersonate the user even without cleartext password

On a linux host we could use impacket that accepts the hashes as parameter:

```
$ psexec.py contoso.local/Anakin@192.168.100.10 -hashes :cdeae556dc28c24b5b7b14e9df5b6e21
Impacket v0.9.21 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 192.168.100.10.....
[*] Found writable share ADMIN$
[*] Uploading file WFKqIQpM.exe
[*] Opening SVCManager on 192.168.100.10.....
[*] Creating service AoRl on 192.168.100.10.....
[*] Starting service AoRl.....
[!] Press help for extra shell commands
The system cannot find message text for message number 0x2350 in the message file for Application.

(c) Microsoft Corporation. All rights reserved.
b'Not enough memory resources are available to process this command.\r\n'
C:\Windows\system32>whoami
nt authority\system
```

**How do we extract the NT hashes?**

To extract NT hashes from lsass you can use [mimikatz sekurlsa::logonpasswords](https://github.com/gentilkiwi/mimikatz/wiki/module-~-sekurlsa#logonpasswords) command, [dump the lsass process](https://www.c0d3xpl0it.com/2016/04/extracting-clear-text-passwords-using-procdump-and-mimikatz.html) with tools like [procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump), [sqldumper or others](https://lolbas-project.github.io/#/dump), and copy the dump to your local machine to read it with [mimikatz](https://github.com/gentilkiwi/mimikatz), [pypykatz](https://github.com/skelsec/pypykatz) or [read the dump remotely](https://en.hackndo.com/remote-lsass-dump-passwords/) with [lsassy](https://github.com/Hackndo/lsassy), extract the hashes [from the local SAM database](https://www.hackingarticles.in/credential-dumping-sam/) or the [NTDS.dit database in Domain Controllers](https://www.hackingarticles.in/credential-dumping-ntds-dit/)

**How does NTLM Relay work?**

{% embed url="https://en.hackndo.com/ntlm-relay/" %}

This attack consists in an attacker redirecting the NTLM authentication to a server of its interest to get an authenticated session (Person-in-The-Middle attack)

```
  client                 attacker               server
      |                       |                     |
      |                       |                -----|--.
      |     NEGOTIATE         |     NEGOTIATE       |  |
      | --------------------> | ------------------> |  |
      |                       |                     |  |
      |     CHALLENGE         |     CHALLENGE       |  |> NTLM Relay
      | <-------------------- | <------------------ |  |
      |                       |                     |  | 
      |     AUTHENTICATE      |     AUTHENTICATE    |  |
      | --------------------> | ------------------> |  |
      |                       |                -----|--'
      |                       |    application      |
      |                       |     messages        |
      |                       | ------------------> |
      |                       |                     |
      |                       | <------------------ |
      |                       |                     |
      |                       | ------------------> |
      |                       |                     |
```

Even after getting authenticated, the attacker does not know the session key so if signing is negotiated between client and server, the attacker won't be able to generate valid signatures for the application messages

**How to crack NTLM hashes?**

When trying to do Person-in-The-Middle attacks we could use [Responder.py](https://github.com/lgandx/Responder) or [Inveigh](https://github.com/Kevin-Robertson/Inveigh) to try and grab the hash

{% embed url="https://felix-billieres.gitbook.io/felix-billieres/certification-prep/cpts/active-directory-enumeration-and-attacks/sniffing-out-a-foothold#start-using-inveigh" %}
You can find some examples here
{% endembed %}

Then we could crack the hash using hashcat and the mode 5600

{% embed url="https://felix-billieres.gitbook.io/felix-billieres/certification-prep/cpts/active-directory-enumeration-and-attacks/sniffing-out-a-foothold#using-responder" %}
you can also find examples of hash cracking here
{% endembed %}

It's good to know that NTLMv1 hashes are faster to crack than NTLMv2 hashes since they are created with weaker algorithms.

### Kerberos <a href="#kerberos" id="kerberos"></a>

**Whart are kerberos principal types?**

A principal is a unique identity to which Kerberos can assign tickets. Principals typically represent users, services, or hosts within the Kerberos realm. There are three principal types that can be used to request for a service: **NT-SRV-INST, NT-SRV-HST or NT-SRV-XHST**

**What is a ticket in kerberos?**

[Tickets](https://tools.ietf.org/html/rfc4120#section-5.3) are structures partially encrypted that contain:

* The target principal (usually a service) for which the ticket applies
* Information related to the client, such as the name and domain
* A key to establish secure channels between the client and the service
* Timestamps to determine the period in which the ticket is valid

**What is the PAC in active directory?**

The Privileged Attribute Certificate (PAC) is an extension to Kerberos service tickets that holds information about the user and their privileges. It is added by a domain controller in an Active Directory domain when the user authenticates.

**What informations does the PAC contain?**

The PAC includes informations about the client:

* domain name and [SID](https://zer1t0.gitlab.io/posts/attacking_ad/#sid)
* The username and user [RID](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/f2ef15b6-1e9b-48b5-bf0b-019f061d41c8#gt_df3d0b61-56cd-4dac-9402-982f1fedc41c)
* The [RIDs](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/f2ef15b6-1e9b-48b5-bf0b-019f061d41c8#gt_df3d0b61-56cd-4dac-9402-982f1fedc41c) (GroupIds) of those domain groups to which the user belongs.
* other [SIDs](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/78eb9013-1c3a-4970-ad1f-2b1dad588a25) (ExtraSids) of non-domain groups, that can be applied for inter-domain authentications

And also [several signatures](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/6e95edd3-af93-41d4-8303-6c7955297315) used to verify the integrity of the PAC and ticket data:

* [Server signature](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/a194aa34-81bd-46a0-a931-2e05b87d1098) (created with the same key used to encrypt the ticket)
* [KDC signature](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/3122bf00-ea87-4c3f-92a0-91c0a99f5eec): A signature of the Server signature created with the KDC key. This could be used to check that the PAC was created by the KDC and prevent [Silver ticket](https://zer1t0.gitlab.io/posts/attacking_ad/#golden-silver-ticket) attacks, but is not checked.
* [Ticket signature](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-pac/76c10ef5-de76-44bf-b208-0d8750fc2edd): A signature of the ticket content created with the KDC key. This signature was recently introduced to prevent the [Bronze bit attack](https://blog.netspi.com/cve-2020-17049-kerberos-bronze-bit-theory/).

{% embed url="https://www.lepide.com/blog/what-is-privileged-attribute-certificate-pac/" %}

**What are the 3 actors Kerberos uses to authenticate users against services**

* **The client,** it's the user that recieves the ticket and gains access to a service in the domain
* The **service or Application server,** that is the machine that offers the sevice a user wants to access
* The **KDC** (Key Distribution Center), the entity that has access to the databse required to authenticate users

**What are the 2 types of tickets?**

**STs** (Service tickets), that a client presents to a AP/service/principal in order to get access to it. The KDC issues the STs for clients that request for them.

We need to be aware that TGSs are not STs, according to [rfc4120](https://datatracker.ietf.org/doc/html/rfc4120/) the TGS refers to the service that provides the service tickets.

The other type of ticket is the **TGT**. In order to get a ST from the KDC, we need to present our TGT.

TGTs are encrypted with the key of the `krbtgt` account of the domain, known as the KDC key. Therefore, if you can retrieve the key of the `krbtgt` (stored in the [domain database](https://zer1t0.gitlab.io/posts/attacking_ad/#domain-database-dumping)), you could create custom TGTs known as [Golden tickets](https://zer1t0.gitlab.io/posts/attacking_ad/#golden-silver-ticket).

**How are tickets issued?**

**What are some kerberos services related to kerberos on port 88 and 464**

The DC listens Kerberos in the port 88/TCP and 88/UDP.

Another service called kpasswd can be found in the port 464/TCP and 464/UDP of the DCs that allows to change the password of the users in the domain

**What happens to the Kerberos keys when you change your password?**

By changing the password, the user changes the Kerberos keys used for encrypting the Kerberos messages and tickets.

**Basic Attacks**

{% embed url="https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a" fullWidth="true" %}

**What are some Kerberos erros related to Brute Force?**

* KDC\_ERR\_PREAUTH\_FAILED: Incorrect password
* KDC\_ERR\_C\_PRINCIPAL\_UNKNOWN: Invalid username
* KDC\_ERR\_WRONG\_REALM: Invalid domain
* KDC\_ERR\_CLIENT\_REVOKED: Disabled/Blocked user

**What are some tools used for kerberos bruteforce?**

[Rubeus brute](https://github.com/GhostPack/Rubeus#brute), [kerbrute (Go)](https://github.com/ropnop/kerbrute), [kerbrute (Python)](https://github.com/TarlogicSecurity/kerbrute) or [cerbero](https://github.com/Zer1t0/cerbero#brute)

**What is kerberoasting?**

{% embed url="https://felix-billieres.gitbook.io/felix-billieres/certification-prep/cpts/active-directory-enumeration-and-attacks/kerberoasting" %}

Most services are registered in machine accounts, which have auto-generated passwords of [120 characters that changes every month](https://adsecurity.org/?p=280) so cracking = impossible. But some services are assigned to regular user so the ST is encrypted with their personal password that is (normally) not 120 chars, so, far more crackable

The [Kerberoast attack](https://en.hackndo.com/kerberoasting/) consist on requests STs for the services of regular user accounts and try to crack them to get the user passwords. Usually, the users that have services also have privileges, so these are juicy accounts.

For this attack we can use [impacket GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) script, the [Rubeus kerberoast](https://github.com/GhostPack/Rubeus#kerberoast) command, or the [Invoke-Kerberoast.ps1](https://github.com/EmpireProject/Empire/blob/master/data/module_source/credentials/Invoke-Kerberoast.ps1) script

**What is ASREProasting?**

Asreproasting tool from the goat: [https://github.com/Yaxxine7/ASRepCatcher/](https://github.com/Yaxxine7/ASRepCatcher/)

When Kerberos pre-authentication is disabled, anyone can impersonate those accounts by sending a AS-REQ message, and an [AS-REP response](https://tools.ietf.org/html/rfc4120#section-5.4.2) will be returned from the KDC

We could use tools such as [impacket GetNPUsers.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py) script, the [Rubeus asreproast](https://github.com/GhostPack/Rubeus#asreproast) command or the [ASREPRoast.ps1](https://github.com/HarmJ0y/ASREPRoast) script then crack the hash locally

**Explain Pass the Key/Over Pass the Hash.**

{% embed url="https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/over-pass-the-hash-pass-the-key" %}

To request a TGT we don't need a password but a Kerberos key, if we can steal a key we can request a TGT as the user.

On windows the Kerberos keys are cached in the lsass process, and them can be retrieved by using the [mimikatz sekurlsa::ekeys](https://github.com/gentilkiwi/mimikatz/wiki/module-~-sekurlsa#ekeys) command. Also you can [dump the lsass process](https://www.c0d3xpl0it.com/2016/04/extracting-clear-text-passwords-using-procdump-and-mimikatz.html) with tools like [procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump), [sqldumper or others](https://lolbas-project.github.io/#/dump), and extract the keys offline with mimikatz.

On Linux, the Kerberos keys are stored in [keytab](https://web.mit.edu/kerberos/krb5-devel/doc/basic/keytab_def.html) files. The keytab file can be usually found in `/etc/krb5.keytab`, or in the value specified by the environment variables `KRB5_KTNAME` or `KRB5_CLIENT_KTNAME`, or specified in the [Kerberos configuration file](https://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf_files/krb5_conf.html) in `/etc/krb5.conf`.

When the keytab is located, we can copy and/or list its keys with [klist](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/klist.html) or [cerbero](https://gitlab.com/Zer1t0/cerbero/#list)

With the keys, we can request a TGT (With windows) in Windows by using [Rubeus asktgt](https://github.com/GhostPack/Rubeus#asktgt)

Or (on linux) using the key directly with the [impacket getTGT.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/getTGT.py) script or [cerbero ask](https://gitlab.com/Zer1t0/cerbero#ask) command

The Kerberos tickets have two formats: **ccache and krb**. The ccache is the one used by Linux and krb format is used in Windows. We can convert tickets to one format to another by using the [ticket\_converter.py](https://github.com/Zer1t0/ticket_converter) script or [cerbero convert](https://gitlab.com/Zer1t0/cerbero#convert)

**What is Pass the Ticket?**

The Pass the Ticket technique consists on steal a ticket and the associated session key and use them to impersonate the user in order to access to resources or services.

**What are Golden/Silver tickets?**

In Active Directory, the Kerberos TGTs are encrypted with the `krbtgt` account keys. In case of knowing the keys, custom TGTs, known as [Golden Tickets](https://en.hackndo.com/kerberos-silver-golden-tickets/#golden-ticket), can be created.

To get the `krbtgt` keys, you need to access to the Active Directory database. You can do this by performing a remote [dcsync attack](https://adsecurity.org/?p=1729), with [mimikatz lsadump::dsync](https://github.com/gentilkiwi/mimikatz/wiki/module-~-lsadump#dcsync) command or the [impacket secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py) script, or by [dumping the NTDS.dit file](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/ntds.dit-enumeration#no-credentials-ntdsutil) locally with [ntdsutil](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc753343\(v=ws.11\)) or [vssadmin](https://docs.microsoft.com/en-gb/windows-server/administration/windows-commands/vssadmin).

{% embed url="https://felix-billieres.gitbook.io/felix-billieres/active-directory/attacking-vectors/methodology/compromised-the-domain-now-what/golden-ticket-attacks" %}

**How does kerberos work with services accross domains?**

A KDC (DC) only can issue STs for the services in its domain. So accros domains we have to ask for a ST to the external domain DC, and for that we require a TGT for that server. The TGT for an external KDC, known as **inter-realm TGT,** is issued by our KDC when we ask for a ST for a service in another domain.

here is the schema:

```
  KDC foo.com                                                    KDC bar.com
    .---.                                                          .---.
   /   /|                       .---4) TGS-REQ (TGT bar)------->  /   /|
  .---. |                       |    + SPN: HTTP\srvbar          .---. |
  |   | '                       |    + TGT client > bar.com      |   | '
  |   |/                        |                                |   |/ 
  '---'                         |   .--5) TGS-REP--------------< '---'
  v  ^                          |   | + ST client > HTTP/srvbar
  |  |                          |   |
  |  |                          ^   v                                   .---.
  |  '-2) TGS-REQ (TGT foo)--<  _____                                  /   /|
  |   + SPN: HTTP\srvbar       |     | <----------1) SPNEGO---------> .---. |
  |   + TGT client > foo.com   |_____|                                |   | '
  |                            /:::::/ >----6) AP-REQ---------------> |   |/
  '--3) TGS-REP--------------> client     + ST client > HTTP/srvbar   '---'  
    + TGT client > bar.com    (foo.com)                               srvbar
                                                                    (bar.com)
```

**Can you tell me more about Inter-realm TGTs?**

{% embed url="https://book.hacktricks.xyz/windows-hardening/active-directory-methodology#forest-privilege-escalation-domain-trusts" %}

This TGT is exactly like a normal one, except that it is encrypted with the inter-realm trust key, which is a secret key that allows to both sides of a trust to comunicate between them.

You can get the inter-realm trust key, you just need [to dump the domain database](https://zer1t0.gitlab.io/posts/attacking_ad/#domain-database-dumping).

Finally, once you get the trust key, to [create a inter-realm ticket](https://adsecurity.org/?p=1588), you can use the [mimikatz kerberos::golden](https://github.com/gentilkiwi/mimikatz/wiki/module-~-kerberos#golden--silver) command or the [impacket ticketer.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketer.py) script. Then you can use it as any ticket.

**Kerberos Delegation**

**Can you explain kerberos delegation?**

Let's say we are a user on a website. When we make a request the webserver goes and fetch the document we're looking for but how does it know what we own on the file server. The delegation will make it possible for the web server to "impersonate" the user and athenticate as the user to go and fetch the ressources. The file server will think that it's the user that directly made the request:

**What's the difference between Contstrained and Unconstrained Delegation?**

In [Kerberos Unconstrained Delegation](https://adsecurity.org/?p=1667) the service can impersonate the client user since this sends its own TGT to the service. Then, the service can use the user TGT (without any constrain) to ask for new STs for other services in behalf of the client.

To create a more secure way of delegation, Microsoft develop two Kerberos extensions known as [Service for User](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-sfu/3bff5864-8135-400e-bdd9-33b552051d94) (S4U). By using these extensions, services can be restricted to only perform delegation against a set of allowed third-services, and no user TGT is required, preventing it from being stored on the service server. This is known as constrained delegation.

### Logon types <a href="#logon-types" id="logon-types"></a>

**Explain why there are different logon types.**

In order to logon users, both locally and remotely, Windows defines different types of logons. It's good to know not every logons can be used by any user and [many logons cache credentials in the lsass process](https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/reference-tools-logon-types) , or even in the LSA secrets. That can be interestin as a pentester to recover that.

**Can you explain the different logon types?**

1. **Interactive Logon**: An interactive logon occurs when a user logs on to a system or network using a username and password, usually through a graphical user interface (GUI) or command-line interface (CLI). This type of logon allows the user to interact with the system, access files, and perform tasks.
2. **Network Logon**: A network logon is a type of logon that occurs when a system or device connects to a network using a username and password. This type of logon allows the system or device to access network resources, such as shared folders, printers, or internet connectivity.
3. **Batch Logon**: A batch logon is a type of logon that occurs when a program or script runs on a system using a username and password. This type of logon is often used for automated tasks, such as batch processing, data backups, or system maintenance.
4. **Service Logon**: A service logon is a type of logon that occurs when a system service or daemon starts up and runs under a specific username and password. This type of logon allows the service to perform its intended function, such as providing network services, managing system resources, or running background processes.
5. **NetworkCleartext Logon**: A NetworkCleartext logon is a type of logon that occurs when a system or device connects to a network using a username and password in plain text. This type of logon is considered insecure as it transmits sensitive information, such as passwords, in an unencrypted format.
6. **NewCredentials Logon**: A NewCredentials logon is a type of logon that occurs when a user logs on to a system or network using a new set of credentials, such as a new username and password. This type of logon is often used for authentication and authorization purposes, such as when a user changes their password or when a new user is added to a system.
7. **RemoteInteractive Logon**: A RemoteInteractive logon is a type of logon that occurs when a user logs on to a system or network remotely using a username and password, often through a remote desktop protocol (RDP) or virtual private network (VPN). This type of logon allows the user to interact with the system as if they were physically present at the system.

### Authorization <a href="#authorization" id="authorization"></a>

We are now athenticated on the domain. Now technically the domain, service or programs must be aware of our permissions and can decide thanks to those if we can have access to certain objects ->

**What are security descriptors?**

Security descriptors are associated to each object in the Active Directory. It's checked when we have to verify if a user has access to a certain object of the Active Directory. It is stored in a [binary format](https://www.gabescode.com/active-directory/2019/07/25/nt-security-descriptors.html), but it can also be translated to a [Security Descriptor String Format](https://docs.microsoft.com/en-us/windows/win32/secauthz/security-descriptor-string-format).

**What is the difference between ACLs and ACEs?**

{% embed url="https://book.hacktricks.xyz/v/fr/windows-hardening/windows-local-privilege-escalation/acls-dacls-sacls-aces" %}

* **ACLs (Access Control Lists)**: These are lists that specify the permissions granted or denied to different users and groups for a particular object. They define who can access the object and what actions they can perform.
* **ACEs (Access Control Entries)**: These are individual entries within an ACL. Each ACE defines the permissions for a specific user or group. Multiple ACEs make up the ACL for an object.

ACLs provide the overall permission structure for an object, and ACEs define the specific permissions for each user or group within that structure.

**Privileges**

**What are some dangers of misconfigured privileges?**

In AD, [some privileges can be also abused](https://adsecurity.org/?p=3700) and lead to malicious actions like the **SeBackupPrivilege that** allows to read any file of a domain controller, in order to backup it, which could be used to read the domain database.

**SeDebugPrivilege** which allows to debug any process in the machine, so we could inject code in any process, which could lead to privilege escalation or **SeRestorePrivilege** that allows to write any file on the domain controller from a backup. This could allow an attacker to modify the database of the domain.

Here is an interesting article with POCs about privilege abuse: [https://github.com/hatRiot/token-priv/blob/master/abusing\_token\_eop\_1.0.txt](https://github.com/hatRiot/token-priv/blob/master/abusing_token_eop_1.0.txt)

### Group Policy <a href="#group-policy" id="group-policy"></a>

**What are Group policies?**

The [Group Policy](https://adsecurity.org/?p=2716) is a mechanism that allows to apply a set of rules/actions to the Active Directory network users and computers. In order to define the rules, you can create Group Policy Objects (GPOs). Each GPO defines a series of policies that can be applied to specific machines of the domains.

**What are some actions you can make through Group policies?**

* Disable NTLM
* Require password complexity
* Execute an scheduled or immediate task
* Create local users in computers
* Set a default wallpaper
* Synchronize files with OneDrive

#### Communication Protocols <a href="#communication-protocols" id="communication-protocols"></a>

**What do you know about SMB?**

[SMB](https://en.wikipedia.org/wiki/Server_Message_Block#SMB_/_CIFS_/_SMB1) (Server Message Block) is a protocol used to share files and communicate between machines on **port 445**

From a pentester POV it's an interesting protocol since shares can contain valuable informations.

Shares are like folders that a machine shares in order to be accessed by other computers/users in the network.

**What do you know about HTTP in an active directory context?**

HTTP is the protocol of the web.

It is used as transport protocol by many other application protocols that are present in a Active Directory domain like [WinRM](https://zer1t0.gitlab.io/posts/attacking_ad/#winrm) (and thus [Powershell Remoting](https://zer1t0.gitlab.io/posts/attacking_ad/#powershell-remoting)), [RPC](https://zer1t0.gitlab.io/posts/attacking_ad/#rpc) or [ADWS](https://zer1t0.gitlab.io/posts/attacking_ad/#adws) (Active Directory Web Services).

HTTP supports authentication with both NTLM and Kerberos. This is important from a security perspective since it implies that HTTP connections are susceptible of suffering from Kerberos Delegation or [NTLM Relay](https://en.hackndo.com/ntlm-relay/#what-can-be-relayed) attacks.

**What can you say about RPC?**

Remote Procedure Call (RPC) is a protocol used in Active Directory (AD) for communication between client and server systems, as well as between different services within the AD infrastructure. RPC allows a program on one computer to execute code on a remote system, facilitating various functions and services essential to AD operations.

**What can you say about WinRM**

Windows Remote Management (WinRM) is a protocol used for remote management of Windows-based systems, including those within an Active Directory (AD) environment. WinRM is based on the Web Services-Management (WS-Man) protocol, which provides a standard way for systems to interact with each other over a network.

WinRM enables administrators to perform remote management tasks, such as running scripts, accessing event logs, and managing services on remote systems within an AD domain.

**What can you say about SSH?**

SSH (Secure Shell) is a protocol used for accessing and managing Unix systems like Linux. Even if it is not related with Active Directory directly, usually many Linux machines deployed in a domain could be accessed through SSH. It allows the user get a shell on a remote system, transfer files (with the [scp](https://linux.die.net/man/1/scp) utility) and establishing SSH tunnels.

**What are 3 types of port forwarding with SSH?**

Local Port Forwarding

* The goal of local port forwarding is ccessing a service running on a remote server as if it were running locally. Imagine you want to access a web server running on `remote_host`'s port 80 through your local port 8080. After performing the port forwarding, accessing `http://localhost:8080` in your browser would connect to `remote_host`'s web server.

Remote Port Forwarding

* The goal of remote port forwarding is to Redirect a remote port to a local destination and allowing access to a local service from a remote server. Imagine You want a remote user to access a web server running on your local machine's port 8080. After executing the command, accessing `http://ssh_server:8080` would connect to your local machine's web server.

Dynamic Port Forwarding

* The goal of Dynamic port forwarding is to Create a SOCKS proxy that routes multiple connections through an SSH tunnel. The goal is to securely tunnel traffic from various applications through a single SSH connection. Imagine You want to route your web traffic through an SSH server to browse securely. After executing the command, configuring your web browser to use `localhost:8080` as a SOCKS proxy will route its traffic through the SSH server.

For now, it is all. More will be added soon after OSCP, stay tuned.
