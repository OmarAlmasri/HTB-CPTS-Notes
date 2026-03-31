Because rainbow tables are such a powerful attack, `salting` is used. A `salt`, in cryptographic terms, is a random sequence of bytes added to a password before it is hashed. To maximize impact, salts should not be reused, e.g. for all passwords stored in one database.

# Introduction to John the Ripper

## Cracking Modes

### Single crack mode

Single crack mode uses the user’s own profile information as the dictionary.

Instead of throwing a massive, generic wordlist (like `rockyou.txt`) at the password hash, John the Ripper creates a tiny, custom wordlist tailored specifically for that exact user.

```sh
john --single passwd
```

## Identifying Hash Formats

```ad-resources
Review [JtR's sample hash documentation](https://openwall.info/wiki/john/sample-hashes), or [this list by PentestMonkey](https://pentestmonkey.net/cheat-sheet/john-the-ripper-hash-formats)

---

We can also use tools like this to identify different type of hashes [psypanda/hashID](https://github.com/psypanda/hashID)
```

|**Hash format**|**Example command**|**Description**|
|---|---|---|
|afs|`john --format=afs [...] <hash_file>`|AFS (Andrew File System) password hashes|
|bfegg|`john --format=bfegg [...] <hash_file>`|bfegg hashes used in Eggdrop IRC bots|
|bf|`john --format=bf [...] <hash_file>`|Blowfish-based crypt(3) hashes|
|bsdi|`john --format=bsdi [...] <hash_file>`|BSDi crypt(3) hashes|
|crypt(3)|`john --format=crypt [...] <hash_file>`|Traditional Unix crypt(3) hashes|
|des|`john --format=des [...] <hash_file>`|Traditional DES-based crypt(3) hashes|
|dmd5|`john --format=dmd5 [...] <hash_file>`|DMD5 (Dragonfly BSD MD5) password hashes|
|dominosec|`john --format=dominosec [...] <hash_file>`|IBM Lotus Domino 6/7 password hashes|
|EPiServer SID hashes|`john --format=episerver [...] <hash_file>`|EPiServer SID (Security Identifier) password hashes|
|hdaa|`john --format=hdaa [...] <hash_file>`|hdaa password hashes used in Openwall GNU/Linux|
|hmac-md5|`john --format=hmac-md5 [...] <hash_file>`|hmac-md5 password hashes|
|hmailserver|`john --format=hmailserver [...] <hash_file>`|hmailserver password hashes|
|ipb2|`john --format=ipb2 [...] <hash_file>`|Invision Power Board 2 password hashes|
|krb4|`john --format=krb4 [...] <hash_file>`|Kerberos 4 password hashes|
|krb5|`john --format=krb5 [...] <hash_file>`|Kerberos 5 password hashes|
|LM|`john --format=LM [...] <hash_file>`|LM (Lan Manager) password hashes|
|lotus5|`john --format=lotus5 [...] <hash_file>`|Lotus Notes/Domino 5 password hashes|
|mscash|`john --format=mscash [...] <hash_file>`|MS Cache password hashes|
|mscash2|`john --format=mscash2 [...] <hash_file>`|MS Cache v2 password hashes|
|mschapv2|`john --format=mschapv2 [...] <hash_file>`|MS CHAP v2 password hashes|
|mskrb5|`john --format=mskrb5 [...] <hash_file>`|MS Kerberos 5 password hashes|
|mssql05|`john --format=mssql05 [...] <hash_file>`|MS SQL 2005 password hashes|
|mssql|`john --format=mssql [...] <hash_file>`|MS SQL password hashes|
|mysql-fast|`john --format=mysql-fast [...] <hash_file>`|MySQL fast password hashes|
|mysql|`john --format=mysql [...] <hash_file>`|MySQL password hashes|
|mysql-sha1|`john --format=mysql-sha1 [...] <hash_file>`|MySQL SHA1 password hashes|
|NETLM|`john --format=netlm [...] <hash_file>`|NETLM (NT LAN Manager) password hashes|
|NETLMv2|`john --format=netlmv2 [...] <hash_file>`|NETLMv2 (NT LAN Manager version 2) password hashes|
|NETNTLM|`john --format=netntlm [...] <hash_file>`|NETNTLM (NT LAN Manager) password hashes|
|NETNTLMv2|`john --format=netntlmv2 [...] <hash_file>`|NETNTLMv2 (NT LAN Manager version 2) password hashes|
|NEThalfLM|`john --format=nethalflm [...] <hash_file>`|NEThalfLM (NT LAN Manager) password hashes|
|md5ns|`john --format=md5ns [...] <hash_file>`|md5ns (MD5 namespace) password hashes|
|nsldap|`john --format=nsldap [...] <hash_file>`|nsldap (OpenLDAP SHA) password hashes|
|ssha|`john --format=ssha [...] <hash_file>`|ssha (Salted SHA) password hashes|
|NT|`john --format=nt [...] <hash_file>`|NT (Windows NT) password hashes|
|openssha|`john --format=openssha [...] <hash_file>`|OPENSSH private key password hashes|
|oracle11|`john --format=oracle11 [...] <hash_file>`|Oracle 11 password hashes|
|oracle|`john --format=oracle [...] <hash_file>`|Oracle password hashes|
|pdf|`john --format=pdf [...] <hash_file>`|PDF (Portable Document Format) password hashes|
|phpass-md5|`john --format=phpass-md5 [...] <hash_file>`|PHPass-MD5 (Portable PHP password hashing framework) password hashes|
|phps|`john --format=phps [...] <hash_file>`|PHPS password hashes|
|pix-md5|`john --format=pix-md5 [...] <hash_file>`|Cisco PIX MD5 password hashes|
|po|`john --format=po [...] <hash_file>`|Po (Sybase SQL Anywhere) password hashes|
|rar|`john --format=rar [...] <hash_file>`|RAR (WinRAR) password hashes|
|raw-md4|`john --format=raw-md4 [...] <hash_file>`|Raw MD4 password hashes|
|raw-md5|`john --format=raw-md5 [...] <hash_file>`|Raw MD5 password hashes|
|raw-md5-unicode|`john --format=raw-md5-unicode [...] <hash_file>`|Raw MD5 Unicode password hashes|
|raw-sha1|`john --format=raw-sha1 [...] <hash_file>`|Raw SHA1 password hashes|
|raw-sha224|`john --format=raw-sha224 [...] <hash_file>`|Raw SHA224 password hashes|
|raw-sha256|`john --format=raw-sha256 [...] <hash_file>`|Raw SHA256 password hashes|
|raw-sha384|`john --format=raw-sha384 [...] <hash_file>`|Raw SHA384 password hashes|
|raw-sha512|`john --format=raw-sha512 [...] <hash_file>`|Raw SHA512 password hashes|
|salted-sha|`john --format=salted-sha [...] <hash_file>`|Salted SHA password hashes|
|sapb|`john --format=sapb [...] <hash_file>`|SAP CODVN B (BCODE) password hashes|
|sapg|`john --format=sapg [...] <hash_file>`|SAP CODVN G (PASSCODE) password hashes|
|sha1-gen|`john --format=sha1-gen [...] <hash_file>`|Generic SHA1 password hashes|
|skey|`john --format=skey [...] <hash_file>`|S/Key (One-time password) hashes|
|ssh|`john --format=ssh [...] <hash_file>`|SSH (Secure Shell) password hashes|
|sybasease|`john --format=sybasease [...] <hash_file>`|Sybase ASE password hashes|
|xsha|`john --format=xsha [...] <hash_file>`|xsha (Extended SHA) password hashes|
|zip|`john --format=zip [...] <hash_file>`|ZIP (WinZip) password hashes|
## Cracking Files

