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

# Spraying, Stuffing, and Defaults

## Password Spraying

[Password spraying](https://owasp.org/www-community/attacks/Password_Spraying_Attack) is a type of brute-force attack in which an attacker attempts to use a single password across many different user accounts. This technique is effective in environments where users are initialized with a default or standard password. (e.g. it's common for administrators to use passwords like `ChangeMe123!`)

```sh
netexec smb 10.100.38.0/24 -u <usernames.list> -p 'ChangeMe123!'
```

## Credential Stuffing

[Credential stuffing](https://owasp.org/www-community/attacks/Credential_stuffing) is another type of brute-force attack in which an attacker uses stolen credentials from one service to attempt access on others.

```sh
hydra -C user_pass.list ssh://10.100.38.23
```

## Default Credentials

There are tools to dedicated to automate the process of providing lists of **default credentials** for provided systems.

One widely used example is the [Default Credentials Cheat Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet), which we can install with `pip3`

```sh
pip3 install defaultcreds-cheat-sheet
```

**Example Usage:**

```sh
creds search linksys
```

![[default creds.png]]

Also remember that default credentials can be found in the product's official docs.

After searching and combining multiple default passwords, we can create a username list and password list and brute-force the target using `hydra`

Beyond applications, default credentials are also commonly associated with routers:

|**Router Brand**|**Default IP Address**|**Default Username**|**Default Password**|
|---|---|---|---|
|3Com|[http://192.168.1.1](http://192.168.1.1/)|admin|Admin|
|Belkin|[http://192.168.2.1](http://192.168.2.1/)|admin|admin|
|BenQ|[http://192.168.1.1](http://192.168.1.1/)|admin|Admin|
|D-Link|[http://192.168.0.1](http://192.168.0.1/)|admin|Admin|
|Digicom|[http://192.168.1.254](http://192.168.1.254/)|admin|Michelangelo|
|Linksys|[http://192.168.1.1](http://192.168.1.1/)|admin|Admin|
|Netgear|[http://192.168.0.1](http://192.168.0.1/)|admin|password|

*The full list can be found here:* [Routers Default Credentials](https://www.softwaretestinghelp.com/default-router-username-and-password-list/)

