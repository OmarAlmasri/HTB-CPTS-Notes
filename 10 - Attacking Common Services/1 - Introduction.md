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

# Other Services
## Email

We can use `Evolution` on Linux
## Databases

### MSSQL

To interact with MSSQL, we can use  [sqsh](https://en.wikipedia.org/wiki/Sqsh) or [sqlcmd](https://docs.microsoft.com/en-us/sql/tools/sqlcmd-utility) if you're using windows
#### Linux - SQSH

```sh
sqsh -S 10.129.20.13 -U username -P Password123
```
#### Windows - SQLCMD

```powershell
sqlcmd -S 10.129.20.13 -U username -P Password123
```

---
### MySQL

Same command on Linux or Windows

```powershell
mysql -u username -pPassword123 -h 10.129.20.13
```

# Tools to Interact with Common Services

| **SMB**                                                                                  | **FTP**                                     | **Email**                                          | **Databases**                                                                                                                |
| ---------------------------------------------------------------------------------------- | ------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)          | [ftp](https://linux.die.net/man/1/ftp)      | [Thunderbird](https://www.thunderbird.net/en-US/)  | [mssql-cli](https://github.com/dbcli/mssql-cli)                                                                              |
| [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)                              | [lftp](https://lftp.yar.ru/)                | [Claws](https://www.claws-mail.org/)               | [mycli](https://github.com/dbcli/mycli)                                                                                      |
| [SMBMap](https://github.com/ShawnDEvans/smbmap)                                          | [ncftp](https://www.ncftp.com/)             | [Geary](https://wiki.gnome.org/Apps/Geary)         | [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py)                             |
| [Impacket](https://github.com/SecureAuthCorp/impacket)                                   | [filezilla](https://filezilla-project.org/) | [MailSpring](https://getmailspring.com/)           | [dbeaver](https://github.com/dbeaver/dbeaver)                                                                                |
| [psexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py)   | [crossftp](http://www.crossftp.com/)        | [mutt](http://www.mutt.org/)                       | [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)                                                                |
| [smbexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py) |                                             | [mailutils](https://mailutils.org/)                | [SQL Server Management Studio or SSMS](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms) |
|                                                                                          |                                             | [sendEmail](https://github.com/mogaal/sendemail)   |                                                                                                                              |
|                                                                                          |                                             | [swaks](http://www.jetmore.org/john/code/swaks/)   |                                                                                                                              |
|                                                                                          |                                             | [sendmail](https://en.wikipedia.org/wiki/Sendmail) |                                                                                                                              |