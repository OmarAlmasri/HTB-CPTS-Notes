An SMB server can provide arbitrary parts of its local file system as shares. Therefore the hierarchy visible to a client is partially independent of the structure on the server. Access rights are defined by `Access Control Lists` (`ACL`). They can be controlled in a fine-grained manner based on attributes such as `execute`, `read`, and `full access` for individual users or user groups. The ACLs are defined based on the shares and therefore do not correspond to the rights assigned locally on the server.

# Samba

As mentioned earlier, there is an alternative implementation of the SMB server called Samba, which is developed for Unix-based operating systems. Samba implements the Common Internet File System (`CIFS`) network protocol. [CIFS](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-cifs/934c2faa-54af-4526-ac74-6a24d126724e) is a dialect of SMB, meaning it is a specific implementation of the SMB protocol originally created by Microsoft. This allows Samba to communicate effectively with newer Windows systems. Therefore, it is often referred to as SMB/CIFS.

However, `CIFS` is considered a specific version of the SMB protocol, primarily aligning with `SMB version 1`. When SMB commands are transmitted over Samba to an older NetBIOS service, connections typically occur over TCP ports `137`, `138`, and `139`. In contrast, CIFS operates over TCP port `445` exclusively. There are several versions of SMB, including newer versions like `SMB 2` and `SMB 3`, which offer improvements and are preferred in modern infrastructures, while older versions like `SMB 1` (`CIFS`) are considered outdated but may still be used in specific environments.

