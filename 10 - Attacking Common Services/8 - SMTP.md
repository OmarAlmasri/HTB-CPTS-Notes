# Attacking Email Services

- **SMTP** is a protocol used to deliver emails from clients to servers and from servers to other servers.
- When we download emails to our email application, it will connect to a **POP3** or **IMAP4** server on the internet.
- By default **POP3** removes downloaded emails from the email server. However, we can configure a **POP3** client to keep copies of the downloaded messages on the server.
- By default, **IMAP4** clients doesn't remove downloaded messages from the email server.

## Enumeration

We can use the `Mail eXchanger` (`MX`) DNS record to identify a mail server.

Tools like `host`, `dig`, or online websites like [MXToolbox](https://mxtoolbox.com/) can be used to query information about the MX records

```sh
host -t MX hackthebox.eu

dig mx plaintext.do | grep "MX" | grep -v ";"

host -t A mail1.inlanefreight.htb.
```

If we are attacking a custom mail server implementation, we can enumerate for the following ports:

| **Port**  | **Service**                                                                |
|:--------- | -------------------------------------------------------------------------- |
| `TCP/25`  | SMTP Unencrypted                                                           |
| `TCP/143` | IMAP4 Unencrypted                                                          |
| `TCP/110` | POP3 Unencrypted                                                           |
| `TCP/465` | SMTP Encrypted                                                             |
| `TCP/587` | SMTP Encrypted/[STARTTLS](https://en.wikipedia.org/wiki/Opportunistic_TLS) |
| `TCP/993` | IMAP4 Encrypted                                                            |
| `TCP/995` | POP3 Encrypted                                                             |
We can use `nmap` with `-sC` option to enumerate these ports:

```sh
sudo nmap -Pn -sV -sC -p25,143,110,465,587,993,995 $IP
```

## Misconfigurations

A misconfiguration can happen when the SMTP service allows anonymous authentication or support protocols that can be used to enumerate valid usernames.
#### Authentication

Commands like `VRFY`, `EXPN`, `RCPT TO` can be used to enumerate valid usernames, if done successfully we can attempt to do password spraying or brute-forcing.
#### VRFY Command

`VRFY` instructs the receiving SMTP server to check the validity of a particular email username.

![[VRFY Command.png]]
#### EXPN 

`EXPN` is similar to `VRFY`, except that when used with a distribution list, it will list all users on that list.

![[EXPN Command.png]]
#### RCPT TO Comand

`RCPT TO` identifies the recipient of the email message. The command can be repeated multiple times to send the message to multiple recipients

![[RCPT TO Command.png]]

---

We can also use `POP3` protocol to enumerate users depending on the service implementation.
#### USER Command

![[USER Command.png]]

---

To Automate the operation, we can use a tool like [smtp-user-enum](https://github.com/pentestmonkey/smtp-user-enum).

![[SMTP User Enum.png]]

## Cloud Enumeration

[O365spray](https://github.com/0xZDH/o365spray) is a username enumeration and password spraying tool aimed at Microsoft Office 365.
#### O365 Spray

![[O365 Spray.png]]

Attempt to identify usernames

```sh
python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz
```

## Password Attacks

