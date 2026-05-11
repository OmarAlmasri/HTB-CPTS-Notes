# SSH for Windows: plink.exe

`Plink` is a Windows Command-Line SSH tool that comes with the PuTTY package.
`Plink` can also be used to create <span style="color:rgb(0, 176, 80)">Dynamic Port Forwards</span> and <span style="color:rgb(0, 176, 80)">SOCKS Proxies</span>.
#### Using Plink.exe

On our Windows attack host:

```powershell
plink.exe -ssh -D 9050 ubuntu@10.129.15.50
```

Another Windows-based tool called [Proxifier](https://www.proxifier.com/) can be used to start a SOCKS tunnel via the SSH session we created. `Proxifier` operates through SOCKS or HTTPS proxy and allows for proxy chaining.

---
# SSH Pivoting with `Sshuttle`

[Sshuttle](https://github.com/sshuttle/sshuttle) is a tool written in Python, it removes the need to configure `Proxychains`. This tool only works over SSH and does not provide other options like TOR or HTTPS.

`Sshuttle` can be extremely useful for automating the execution of iptables and adding pivot rules for the remote host.

We don't need to use `Proxychains` with `Sshuttle` to connect to remote host.
#### Installing sshuttle

```sh
sudo apt-get install sshuttle
```
#### Running sshuttle

We use `-r` to connect to the remote machine

```sh
sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v
```

```ad-info
With this command, sshuttle creates an entry in our `iptables` to redirect all traffic to the 172.16.5.0/23 network through the pivot host.
```

Now we can use any tool we want without `Proxychains`

---
# Web Server Pivoting with Rpivot

[Rpivot](https://github.com/klsecservices/rpivot) is a reverse SOCKS proxy tool written in Python for SOCKS tunneling.

![[RPivot.png]]
#### Cloning rpivot

```sh
git clone https://github.com/klsecservices/rpivot.git
```
#### Installing Python2.7

```sh
sudo apt-get install python2.7
```
#### Alternative Installation of Python2.7

```sh
curl https://pyenv.run | bash 
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc 
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc 
echo 'eval "$(pyenv init -)"' >> ~/.bashrc 
source ~/.bashrc 
pyenv install 2.7 
pyenv shell 2.7
```
#### Running server.py from the Attack Host

```sh
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```
#### Transferring rpivot to the Target

```sh
scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/
```
#### Running client.py from Pivot Target

```sh
python2.7 client.py --server-ip 10.10.14.18 --server-port 9999
```
#### Connecting to a Web Server using HTTP-Proxy & NTLM Auth

```sh
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```

---
# Port Forwarding with Windows `Netsh`

We can use `netsh` for:
- `Finding routes`
- `Viewing the firewall configuration`
- `Adding proxies`
- `Creating port forwarding rules`

![[Netsh Port Forwarding.png]]
#### Using Netsh.exe to Port Forward

```powershell
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25
```
#### Verifying Port Forward

```powershell
netsh.exe interface portproxy show v4tov4
```

