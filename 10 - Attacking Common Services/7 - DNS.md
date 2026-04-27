# Attacking DNS

## DNS Zone Transfer

A DNS zone is a small portion of the DNS namespace that the organization or admin manages.
Since DNS comprises multiple DNS zones, DNS servers utilizes DNS Zone Transfer to copy a portion of their database to another DNS server. If the DNS servers is not configured to allow certain IPs to perform DNS Zone Transfer, anyone can ask a DNS server for a copy of its zone information (This operation doesn't require authentication).

DNS service usually runs on UDP, but when performing DNS Zone Transfer, it uses TCP.

For exploitation, we can use the `dig` utility with DNS query type `AXFR` option to dump the entire DNS namespaces from a vulnerable DNS server:

```sh
dig AXFR @ns1.inlanefreight.htb inlanefreight.htb
```

Tools like [Fierce](https://github.com/mschwager/fierce) can also be used to enumerate all DNS servers of the root domain and scan for a DNS zone transfer:

## Domain Takeovers & Subdomain Enumeration

### Domain Takeover

**Example Scenario:**

```sh
sub.target.com.   60   IN   CNAME   anotherdomain.com
```

The domain name (e.g., `sub.target.com`) uses a CNAME record to another domain (e.g., `anotherdomain.com`). Suppose the `anotherdomain.com` expires and is available for anyone to claim the domain since the `target.com`'s DNS server has the `CNAME` record. In that case, anyone who registers `anotherdomain.com` will have complete control over `sub.target.com` until the DNS record is updated.

### Subdomain Enumeration

```ad-tldr
Tools like **Subfinder**, **Sublist3r**, **Subbrute**, or [DNSDumpster - Find & lookup dns records for recon & research](https://dnsdumpster.com/)
Can be used enumerate subdomains for target domain.
```

```ad-info
We can enumerate **CNAME** records for subdomains using **nslookup** or **host** command.
```

```ad-resources
The [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz) repository can be used as a reference to check whether the target services are vulnerable to a subdomain takeover, and provides guidelines on assessing the vulnerability.
```

## DNS Spoofing

In **DNS Spoofing (DNS Cache Poisoning)**, the attack involves altering legit DNS records with false data so they can be used to redirect online traffic to fraudulent websites.

Example attack paths for the DNS Cache Poisoning are as follows:

- An attacker could intercept the communication between a user and a DNS server to route the user to a fraudulent destination instead of a legitimate one by performing a Man-in-the-Middle (`MITM`) attack.
- Exploiting a vulnerability found in a DNS server could yield control over the server by an attacker to modify the DNS records.

#### Local DNS Cache Poisoning

Attacker can perform local DNS Cache Poisoning using MITM tools like `Bettercap` or `Ettercap`. 

To exploit DNS Cache Poisoning via `Ettercap`, we need to edit `/etc/ettercap/etter.dns` file to map the target domain name that we want to spoof and the attack IP address that we want to redirect a user to.

---

```ad-important
Use multiple tools and compare results. `subbrute` was the only tool to work on the HTB lab (I tried `gobuster`, and `ffuf`, both didn't work).
```