|**SMB Version**|**Supported**|**Features**|
|---|---|---|
|CIFS|Windows NT 4.0|Communication via NetBIOS interface|
|SMB 1.0|Windows 2000|Direct connection via TCP|
|SMB 2.0|Windows Vista, Windows Server 2008|Performance upgrades, improved message signing, caching feature|
|SMB 2.1|Windows 7, Windows Server 2008 R2|Locking mechanisms|
|SMB 3.0|Windows 8, Windows Server 2012|Multichannel connections, end-to-end encryption, remote storage access|
|SMB 3.0.2|Windows 8.1, Windows Server 2012 R2||
|SMB 3.1.1|Windows 10, Windows Server 2016|Integrity checking, AES-128 encryption|
We know that Samba is suitable for both Linux and Windows systems. In a network, each host participates in the same `workgroup`. A workgroup is a group name that identifies an arbitrary collection of computers and their resources on an SMB network. There can be multiple workgroups on the network at any given time. IBM developed an `application programming interface` (`API`) for networking computers called the `Network Basic Input/Output System` (`NetBIOS`). The NetBIOS API provided a blueprint for an application to connect and share data with other computers. In a NetBIOS environment, when a machine goes online, it needs a name, which is done through the so-called `name registration` procedure. Either each host reserves its hostname on the network, or the [NetBIOS Name Server](https://networkencyclopedia.com/netbios-name-server-nbns/) (`NBNS`) is used for this purpose. It also has been enhanced to [Windows Internet Name Service](https://networkencyclopedia.com/windows-internet-name-service-wins/) (`WINS`).

# Default Configuration

`/etc/samba/smb.conf`

|**Setting**|**Description**|
|---|---|
|`[sharename]`|The name of the network share.|
|`workgroup = WORKGROUP/DOMAIN`|Workgroup that will appear when clients query.|
|`path = /path/here/`|The directory to which user is to be given access.|
|`server string = STRING`|The string that will show up when a connection is initiated.|
|`unix password sync = yes`|Synchronize the UNIX password with the SMB password?|
|`usershare allow guests = yes`|Allow non-authenticated users to access defined share?|
|`map to guest = bad user`|What to do when a user login request doesn't match a valid UNIX user?|
|`browseable = yes`|Should this share be shown in the list of available shares?|
|`guest ok = yes`|Allow connecting to the service without using a password?|
|`read only = yes`|Allow users to read files only?|
|`create mask = 0700`|What permissions need to be set for newly created files?|

## Dangerous Settings

| **Setting**                 | **Description**                                                     |
| --------------------------- | ------------------------------------------------------------------- |
| `browseable = yes`          | Allow listing available shares in the current share?                |
| `read only = no`            | Forbid the creation and modification of files?                      |
| `writable = yes`            | Allow users to create and modify files?                             |
| `guest ok = yes`            | Allow connecting to the service without using a password?           |
| `enable privileges = yes`   | Honor privileges assigned to specific SID?                          |
| `create mask = 0777`        | What permissions must be assigned to the newly created files?       |
| `directory mask = 0777`     | What permissions must be assigned to the newly created directories? |
| `logon script = script.sh`  | What script needs to be executed on the user's login?               |
| `magic script = script.sh`  | Which script should be executed when the script gets closed?        |
| `magic output = script.out` | Where the output of the magic script needs to be stored?            |

# Footprinting the Service

We can use `rpcclient` for our enumeration

```shell
Ripcord88x@htb[/htb]$ rpcclient -U "" 10.129.14.128

Enter WORKGROUP\'s password:
rpcclient $> 
```

|**Query**|**Description**|
|---|---|
|`srvinfo`|Server information.|
|`enumdomains`|Enumerate all domains that are deployed in the network.|
|`querydominfo`|Provides domain, server, and user information of deployed domains.|
|`netshareenumall`|Enumerates all available shares.|
|`netsharegetinfo <share>`|Provides information about a specific share.|
|`enumdomusers`|Enumerates all domain users.|
|`queryuser <RID>`|Provides information about a specific user.|
## Brute Forcing User RIDs

```shell
Ripcord88x@htb[/htb]$ for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done

        User Name   :   sambauser
        user_rid :      0x1f5
        group_rid:      0x201
		
        User Name   :   mrb3n
        user_rid :      0x3e8
        group_rid:      0x201
		
        User Name   :   cry0l1t3
        user_rid :      0x3e9
        group_rid:      0x201
```

Alternative way is using `samrdump.py` from **Impacket**

```shell
Ripcord88x@htb[/htb]$ samrdump.py 10.129.14.128

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Retrieving endpoint list from 10.129.14.128
Found domain(s):
 . DEVSMB
 . Builtin
[*] Looking up users in domain DEVSMB
Found user: mrb3n, uid = 1000
Found user: cry0l1t3, uid = 1001
mrb3n (1000)/FullName: 
mrb3n (1000)/UserComment: 
mrb3n (1000)/PrimaryGroupId: 513
mrb3n (1000)/BadPasswordCount: 0
mrb3n (1000)/LogonCount: 0
mrb3n (1000)/PasswordLastSet: 2021-09-22 17:47:59
mrb3n (1000)/PasswordDoesNotExpire: False
mrb3n (1000)/AccountIsDisabled: False
mrb3n (1000)/ScriptPath: 
cry0l1t3 (1001)/FullName: cry0l1t3
cry0l1t3 (1001)/UserComment: 
cry0l1t3 (1001)/PrimaryGroupId: 513
cry0l1t3 (1001)/BadPasswordCount: 0
cry0l1t3 (1001)/LogonCount: 0
cry0l1t3 (1001)/PasswordLastSet: 2021-09-22 17:50:56
cry0l1t3 (1001)/PasswordDoesNotExpire: False
cry0l1t3 (1001)/AccountIsDisabled: False
cry0l1t3 (1001)/ScriptPath: 
[*] Received 2 entries.
```

The information we have already obtained with `rpcclient` can also be obtained using other tools. For example, the [SMBMap](https://github.com/ShawnDEvans/smbmap) and [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) tools are also widely used and helpful for the enumeration of SMB services.

### SMBmap

```shell
Ripcord88x@htb[/htb]$ smbmap -H 10.129.14.128

[+] Finding open SMB ports....
[+] User SMB session established on 10.129.14.128...
[+] IP: 10.129.14.128:445       Name: 10.129.14.128                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        home                                                    NO ACCESS       INFREIGHT Samba
        dev                                                     NO ACCESS       DEVenv
        notes                                                   NO ACCESS       CheckIT
        IPC$                                                    NO ACCESS       IPC Service (DEVSM)
```

### CrackMapExec

```shell
Ripcord88x@htb[/htb]$ crackmapexec smb 10.129.14.128 --shares -u '' -p ''

SMB         10.129.14.128   445    DEVSMB           [*] Windows 6.1 Build 0 (name:DEVSMB) (domain:) (signing:False) (SMBv1:False)
SMB         10.129.14.128   445    DEVSMB           [+] \: 
SMB         10.129.14.128   445    DEVSMB           [+] Enumerated shares
SMB         10.129.14.128   445    DEVSMB           Share           Permissions     Remark
SMB         10.129.14.128   445    DEVSMB           -----           -----------     ------
SMB         10.129.14.128   445    DEVSMB           print$                          Printer Drivers
SMB         10.129.14.128   445    DEVSMB           home                            INFREIGHT Samba
SMB         10.129.14.128   445    DEVSMB           dev                             DEVenv
SMB         10.129.14.128   445    DEVSMB           notes           READ,WRITE      CheckIT
SMB         10.129.14.128   445    DEVSMB           IPC$                            IPC Service (DEVSM)
```

### enum4linux-ng

```shell
Ripcord88x@htb[/htb]$ ./enum4linux-ng.py 10.129.14.128 -A

ENUM4LINUX - next generation

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.129.14.128
[*] Username ......... ''
[*] Random Username .. 'juzgtcsu'
[*] Password ......... ''
[*] Timeout .......... 5 second(s)

 =====================================
|    Service Scan on 10.129.14.128    |
 =====================================
[*] Checking LDAP
[-] Could not connect to LDAP on 389/tcp: connection refused
[*] Checking LDAPS
[-] Could not connect to LDAPS on 636/tcp: connection refused
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

----SNIP----
```

We need to use more than two tools for enumeration. Because it can happen that due to the programming of the tools, we get different information that we have to check manually. Therefore, we should never rely only on automated tools where we do not know precisely how they were written.