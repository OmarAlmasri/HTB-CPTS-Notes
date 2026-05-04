# SNMP

SNMP Community strings provide information and statistics about a router or device, helping us gain access to it. The manufacturer default community strings of `public` and `private` are often unchanged. In SNMP versions 1 and 2c, access is controlled using a plaintext community string, and if we know the name, we can gain access to it. Encryption and authentication were only added in SNMP version 3. Much information can be gained from SNMP. Examination of process parameters might reveal credentials passed on the command line, which might be possible to reuse for other externally accessible services given the prevalence of password reuse in enterprise environments. Routing information, services bound to additional interfaces, and the version of installed software can also be revealed.

A tool such as [onesixtyone](https://github.com/trailofbits/onesixtyone) can be used to brute force the community string names using a dictionary file of common community strings such as the `dict.txt` file included in the GitHub repo for the tool.

```shell
[!bash!]$ onesixtyone -c dict.txt 10.129.42.254

Scanning 1 hosts, 51 communities
10.129.42.254 [public] Linux gs-svcscan 5.4.0-66-generic #74-Ubuntu SMP Wed Jan 27 22:54:38 UTC 2021 x86_64
```

**Cheat Sheet:**

| **Service Scanning**                                       |                                     |
| ---------------------------------------------------------- | ----------------------------------- |
| `nmap 10.129.42.253`                                       | Run nmap on an IP                   |
| `nmap -sV -sC -p- 10.129.42.253`                           | Run an nmap script scan on an IP    |
| `locate scripts/citrix`                                    | List various available nmap scripts |
| `nmap --script smb-os-discovery.nse -p445 10.10.10.40`     | Run an nmap script on an IP         |
| `netcat 10.10.10.10 22`                                    | Grab banner of an open port         |
| `smbclient -N -L \\\\10.129.42.253`                        | List SMB Shares                     |
| `smbclient \\\\10.129.42.253\\users`                       | Connect to an SMB share             |
| `snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0` | Scan SNMP on an IP                  |
| `onesixtyone -c dict.txt 10.129.42.254`                    | Brute force SNMP secret string      |

