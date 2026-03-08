| DNS Concept                 | Description                                                                      | Example                                                                                                                                 |
| --------------------------- | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `Domain Name`               | A human-readable label for a website or other internet resource.                 | `www.example.com`                                                                                                                       |
| `IP Address`                | A unique numerical identifier assigned to each device connected to the internet. | `192.0.2.1`                                                                                                                             |
| `DNS Resolver`              | A server that translates domain names into IP addresses.                         | Your ISP's DNS server or public resolvers like Google DNS (`8.8.8.8`)                                                                   |
| `Root Name Server`          | The top-level servers in the DNS hierarchy.                                      | There are 13 root servers worldwide, named A-M: `a.root-servers.net`                                                                    |
| `TLD Name Server`           | Servers responsible for specific top-level domains (e.g., .com, .org).           | [Verisign](https://en.wikipedia.org/wiki/Verisign) for `.com`, [PIR](https://en.wikipedia.org/wiki/Public_Interest_Registry) for `.org` |
| `Authoritative Name Server` | The server that holds the actual IP address for a domain.                        | Often managed by hosting providers or domain registrars.                                                                                |
| `DNS Record Types`          | Different types of information stored in DNS.                                    | A, AAAA, CNAME, MX, NS, TXT, etc.                                                                                                       |

|Record Type|Full Name|Description|Zone File Example|
|---|---|---|---|
|`A`|Address Record|Maps a hostname to its IPv4 address.|`www.example.com.` IN A `192.0.2.1`|
|`AAAA`|IPv6 Address Record|Maps a hostname to its IPv6 address.|`www.example.com.` IN AAAA `2001:db8:85a3::8a2e:370:7334`|
|`CNAME`|Canonical Name Record|Creates an alias for a hostname, pointing it to another hostname.|`blog.example.com.` IN CNAME `webserver.example.net.`|
|`MX`|Mail Exchange Record|Specifies the mail server(s) responsible for handling email for the domain.|`example.com.` IN MX 10 `mail.example.com.`|
|`NS`|Name Server Record|Delegates a DNS zone to a specific authoritative name server.|`example.com.` IN NS `ns1.example.com.`|
|`TXT`|Text Record|Stores arbitrary text information, often used for domain verification or security policies.|`example.com.` IN TXT `"v=spf1 mx -all"` (SPF record)|
|`SOA`|Start of Authority Record|Specifies administrative information about a DNS zone, including the primary name server, responsible person's email, and other parameters.|`example.com.` IN SOA `ns1.example.com. admin.example.com. 2024060301 10800 3600 604800 86400`|
|`SRV`|Service Record|Defines the hostname and port number for specific services.|`_sip._udp.example.com.` IN SRV 10 5 5060 `sipserver.example.com.`|
|`PTR`|Pointer Record|Used for reverse DNS lookups, mapping an IP address to a hostname.|`1.2.0.192.in-addr.arpa.` IN PTR `www.example.com.`|

# Digging DNS

|Tool|Key Features|Use Cases|
|---|---|---|
|`dig`|Versatile DNS lookup tool that supports various query types (A, MX, NS, TXT, etc.) and detailed output.|Manual DNS queries, zone transfers (if allowed), troubleshooting DNS issues, and in-depth analysis of DNS records.|
|`nslookup`|Simpler DNS lookup tool, primarily for A, AAAA, and MX records.|Basic DNS queries, quick checks of domain resolution and mail server records.|
|`host`|Streamlined DNS lookup tool with concise output.|Quick checks of A, AAAA, and MX records.|
|`dnsenum`|Automated DNS enumeration tool, dictionary attacks, brute-forcing, zone transfers (if allowed).|Discovering subdomains and gathering DNS information efficiently.|
|`fierce`|DNS reconnaissance and subdomain enumeration tool with recursive search and wildcard detection.|User-friendly interface for DNS reconnaissance, identifying subdomains and potential targets.|
|`dnsrecon`|Combines multiple DNS reconnaissance techniques and supports various output formats.|Comprehensive DNS enumeration, identifying subdomains, and gathering DNS records for further analysis.|
|`theHarvester`|OSINT tool that gathers information from various sources, including DNS records (email addresses).|Collecting email addresses, employee information, and other data associated with a domain from multiple sources.|
|`Online DNS Lookup Services`|User-friendly interfaces for performing DNS lookups.|Quick and easy DNS lookups, convenient when command-line tools are not available, checking for domain availability or basic information|

## Common dig Commands

|Command|Description|
|---|---|
|`dig domain.com`|Performs a default A record lookup for the domain.|
|`dig domain.com A`|Retrieves the IPv4 address (A record) associated with the domain.|
|`dig domain.com AAAA`|Retrieves the IPv6 address (AAAA record) associated with the domain.|
|`dig domain.com MX`|Finds the mail servers (MX records) responsible for the domain.|
|`dig domain.com NS`|Identifies the authoritative name servers for the domain.|
|`dig domain.com TXT`|Retrieves any TXT records associated with the domain.|
|`dig domain.com CNAME`|Retrieves the canonical name (CNAME) record for the domain.|
|`dig domain.com SOA`|Retrieves the start of authority (SOA) record for the domain.|
|`dig @1.1.1.1 domain.com`|Specifies a specific name server to query; in this case 1.1.1.1|
|`dig +trace domain.com`|Shows the full path of DNS resolution.|
|`dig -x 192.168.1.1`|Performs a reverse lookup on the IP address 192.168.1.1 to find the associated host name. You may need to specify a name server.|
|`dig +short domain.com`|Provides a short, concise answer to the query.|
|`dig +noall +answer domain.com`|Displays only the answer section of the query output.|
|`dig domain.com ANY`|Retrieves all available DNS records for the domain (Note: Many DNS servers ignore `ANY` queries to reduce load and prevent abuse, as per [RFC 8482](https://datatracker.ietf.org/doc/html/rfc8482)).|

# Subdomains

```shell
 ____                                      __     __      __
/\  _`\   __                              /\ \  /'_ `\  /'_ `\
\ \ \L\ \/\_\  _____    ___    ___   _ __ \_\ \/\ \L\ \/\ \L\ \  __  _
 \ \ ,  /\/\ \/\ '__`\ /'___\ / __`\/\`'__/'_` \/_> _ <\/_> _ <_/\ \/'\
  \ \ \\ \\ \ \ \ \L\ /\ \__//\ \L\ \ \ \/\ \L\ \/\ \L\ \/\ \L\ \/>  </
   \ \_\ \_\ \_\ \ ,__\ \____\ \____/\ \_\ \___,_\ \____/\ \____//\_/\_\
    \/_/\/ /\/_/\ \ \/ \/____/\/___/  \/_/\/__,_ /\/___/  \/___/ \//\/_/
                 \ \_\
                  \/_/
```

# Subdomain Bruteforcing

| Tool                                                    | Description                                                                                                                     |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| [dnsenum](https://github.com/fwaeytens/dnsenum)         | Comprehensive DNS enumeration tool that supports dictionary and brute-force attacks for discovering subdomains.                 |
| [fierce](https://github.com/mschwager/fierce)           | User-friendly tool for recursive subdomain discovery, featuring wildcard detection and an easy-to-use interface.                |
| [dnsrecon](https://github.com/darkoperator/dnsrecon)    | Versatile tool that combines multiple DNS reconnaissance techniques and offers customisable output formats.                     |
| [amass](https://github.com/owasp-amass/amass)           | Actively maintained tool focused on subdomain discovery, known for its integration with other tools and extensive data sources. |
| [assetfinder](https://github.com/tomnomnom/assetfinder) | Simple yet effective tool for finding subdomains using various techniques, ideal for quick and lightweight scans.               |
| [puredns](https://github.com/d3mondev/puredns)          | Powerful and flexible DNS brute-forcing tool, capable of resolving and filtering results effectively.                           |

```shell
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```

# DNS Zone Transfers

**What is DNS Zone Transfer?**
A DNS zone transfer is essentially a wholesale copy of all DNS records within a zone (a domain and its subdomains) from one name server to another. If not adequately secured, unauthorized parties can download the entire zone file, revealing a complete list of subdomains, their associated IP addresses, and other sensitive DNS data.

![[DNS Zone Transfer.png]]

**Exploiting Zone Transfer**

```shell
dig axfr @nsztm1.digi.ninja zonetransfer.me
```

# Virtual Hosts

## Server VHost Lookup

The following illustrates the process of how a web server determines the correct content to serve based on the `Host` header:

![[Pasted image 20251208190305.png]]

1. `Browser Requests a Website`: When you enter a domain name (e.g., `www.inlanefreight.com`) into your browser, it initiates an HTTP request to the web server associated with that domain's IP address.
2. `Host Header Reveals the Domain`: The browser includes the domain name in the request's `Host` header, which acts as a label to inform the web server which website is being requested.
3. `Web Server Determines the Virtual Host`: The web server receives the request, examines the `Host` header, and consults its virtual host configuration to find a matching entry for the requested domain name.
4. `Serving the Right Content`: Upon identifying the correct virtual host configuration, the web server retrieves the corresponding files and resources associated with that website from its document root and sends them back to the browser as the HTTP response.

In essence, the `Host` header functions as a switch, enabling the web server to dynamically determine which website to serve based on the domain name requested by the browser.

# Certificate Transparency Logs

## crt.sh lookup

```sh
curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[]
 | select(.name_value | contains("dev")) | .name_value' | sort -u
```

