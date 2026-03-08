# SSH
## Dangerous Settings

| **Setting**                  | **Description**                             |
| ---------------------------- | ------------------------------------------- |
| `PasswordAuthentication yes` | Allows password-based authentication.       |
| `PermitEmptyPasswords yes`   | Allows the use of empty passwords.          |
| `PermitRootLogin yes`        | Allows to log in as the root user.          |
| `Protocol 1`                 | Uses an outdated version of encryption.     |
| `X11Forwarding yes`          | Allows X11 forwarding for GUI applications. |
| `AllowTcpForwarding yes`     | Allows forwarding of TCP ports.             |
| `PermitTunnel`               | Allows tunneling.                           |
| `DebianBanner yes`           | Displays a specific banner when logging in. |
## Footprinting the Service

```shell
Ripcord88x@htb[/htb]$ git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
Ripcord88x@htb[/htb]$ ./ssh-audit.py 10.129.14.132

# general
(gen) banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
(gen) software: OpenSSH 8.2p1
(gen) compatibility: OpenSSH 7.4+, Dropbear SSH 2018.76+
(gen) compression: enabled (zlib@openssh.com)   
...SNIP...
```

```shell
Ripcord88x@htb[/htb]$ ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password

OpenSSH_8.2p1 Ubuntu-4ubuntu0.3, OpenSSL 1.1.1f  31 Mar 2020
debug1: Reading configuration data /etc/ssh/ssh_config
...SNIP...
debug1: Authentications that can continue: publickey,password,keyboard-interactive
debug1: Next authentication method: password

cry0l1t3@10.129.14.132's password:
```


# Rsync

[Rsync](https://linux.die.net/man/1/rsync) is a fast and efficient tool for locally and remotely copying files.
By default, it uses port `873` and can be configured to use SSH for secure file transfers by piggybacking on top of an established SSH server connection.
## Scanning for Rsync

```shell
Ripcord88x@htb[/htb]$ sudo nmap -sV -p 873 127.0.0.1

Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-19 09:31 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0058s latency).

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.13 seconds
```

## Probing for Accessible Shares

```shell
Ripcord88x@htb[/htb]$ nc -nv 127.0.0.1 873

(UNKNOWN) [127.0.0.1] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
dev            	Dev Tools
@RSYNCD: EXIT
```

## Enumerating an Open Share

```shell
Ripcord88x@htb[/htb]$ rsync -av --list-only rsync://127.0.0.1/dev

receiving incremental file list
drwxr-xr-x             48 2022/09/19 09:43:10 .
-rw-r--r--              0 2022/09/19 09:34:50 build.sh
-rw-r--r--              0 2022/09/19 09:36:02 secrets.yaml
drwx------             54 2022/09/19 09:43:10 .ssh

sent 25 bytes  received 221 bytes  492.00 bytes/sec
total size is 0  speedup is 0.00
```

From the above output, we can see a few interesting files that may be worth pulling down to investigate further. We can also see that a directory likely containing SSH keys is accessible. From here, we could sync all files to our attack host with the command `rsync -av rsync://127.0.0.1/dev`. If Rsync is configured to use SSH to transfer files, we could modify our commands to include the `-e ssh` flag, or `-e "ssh -p2222"` if a non-standard port is in use for SSH. This [guide](https://phoenixnap.com/kb/how-to-rsync-over-ssh) is helpful for understanding the syntax for using Rsync over SSH.

# R-Services

`R-services` span across the ports `512`, `513`, and `514` and are only accessible through a suite of programs known as `r-commands`.

The [R-commands](https://en.wikipedia.org/wiki/Berkeley_r-commands) suite consists of the following programs:
- rcp (`remote copy`)
- rexec (`remote execution`)
- rlogin (`remote login`)
- rsh (`remote shell`)
- rstat
- ruptime
- rwho (`remote who`)

| **Command** | **Service Daemon** | **Port** | **Transport Protocol** | **Description**                                                                                                                                                                                                                                                            |
| ----------- | ------------------ | -------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `rcp`       | `rshd`             | 514      | TCP                    | Copy a file or directory bidirectionally from the local system to the remote system (or vice versa) or from one remote system to another. It works like the `cp` command on Linux but provides `no warning to the user for overwriting existing files on a system`.        |
| `rsh`       | `rshd`             | 514      | TCP                    | Opens a shell on a remote machine without a login procedure. Relies upon the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files for validation.                                                                                                                 |
| `rexec`     | `rexecd`           | 512      | TCP                    | Enables a user to run shell commands on a remote machine. Requires authentication through the use of a `username` and `password` through an unencrypted network socket. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files. |
| `rlogin`    | `rlogind`          | 513      | TCP                    | Enables a user to log in to a remote host over the network. It works similarly to `telnet` but can only connect to Unix-like hosts. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files.                                     |

## Scanning for R-Services


```shell
Ripcord88x@htb[/htb]$ sudo nmap -sV -p 512,513,514 10.0.17.2

Starting Nmap 7.80 ( https://nmap.org ) at 2022-12-02 15:02 EST
Nmap scan report for 10.0.17.2
Host is up (0.11s latency).

PORT    STATE SERVICE    VERSION
512/tcp open  exec?
513/tcp open  login?
514/tcp open  tcpwrapped

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 145.54 seconds
```