Some of the tools included with JtR are:

|**Tool**|**Description**|
|---|---|
|`pdf2john`|Converts PDF documents for John|
|`ssh2john`|Converts SSH private keys for John|
|`mscash2john`|Converts MS Cash hashes for John|
|`keychain2john`|Converts OS X keychain files for John|
|`rar2john`|Converts RAR archives for John|
|`pfx2john`|Converts PKCS#12 files for John|
|`truecrypt_volume2john`|Converts TrueCrypt volumes for John|
|`keepass2john`|Converts KeePass databases for John|
|`vncpcap2john`|Converts VNC PCAP files for John|
|`putty2john`|Converts PuTTY private keys for John|
|`zip2john`|Converts ZIP archives for John|
|`hccap2john`|Converts WPA/WPA2 handshake captures for John|
|`office2john`|Converts MS Office documents for John|
|`wpa2john`|Converts WPA/WPA2 handshakes for John|
|...SNIP...|...SNIP...|
An even larger collection can be found on the attacker host:

```sh
locate *2john*
```

# Introduction to Hashcat

```sh
hashcat -a 0 -m 0 <hashes> [wordlist, rule, mask, ...]
```

- `-a`: Attack mode
- `-m`: Hash type

## Hash Type Identification

```sh
hashid -m  e3e3ec5831ad5e7288241960e5d4fdb8
Analyzing 'e3e3ec5831ad5e7288241960e5d4fdb8'
[+] MD2 
[+] MD5 [Hashcat Mode: 0]
[+] MD4 [Hashcat Mode: 900]
[+] Double MD5 [Hashcat Mode: 2600]
[+] LM [Hashcat Mode: 3000]
[+] RIPEMD-128 
[+] Haval-128 
[+] Tiger-128 
[+] Skein-256(128) 
[+] Skein-512(128) 
[+] Lotus Notes/Domino 5 [Hashcat Mode: 8600]
[+] Skype [Hashcat Mode: 23]
[+] Snefru-128 
[+] NTLM [Hashcat Mode: 1000]
[+] Domain Cached Credentials [Hashcat Mode: 1100]
[+] Domain Cached Credentials 2 [Hashcat Mode: 2100]
[+] DNSSEC(NSEC3) [Hashcat Mode: 8300]
[+] RAdmin v2.x [Hashcat Mode: 9900]
```

