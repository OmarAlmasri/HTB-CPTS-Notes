
# Credential Hunting in Network Traffic

| **Unencrypted Protocol** | **Encrypted Counterpart**  | **Description**                                                             |
| ------------------------ | -------------------------- | --------------------------------------------------------------------------- |
| `HTTP`                   | `HTTPS`                    | Used for transferring web pages and resources over the internet.            |
| `FTP`                    | `FTPS/SFTP`                | Used for transferring files between a client and a server.                  |
| `SNMP`                   | `SNMPv3 (with encryption)` | Used for monitoring and managing network devices like routers and switches. |
| `POP3`                   | `POP3S`                    | Retrieves emails from a mail server to a local client.                      |
| `IMAP`                   | `IMAPS`                    | Accesses and manages email messages directly on the mail server.            |
| `SMTP`                   | `SMTPS`                    | Sends email messages from client to server or between mail servers.         |
| `LDAP`                   | `LDAPS`                    | Queries and modifies directory services like user credentials and roles.    |
| `RDP`                    | `RDP (with TLS)`           | Provides remote desktop access to Windows systems.                          |
| `DNS (Traditional)`      | `DNS over HTTPS (DoH)`     | Resolves domain names into IP addresses.                                    |
| `SMB`                    | `SMB over TLS (SMB 3.0)`   | Shares files, printers, and other resources over a network.                 |
| `VNC`                    | `VNC with TLS/SSL`         | Allows graphical remote control of another computer.                        |

## Wireshark

| **Wireshark filter**                              | **Description**                                                                                                                                                                      |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `ip.addr == 56.48.210.13`                         | Filters packets with a specific IP address                                                                                                                                           |
| `tcp.port == 80`                                  | Filters packets by port (HTTP in this case).                                                                                                                                         |
| `http`                                            | Filters for HTTP traffic.                                                                                                                                                            |
| `dns`                                             | Filters DNS traffic, which is useful to monitor domain name resolution.                                                                                                              |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0`        | Filters SYN packets (used in TCP handshakes), useful for detecting scanning or connection attempts.                                                                                  |
| `icmp`                                            | Filters ICMP packets (used for Ping), which can be useful for reconnaissance or network issues.                                                                                      |
| `http.request.method == "POST"`                   | Filters for HTTP POST requests. In the case that POST requests are sent over unencrypted HTTP, it may be the case that passwords or other sensitive information is contained within. |
| `tcp.stream eq 53`                                | Filters for a specific TCP stream. Helps track a conversation between two hosts.                                                                                                     |
| `eth.addr == 00:11:22:33:44:55`                   | Filters packets from/to a specific MAC address.                                                                                                                                      |
| `ip.src == 192.168.24.3 && ip.dst == 56.48.210.3` | Filters traffic between two specific IP addresses. Helps track communication between specific hosts.                                                                                 |

**How to locate packets that contain specific bytes or strings?**
We can use display filters like `http contains passw`

![[wireshark1.png]]

Alternatively, we can navigate to `Edit > Find Packet` and enter the desired search query manually.

## Pcredz

[Pcredz](https://github.com/lgandx/PCredz) is a tool that can be used to extract credentials from live traffic or network packet captures. Specifically, it supports extracting the following information:

- Credit card numbers
- POP credentials
- SMTP credentials
- IMAP credentials
- SNMP community strings
- FTP credentials
- Credentials from HTTP NTLM/Basic headers, as well as HTTP Forms
- NTLMv1/v2 hashes from various traffic including DCE-RPC, SMBv1/2, LDAP, MSSQL, and HTTP
- Kerberos (AS-REQ Pre-Auth etype 23) hashes

**Installation:** [GitHub/Pcredz](https://github.com/lgandx/PCredz?tab=readme-ov-file#install)

**Usage:**

We can use this command against a `.pcap` file

```sh
./Pcredz -f demo.pcapng -t -v
```

# Credential Hunting in Network Shares

#### Common credential patterns

- Look for keywords within files such as `passw`, `user`, `token`, `key`, and `secret`.
- Search for files with extensions commonly associated with stored credentials, such as `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, and `.bat`.
- Watch for files with "interesting" names that include terms like `config`, `user`, `passw`, `cred`, or `initial`.
- If you're trying to locate credentials within the `INLANEFREIGHT.LOCAL` domain, it may be helpful to search for files containing the string `INLANEFREIGHT\`.
- Keywords should be localized based on the target; if you are attacking a German company, it's more likely they will reference a `"Benutzer"` than a `"User"`.
- Pay attention to the shares you are looking at, and be strategic. If you scan ten shares with thousands of files each, it's going to take a significant amount of time. Shares used by `IT employees` might be a more valuable target than those used for company photos.

## Hunting from Windows

#### Snaffler

 [Snaffler](https://github.com/SnaffCon/Snaffler). When run on a `domain-joined` machine, it'll automatically search for accessible network searches and interesting files. 

```powershell
snaffler.exe -s
```

To reduce false positives in this tool, we can use:
- `-u` retrieves a list of users from Active Directory and searches for references to them in files
- `-i` and `-n` allow you to specify which shares should be included in the search

#### PowerHuntShares

Doesn't necessarily need to be run on a `domain-joined` machine. It generates an HTML report upon finishing.

![[PowerHuntShares.png]]

```PowerShell
Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
```

## Hunting from Linux

#### MANSPIDER

If we don’t have access to a domain-joined computer, or simply prefer to search for files remotely, tools like [MANSPIDER](https://github.com/blacklanternsecurity/MANSPIDER) allow us to scan SMB shares from Linux.

A basic scan for files containing the string `passw` can be run as follows:

```sh
docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider 10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'
```

#### NetExec

`NetExec` can also be used to search through network shares using the `--spider` option. This functionality is described in great detail on the [official wiki](https://www.netexec.wiki/smb-protocol/spidering-shares). A basic scan of network shares for files containing the string `"passw"` can be run like so:

```sh
nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' --spider IT --content --pattern "passw"
```

