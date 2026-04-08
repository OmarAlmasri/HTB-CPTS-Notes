# Linux Authentication Process

One of most commonly used methods for authentication systems is **PAM (Pluggable Authentication Modules)**. Modules responsible are `pam_unix.so` or `pam_unix2.so`, typically located in `/usr/lib/x86_64-linux-gnu/security/`. These modules manage user information, authentication, sessions, and password changes.
## Passwd File

```
htb-student:x:1000:1000:,,,:/home/htb-student:/bin/bash
```

|Field|Value|
|---|---|
|Username|`htb-student`|
|Password|`x`|
|User ID|`1000`|
|Group ID|`1000`|
|[GECOS](https://en.wikipedia.org/wiki/Gecos_field)|`,,,`|
|Home directory|`/home/htb-student`|
|Default shell|`/bin/bash`|
If somehow, the `/etc/passwd` file was writeable, we can edit it to remove the password field for the user, which will results in not asking for password when switching to that user account.

## Shadow File

```
htb-student:$y$j9T$3QSBB6CbHEu...SNIP...f8Ms:18955:0:99999:7:::
```

|Field|Value|
|---|---|
|Username|`htb-student`|
|Password|`$y$j9T$3QSBB6CbHEu...SNIP...f8Ms`|
|Last change|`18955`|
|Min age|`0`|
|Max age|`99999`|
|Warning period|`7`|
|Inactivity period|`-`|
|Expiration date|`-`|
|Reserved field|`-`|
The `Password` field also follows a particular format, from which we can extract additional information:

`$<id>$<salt>$<hashed>`

As we can see here, the hashed passwords are divided into three parts. The `ID` value specifies which cryptographic hash algorithm was used, typically one of the following:

|ID|Cryptographic Hash Algorithm|
|---|---|
|`1`|[MD5](https://en.wikipedia.org/wiki/MD5)|
|`2a`|[Blowfish](https://en.wikipedia.org/wiki/Blowfish_\(cipher\))|
|`5`|[SHA-256](https://en.wikipedia.org/wiki/SHA-2)|
|`6`|[SHA-512](https://en.wikipedia.org/wiki/SHA-2)|
|`sha1`|[SHA1crypt](https://en.wikipedia.org/wiki/SHA-1)|
|`y`|[Yescrypt](https://github.com/openwall/yescrypt)|
|`gy`|[Gost-yescrypt](https://www.openwall.com/lists/yescrypt/2019/06/30/1)|
|`7`|[Scrypt](https://en.wikipedia.org/wiki/Scrypt)|

```ad-info
Many Linux distributions, including Debian, now use `yescrypt` as the default hashing algorithm.
```
## Opasswd

The PAM library (`pam_unix.so`) can prevent users from reusing old passwords. These old passwords are stored in `/etc/security/opasswd` file. root privileges are required to read this file.

```sh
sudo cat /etc/security/opasswd 

cry0l1t3:1000:2:$1$HjFAfYTG$qNDkF0zJ3v8ylCOrKB0kt0,$1$kcUjWZJX$E9uMSmiQeRh4pAAgzuvkq1
```

# Cracking Linux Credentials

```sh
sudo cp /etc/passwd /tmp/passwd.bak
sudo cp /etc/shadow /tmp/shadow.bak
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

This "unshadowed" file can now be attacked with either JtR or hashcat.

```sh
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

This is the exact scenario that JtR's `single crack mode` was designed for.

# Credential Hunting in Linux

There are several sources that can provide us with credentials that we put in four categories. These include, but are not limited to:

- `Files` including configs, databases, notes, scripts, source code, cronjobs, and SSH keys
- `History` including logs, and command-line history
- `Memory` including cache, and in-memory processing
- `Key-rings` such as browser stored credentials
## Files

We should look for, find, and inspect several categories of files one by one. These categories are the following:

- Configuration files
- Databases
- Notes
- Scripts
- Cronjobs
- SSH keys

Config files are most interesting for us, they can be found with the following extensions:
`.config`, `.conf`, `.cnf`
#### Searching for configuration files

There are many methods to find these config files. As an example, we can use the following one-liner to search for these three file extensions:

```sh
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

Optionally, we can save the result in a text file and use it to examine the individual files one after the other. Another option is to run the scan directly for each file found with the specified file extension and output the contents. In this example, we search for three words (`user`, `password`, `pass`) in each file with the file extension `.cnf`.

```sh
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

#### Searching for databases

```sh
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```

#### Searching for notes

```sh
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```

#### Searching for scripts

Scripts are files that often contain highly sensitive information and processes.

```sh
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

#### Enumerating cronjobs

```sh
cat /etc/crontab

ls -la /etc/cron.*/
```

#### Enumerating history files

All history files provide crucial information about the current and past/historical course of processes.

**Example Files:** `.bash_history`, `.bashrc`, `.bash_profile`

#### Enumerating log files

The entirety of log files can be divided into four categories:

- Application logs
- Event logs
- Service logs
- System logs
Many different logs exist on the system. These can vary depending on the applications installed, but here are some of the most important ones:

|**File**|**Description**|
|---|---|
|`/var/log/messages`|Generic system activity logs.|
|`/var/log/syslog`|Generic system activity logs.|
|`/var/log/auth.log`|(Debian) All authentication related logs.|
|`/var/log/secure`|(RedHat/CentOS) All authentication related logs.|
|`/var/log/boot.log`|Booting information.|
|`/var/log/dmesg`|Hardware and drivers related information and logs.|
|`/var/log/kern.log`|Kernel related warnings, errors and logs.|
|`/var/log/faillog`|Failed login attempts.|
|`/var/log/cron`|Information related to cron jobs.|
|`/var/log/mail.log`|All mail server related logs.|
|`/var/log/httpd`|All Apache related logs.|
|`/var/log/mysqld.log`|All MySQL server related logs.|

**Example:**

```sh
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```

## Memory and cache

#### Mimipenguin

Many applications and processes work with credentials needed for authentication and store them either in memory or in files so that they can be reused.

Also some important credentials can be stored in browsers.

We can use the following tool [mimipenguin](https://github.com/huntergregal/mimipenguin) to make the process easy for us, *but this tool requires root permessions*

```sh
sudo python3 mimipenguin.py
```

#### LaZagne

We can also use `LaZagne` to extract credentials from the system.
The passwords and hashes we can obtain come from the following sources but are not limited to:

- Wifi
- Wpa_supplicant
- Libsecret
- Kwallet
- Chromium-based
- CLI
- Mozilla
- Thunderbird
- Git
- ENV variables
- Grub
- Fstab
- AWS
- Filezilla
- Gftp
- SSH
- Apache
- Shadow
- Docker
- Keepass
- Mimipy
- Sessions
- Keyrings

#### Browser credentials

Browsers stores encrypted user credentials on the system so they can be reused later. These often include the associated field names, URLs, and other valuable information.

**For Example:**


```sh
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq . 

{ "nextId": 2, "logins": [ { "id": 1, "hostname": "https://www.inlanefreight.com", "httpRealm": null, "formSubmitURL": "https://www.inlanefreight.com", "usernameField": "username", "passwordField": "password", "encryptedUsername": "MDoEEPgAAAA...SNIP...1liQiqBBAG/8/UpqwNlEPScm0uecyr", "encryptedPassword": "MEIEEPgAAAA...SNIP...FrESc4A3OOBBiyS2HR98xsmlrMCRcX2T9Pm14PMp3bpmE=", "guid": "{412629aa-4113-4ff9-befe-dd9b4ca388e2}", "encType": 1, "timeCreated": 1643373110869, "timeLastUsed": 1643373110869, "timePasswordChanged": 1643373110869, "timesUsed": 1 } ], "potentiallyVulnerablePasswords": [], "dismissedBreachAlertsByLoginGUID": {}, "version": 3 }
```

The tool [Firefox Decrypt](https://github.com/unode/firefox_decrypt) is excellent for decrypting these credentials.

![[FireFox Decrypt.png]]

Alternatively, we can use `LaZagne` to do the same process

```sh
python3 LaZagne.py browsers
```

