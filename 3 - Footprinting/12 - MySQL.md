# Dangerous Settings

Many things can be misconfigured with MySQL. We can look in more detail at the [MySQL reference](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html) to determine which options can be made in the server configuration. The main options that are security-relevant are:

|**Settings**|**Description**|
|---|---|
|`user`|Sets which user the MySQL service will run as.|
|`password`|Sets the password for the MySQL user.|
|`admin_address`|The IP address on which to listen for TCP/IP connections on the administrative network interface.|
|`debug`|This variable indicates the current debugging settings|
|`sql_warnings`|This variable controls whether single-row INSERT statements produce an information string if warnings occur.|
|`secure_file_priv`|This variable is used to limit the effect of data import and export operations.|

|**Command**|**Description**|
|---|---|
|`mysql -u <user> -p<password> -h <IP address>`|Connect to the MySQL server. There should **not** be a space between the '-p' flag, and the password.|
|`show databases;`|Show all databases.|
|`use <database>;`|Select one of the existing databases.|
|`show tables;`|Show all available tables in the selected database.|
|`show columns from <table>;`|Show all columns in the selected table.|
|`select * from <table>;`|Show everything in the desired table.|
|`select * from <table> where <column> = "<string>";`|Search|
