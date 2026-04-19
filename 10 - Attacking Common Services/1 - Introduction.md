# Server Message Block (SMB)
## Windows CMD
#### Windows CMD - Net Use

```powershell
net use n: \\192.168.220.129\Finance
```

We can also provide username and password.

```powershell
net use n: \\192.168.220.129\Finance /user:plaintext Password123
```
#### Windows CMD - Find

| **Syntax** | **Description**                                                |
| ---------- | -------------------------------------------------------------- |
| `dir`      | Application                                                    |
| `n:`       | Directory or drive to search                                   |
| `/a-d`     | `/a` is the attribute and `-d` means not directories           |
| `/s`       | Displays files in a specified directory and all subdirectories |
| `/b`       | Uses bare format (no heading information or summary)           |
**Example:**

```powershell
dir n:\*cred* /s /b
```

#### Windows CMD - Findstr

```powershell
findstr /s /i cred n:\*.*
```

## Windows PowerShell

```powershell
Get-ChildItem \\192.168.220.129\Finance\
```

Instead of `net use` we can use `New-PSDrive`

```powershell
New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"
```

To use credentials in PowerShell, we have to create a `PSCredential object`
#### Windows PowerShell - PSCredential Object

```powershell
$username = 'plaintext'
$password = 'Password123'
$secpassword = ConvertTo-SecureString $password -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
```

#### Windows PowerShell - Get-ChildItem

We can use `Get-ChildItem` instead of `dir`

```powershell
N:

(Get-ChildItem -File -Recurse | Measure-Object).Count
```

We can use `-Include` to find specific items.

```powershell
Get-ChildItem -Recurse -Path N:\ -Include *cred* -File
```

We can also use `Select-String` to search for patterns with Regular Expressions
#### Windows PowerShell - Select-String

```powershell
Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List
```

## Linux

#### Linux - Mount

```sh
sudo mkdir /mnt/Finance
sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance
```

We can use a credential file:

```sh
mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```

#### Linux - Find

```sh
find /mnt/Finance -name *cred*
```

