# Enumerating & Retrieving Password Policies

## Enumerating the Password Policy - from Linux - Credentialed

With valid domain credentials, we can know the password policy remotely using `rpcclient` or `crackmapexec (NetExec)`

```sh
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```

## Enumerating the Password Policy - from Linux - SMB NULL Sessions

**SMB NULL** sessions allow an unauthenticated attacker to retrieve information from the domain, such as a <span style="color:rgb(0, 176, 80)">complete listing of users, groups, computers, user account attributes</span>, and the <span style="color:rgb(192, 0, 0)">domain password policy</span>.
#### Using rpcclient

```sh
rpcclient -U "" -N 172.16.5.5

$> querydominfo
```

#### Obtaining the Password Policy using rpcclient - Example

![[querydominfo - password policy.png|434]]

We can also use tools such as `enum4linux-ng` to do full enumeration on common protocols, the result will give us the password policy.

## Enumerating Null Session - from Windows

#### Establish a null session from windows

```powershell
net use \\DC01\ipc$ "" /u:""

The command completed successfully.
```

We can also use a username/password combination to attempt to connect. Let's see some common errors when trying to authenticate:

#### Error: Account is Disabled

```powershell
net use \\DC01\ipc$ "" /u:guest 

System error 1331 has occurred. 
This user can't sign in because this account is currently disabled.
```

#### Error: Password is Incorrect

```powershell
C:\htb> net use \\DC01\ipc$ "password" /u:guest 

System error 1326 has occurred. 
The user name or password is incorrect.
```

#### Error: Account is locked out (Password Policy)

```powershell
net use \\DC01\ipc$ "password" /u:guest 

System error 1909 has occurred. 
The referenced account is currently locked out and may not be logged on to.
```

## Enumerating the Password Policy - from Linux - LDAP Anonymous Bind

With an LDAP anonymous bind, we can use LDAP-specific enumeration tools such as `windapsearch.py`, `ldapsearch`, `ad-ldapdomaindump.py`, etc.
#### Using ldapsearch

```sh
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

In newer versions of `ldapsearch`, the `-h` parameter was deprecated in favor for `-H`.

## Enumerating the Password Policy - from Windows

#### Using net.exe

```powershell
net accounts
```
#### Using PowerView

```powershell
Get-DomainPolicy
```

---

The default password policy when a new domain is created is as follows, and there have been plenty of organizations that never changed this policy:

| Policy                                      | Default Value |
| ------------------------------------------- | ------------- |
| Enforce password history                    | 24 days       |
| Maximum password age                        | 42 days       |
| Minimum password age                        | 1 day         |
| Minimum password length                     | 7             |
| Password must meet complexity requirements  | Enabled       |
| Store passwords using reversible encryption | Disabled      |
| Account lockout duration                    | Not set       |
| Account lockout threshold                   | 0             |
| Reset account lockout counter after         | Not set       |

---
# Password Spraying - Making a Target User List

## Detailed User Enumeration

```ad-tldr
We can get usernames by leveraging: 
- SMB Null sessions. 
- LDAP Anonymous Bind.
- Using `Kerbrute`.
- Using a wordlist such as [statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames) GitHub repo. Or by using a tool such as [linkedin2username](https://github.com/initstring/linkedin2username)

--- 

Don't forget to check the **Password Policy** which is the most important part before doing a Passwrod Spraying attack so we create a good potential password, and to be aware of the **Account Lockout Policy**.
```

## SMB NULL Session to Pull User List