## Rules

Most of times a password list like `rockyou.txt` isn't enough to crack a password, so we have to apply rules which apply modification to the specified wordlist. Modifications using rules can be like the common or standard password modifications like appending numbers, replacing characters with their `leet` equivalent. To perform this attack we specify `-r <ruleset>` .

```sh
hashcat -a 0 -m 0 1b0556a75770563578569ae21392630c /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

## Mass Attack

[Mask attack](https://hashcat.net/wiki/doku.php?id=mask_attack) (`-a 3`) is a type of brute-force attack in which the keyspace is explicitly defined by the user. 
A mask is defined by combining a sequence of symbols, each representing a built-in or custom character set. Hashcat includes several built-in character sets:

|Symbol|Charset|
|---|---|
|?l|abcdefghijklmnopqrstuvwxyz|
|?u|ABCDEFGHIJKLMNOPQRSTUVWXYZ|
|?d|0123456789|
|?h|0123456789abcdef|
|?H|0123456789ABCDEF|
|?s|«space»!"#$%&'()*+,-./:;<=>?@[]^_`{|
|?a|?l?u?d?s|
|?b|0x00 - 0xff|
Custom charsets can be defined with the `-1`, `-2`, `-3`, and `-4` arguments, then referred to with `?1`, `?2`, `?3`, and `?4`.

**Example Usage:**

We want a password which starts with *uppercase letter*, continue with *four lowercase letters*, *a digit*, and then a *symbol*

The resulting hashcat mask will be the following: `?u?l?l?l?l?d?s`.

```sh
hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
```

# Writing Custom Wordlists and Rules

## Hashcat Mutates

| **Function** | **Description**                                  |
| ------------ | ------------------------------------------------ |
| `:`          | Do nothing                                       |
| `l`          | Lowercase all letters                            |
| `u`          | Uppercase all letters                            |
| `c`          | Capitalize the first letter and lowercase others |
| `sXY`        | Replace all instances of X with Y                |
| `$!`         | Add the exclamation character at the end         |

```ad-resources
Full list can be reviewed from [rule_based_attack [hashcat wiki]](https://hashcat.net/wiki/doku.php?id=rule_based_attack) 
```

 **Custom Rule Example:**

```sh
Ripcord88x@htb[/htb]$ cat custom.rule

:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
```

### Hashcat Rules Reference Table

| **Symbol** | **Function**                   |
| ---------- | ------------------------------ |
| `c`        | Capitalize first char          |
| `u`        | Uppercase all                  |
| `l`        | Lowercase all                  |
| `t`        | Toggle all case                |
| `r`        | Reverse the word               |
| `$X`       | Append character X             |
| `^X`       | Prepend character X            |
| `sXY`      | Substitute X with Y            |
| `D3`       | Delete character at position 3 |
| `i3X`      | Insert X at position 3         |

## Generating Wordlists using CeWL

**Usage Example:**

```sh
cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
```

- `-d`: depth to spider
- `-m`: minimum length of the word
- `--lowercase`: the storage of the found words in lowercase
- `-w`: where we want to store the results

```ad-resources
A tool `cupp` can be used for interactive password list generation based on OSINT data.

**Usage Example:**
`cupp -i`
```

# Cracking Protected Files

## Hunting for Encrypted Files

An example bash one-liner that can be used to find important file extensions in the file-system

```sh
for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

## Hunting for SSH keys

We can also hunt for accessible SSH keys

```sh
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null
```

Some SSH keys are encrypted with a passphrase. One way to tell whether an SSH key is encrypted or not, is to try reading the key with `ssh-keygen`.

```sh
ssh-keygen -yf ~/.ssh/id_ed25519

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIpNefJd834VkD5iq+22Zh59Gzmmtzo6rAffCx2UtaS6
```

Attempting to read a protected SSH key will prompt the user for a passphrase:

```sh
ssh-keygen -yf ~/.ssh/id_rsa 

Enter passphrase for "/home/jsmith/.ssh/id_rsa":
```

We can crack encrypted SSH keys using `ssh2john` tool

**Usage Example:**

```sh
ssh2john.py SSH.private > ssh.hash

john --wordlist=rockyou.txt ssh.hash
```

## Cracking password-protected documents

`office2john.py` has the ability to extract hashes from all protected common Office documents formats. Then, these hashes can be supplied to `john` for offline cracking

```sh
office2john.py Protected.docx > protected-docx.hash

john --wordlist=rockyou.txt protected-docx.hash
```

Same thing can be done for protected PDF documents using `pdf2john.py`.

```sh
pdf2john.py PDF.pdf > pdf.hash
john --wordlist=rockyou.txt pdf.hash
```

# Cracking Protected Archives

We can get a full list of archive file types from **FileInfo** website, and we can extract them as text instead of writing them manually:

```sh
curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt
```

## Cracking ZIP Files

```sh
zip2john.py file.zip > file.hash
john --wordlist=rockyou.txt file.hash
```

## Cracking OpenSSL Encrypted GZIP Files

We can't always immediately know if a file using an extension that doesn't natively support passwords is protected (e.g. `openssl` can be used to encrypt `GZIP` files). To determine the actual format of the file we can use `file` command.

```sh
file GZIP.gzip

# Output:
GZIP.gzip: openssl enc'd data with salted password
```

We can use the following one-liner to crack the encrypted file.
*Note that this approach may produce several GZIP errors that can be safely ignored*

```sh
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```

Once the `for` finishes we can check the current directory for newly extracted files.
## Cracking BitLocker-encrypted drives

We can use `bitlocker2john` to extract four hashes: the first two correspond to the BitLocker password, while the latter two represent the recovery key. Because the recovery key is very long and randomly generated, it is not a good idea to try to guess it. Therefore, we'll focus on the password using the first hash.

**Example Usage:**

```sh
bitlocker2john -i Backup.vhd > backup.hashes 

grep "bitlocker\$0" backup.hashes > backup.hash 

cat backup.hash 

$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f
```

After that we can try to crack the hash using `hashcat`

```sh
hashcat -a 0 -m 22100 '$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f' /usr/share/wordlists/rockyou.txt
```

#### Mounting BitLocker-encrypted drives in Windows

Double-Click on the `.vhd` 
#### Mounting BitLocker-encrypted drives in Linux (or macOS)

```sh
sudo apt-get install dislocker
sudo mkdir -p /media/bitlocker 
sudo mkdir -p /media/bitlockermount
sudo losetup -f -P Backup.vhd 
sudo losetup --all
sudo dislocker /dev/loop0p2 -u "PASSWORD" -- /media/bitlocker 
sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
```

If everything was done correctly, we can browse the files:

```sh
cd /media/bitlockermount/
ls -la
```

Once we have analyzed the files on the mounted drive, we can unmount it using the following commands:

```sh
sudo umount /media/bitlockermount
sudo umount /media/bitlocker
```

