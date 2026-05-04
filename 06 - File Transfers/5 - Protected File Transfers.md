# File Encryption on Windows

One of the simplest methods is the [Invoke-AESEncryption.ps1](https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions%5CInvoke-AESEncryption.ps1) PowerShell script.

**Import Module**

```PowerShell
Import-Module .\Invoke-AESEncryption.ps1
```

**File Encryption Example:**

```PowerShell
Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
```

# File Encryption on Linux

We can use **OpenSSL** to do so.

**Encrypt with OpenSSL:**

```sh
openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```

**Decrypt with OpenSSL**

```sh
openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
```

