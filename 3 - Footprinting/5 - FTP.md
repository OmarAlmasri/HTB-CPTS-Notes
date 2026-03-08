# vsFTPd Config File

|**Setting**|**Description**|
|---|---|
|`listen=NO`|Run from inetd or as a standalone daemon?|
|`listen_ipv6=YES`|Listen on IPv6 ?|
|`anonymous_enable=NO`|Enable Anonymous access?|
|`local_enable=YES`|Allow local users to login?|
|`dirmessage_enable=YES`|Display active directory messages when users go into certain directories?|
|`use_localtime=YES`|Use local time?|
|`xferlog_enable=YES`|Activate logging of uploads/downloads?|
|`connect_from_port_20=YES`|Connect from port 20?|
|`secure_chroot_dir=/var/run/vsftpd/empty`|Name of an empty directory|
|`pam_service_name=vsftpd`|This string is the name of the PAM service vsftpd will use.|
|`rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem`|The last three options specify the location of the RSA certificate to use for SSL encrypted connections.|
|`rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key`||
|`ssl_enable=NO`||

In addition, there is a file called `/etc/ftpusers` that we also need to pay attention to, as this file is used to deny certain users access to the FTP service. In the following example, the users `guest`, `john`, and `kevin` are not permitted to log in to the FTP service, even if they exist on the Linux system.

```shell
Ripcord88x@htb[/htb]$ cat /etc/ftpusers
guest
john
kevin
```

## Dangerous Settings

| **Setting**                    | **Description**                                                                    |
| ------------------------------ | ---------------------------------------------------------------------------------- |
| `anonymous_enable=YES`         | Allowing anonymous login?                                                          |
| `anon_upload_enable=YES`       | Allowing anonymous to upload files?                                                |
| `anon_mkdir_write_enable=YES`  | Allowing anonymous to create new directories?                                      |
| `no_anon_password=YES`         | Do not ask anonymous for password?                                                 |
| `anon_root=/home/username/ftp` | Directory for anonymous.                                                           |
| `write_enable=YES`             | Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE? |
## vsFTPd Detailed Output

|**Setting**|**Description**|
|---|---|
|`dirmessage_enable=YES`|Show a message when they first enter a new directory?|
|`chown_uploads=YES`|Change ownership of anonymously uploaded files?|
|`chown_username=username`|User who is given ownership of anonymously uploaded files.|
|`local_enable=YES`|Enable local users to login?|
|`chroot_local_user=YES`|Place local users into their home directory?|
|`chroot_list_enable=YES`|Use a list of local users that will be placed in their home directory?|

|**Setting**|**Description**|
|---|---|
|`hide_ids=YES`|All user and group information in directory listings will be displayed as "ftp".|
|`ls_recurse_enable=YES`|Allows the use of recurse listings.|

# Download All Available File At Once

```shell
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

# Service Interaction

```sh
nc -nv 10.129.14.136 21
nc 10.129.14.136 21

# if SSL
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

