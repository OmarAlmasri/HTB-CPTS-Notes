# Network Services

## WinRM

In Windows 10/11, `WinRM` must be implemented and configured manually, it's not done by default for security reasons. By default, `WinRM` uses the TCP ports `5985` (`HTTP`) and `5986` (`HTTPS`).

Common tools for interaction with this protocol: `netexec`, `evil-winrm`

---

## SSH

### Hydra

We can use `hydra` to brute force SSH passwords

```sh
hydra -L user.list -P password.list ssh://10.129.42.197
```

---

## Remote Desktop Protocol (RDP)

On Windows systems via `TCP Port 3389` by default

### Hydra

```sh
hydra -L user.list -P password.list rdp://10.129.42.197
```

We can connect to hosts via RDP using the following tools: `xfreerdp`, `remmina`

---

## SMB

SMB is also known as [Common Internet File System](https://cifs.com/) (`CIFS`). It is part of the SMB protocol and enables universal remote connection of multiple platforms such as Windows, Linux, or macOS.
`Samba` is an opensource implementation of the above functions.

### Hydra

We can also use `hydra` with SMB

```sh
hydra -L user.list -P password.list smb://10.129.42.197
```

We might face errors with old versions of `hydra` because it doesn't support `SMBv3`, we can update it manually, or use **MSF**

```sh
use auxiliary/scanner/smb/smb_login
options
```

