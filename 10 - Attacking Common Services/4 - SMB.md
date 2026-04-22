SMB protocol was initially designed to run on top of `NETBIOS` on TCP port `139` and UDP ports `137` & `138`. With **Windows 2000**, Microsoft introduced SMB over TCP/IP on port `445`. The `NETBIOS` option is still supported as a failover.

**Samba** is a Unix/Linux implementation of SMB. Allows Linux servers and Windows clients to use the same SMB services.
## Misconfigurations

### Null Authentication

```sh
smbclient -N -L //IP_HERE
```

- `-N` is used for Null Session
- `-L` is used to list server's shares

Using `smbmap`, we can use `-r` or `-R` to recursively list the content of directories

```sh
smbmap -H IP_HERE -r notes
```

- If we have `READ` privilege it means that we can download files
- If we have `WRITE` privilege it means that we can upload files

```sh
smbmap -H 10.129.14.128 --download "notes\note.txt"
```

```sh
smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"
```

## Remote Procedure Call (RPC)

We can use `rpcclient` tool with null authentication to enumerate the target host.

```sh
rpcclient -U'%' IP_HERE
```

We can also use tools like `enum4linux-ng` for further enumeration.

## Protocol Specifics Attacks

### RCE

[PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) is a tool that lets us execute processes on other systems, complete with full interactivity for console applications, without having to install client software manually. It works because it has a Windows service image inside of its executable. It takes this service and deploys it to the admin$ share (by default) on the remote machine. It then uses the DCE/RPC interface over SMB to access the Windows Service Control Manager API. Next, it starts the PSExec service on the remote machine. The PSExec service then creates a [named pipe](https://docs.microsoft.com/en-us/windows/win32/ipc/named-pipes) that can send commands to the system.

We can download PsExec from [Microsoft website](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec), or we can use some Linux implementations:

- [Impacket PsExec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py) - Python PsExec like functionality example using [RemComSvc](https://github.com/kavika13/RemCom).
- [Impacket SMBExec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py) - A similar approach to PsExec without using [RemComSvc](https://github.com/kavika13/RemCom). The technique is described [here](https://web.archive.org/web/20190515131124/https://www.optiv.com/blog/owning-computers-without-shell-access). This implementation goes one step further, instantiating a local SMB server to receive the output of the commands. This is useful when the target machine does NOT have a writeable share available.
- [Impacket atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py) - This example executes a command on the target machine through the Task Scheduler service and returns the output of the executed command.
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - includes an implementation of `smbexec` and `atexec`.
- [Metasploit PsExec](https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/windows/smb/psexec.md) - Ruby PsExec implementation.

### NetExec (Formerly CrackMapExec)

```ad-note
For authentication on a non domain-joined machine, the **--local-auth** flag can be used.

---

If **--exec-method** is not defined, **atexec** method will be tried. If it fails we can try other methods by specifing **smbexec**
```

#### Enumerate Logged-On Users

There are two ways to enumerate logged-on users:

- `--loggedon-user`
  
```sh
nxc smb <ip> -u <localAdmin> -p <password> --loggedon-users
```

- `--qwinsta`
  
```sh
nxc smb <ip> -u <localAdmin> -p <password> --qwinsta
```

## Forced Authentication Attacks

We can capture user NTLMv2 hashes using `responder`

```sh
responder -I interface_name
```

After capturing the NTLMv2, we can crack it via hashcat using module `5600`

If we cannot crack the hash, we can potentially relay the captured hash to another machine using [impacket-ntlmrelayx](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py) or Responder [MultiRelay.py](https://github.com/lgandx/Responder/blob/master/tools/MultiRelay.py). Let us see an example using `impacket-ntlmrelayx`.

First, we need to set SMB to `OFF` in our responder configuration file (`/etc/responder/Responder.conf`).

Then we execute `impacket-ntlmrelayx` with the option `--no-http-server`, `-smb2support`, and the target machine with the option `-t`. By default, `impacket-ntlmrelayx` will dump the SAM database, but we can execute commands by adding the option `-c`.

```sh
impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146
```

