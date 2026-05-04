By default, SMTP servers accept connection requests on port `25`. However, newer SMTP servers also use other ports such as TCP port `587`.

- **ESMTP and SMTP-Auth**: modern servers use an extended SMTP (ESMTP) that supports authentication (SMTP-Auth). That means you must log in before you can send — this helps stop spam from anonymous users.
- **MSA vs MTA**: MSA handles _submission_ and validating the sender; MTA handles _transfer/delivery_.
- **Open Relay problem**: if the MSA/MTA allows anyone to forward mail without authentication, spammers can use it to send bulk mail. That’s called an **open relay** and it’s why correct configuration and SMTP-Auth are important.

| **Command**  | **Description**                                                                                  |
| ------------ | ------------------------------------------------------------------------------------------------ |
| `AUTH PLAIN` | AUTH is a service extension used to authenticate the client.                                     |
| `HELO`       | The client logs in with its computer name and thus starts the session.                           |
| `MAIL FROM`  | The client names the email sender.                                                               |
| `RCPT TO`    | The client names the email recipient.                                                            |
| `DATA`       | The client initiates the transmission of the email.                                              |
| `RSET`       | The client aborts the initiated transmission but keeps the connection between client and server. |
| `VRFY`       | The client checks if a mailbox is available for message transfer.                                |
| `EXPN`       | The client also checks if a mailbox is available for messaging with this command.                |
| `NOOP`       | The client requests a response from the server to prevent disconnection due to time-out.         |
| `QUIT`       | The client terminates the session.                                                               |

# Footprinting the Service

The default Nmap scripts include `smtp-commands`, which uses the `EHLO` command to list all possible commands that can be executed on the target SMTP server.

```shell
Ripcord88x@htb[/htb]$ sudo nmap 10.129.14.128 -sC -sV -p25

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-27 17:56 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00025s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: mail1.inlanefreight.htb, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.09 seconds
```

**Enumerating for available users**

```sh
┌─[us-academy-6]─[10.10.15.205]─[htb-ac-1041214@htb-twix4yiwpc]─[~]
└──╼ [★]$ smtp-user-enum -M VRFY -U wordlist.txt  -t 10.129.88.93 -m 60 -w 20
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... VRFY
Worker Processes ......... 60
Usernames file ........... wordlist.txt
Target count ............. 1
Username count ........... 101
Target TCP port .......... 25
Query timeout ............ 20 secs
Target domain ............ 

######## Scan started at Thu Oct 23 14:31:42 2025 #########
10.129.88.93: robin exists
######## Scan completed at Thu Oct 23 14:32:00 2025 #########
1 results.

101 queries in 18 seconds (5.6 queries / sec
```

