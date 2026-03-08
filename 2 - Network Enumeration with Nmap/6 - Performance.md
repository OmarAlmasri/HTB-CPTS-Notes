Scanning performance plays a significant role when we need to scan an extensive network or are dealing with low network bandwidth. We can use various options to tell `Nmap` how fast (`-T <0-5>`), with which frequency (`--min-parallelism <number>`), which timeouts (`--max-rtt-timeout <time>`) the test packets should have, how many packets should be sent simultaneously (`--min-rate <number>`), and with the number of retries (`--max-retries <number>`) for the scanned ports the targets should be scanned.

# Timeouts

When Nmap sends a packet, it takes some time (`Round-Trip-Time` - `RTT`) to receive a response from the scanned port. Generally, `Nmap` starts with a high timeout (`--min-RTT-timeout`) of 100ms. Let us look at an example by scanning the whole network with 256 hosts, including the top 100 ports.

**Default Scan**

```shell
Ripcord88x@htb[/htb]$ sudo nmap 10.129.2.0/24 -F <SNIP> Nmap done: 256 IP addresses (10 hosts up) scanned in 39.44 seconds
```

**Optimized RTT**

```shell
Ripcord88x@htb[/htb]$ sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms <SNIP> Nmap done: 256 IP addresses (8 hosts up) scanned in 12.29 seconds
```

| **Scanning Options**         | **Description**                                       |
| ---------------------------- | ----------------------------------------------------- |
| `10.129.2.0/24`              | Scans the specified target network.                   |
| `-F`                         | Scans top 100 ports.                                  |
| `--initial-rtt-timeout 50ms` | Sets the specified time value as initial RTT timeout. |
| `--max-rtt-timeout 100ms`    | Sets the specified time value as maximum RTT timeout. |
When comparing the two scans, we can see that we found two hosts less with the optimized scan, but the scan took only a quarter of the time. From this, we can conclude that setting the initial RTT timeout (`--initial-rtt-timeout`) to too short a time period may cause us to overlook hosts.

# Max Retries

Another way to increase scan speed is by specifying the retry rate of sent packets (`--max-retries`). The default value is `10`, but we can reduce it to `0`. This means if Nmap does not receive a response for a port, it won't send any more packets to that port and will skip it.

# Rates

During a white-box penetration test, we may get whitelisted for the security systems to check the systems in the network for vulnerabilities and not only test the protection measures. If we know the network bandwidth, we can work with the rate of packets sent, which significantly speeds up our scans with `Nmap`. When setting the minimum rate (`--min-rate <number>`) for sending packets, we tell `Nmap` to simultaneously send the specified number of packets. It will attempt to maintain the rate accordingly.

**Default Scan**

```shell
Ripcord88x@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.default <SNIP> Nmap done: 256 IP addresses (10 hosts up) scanned in 29.83 seconds
```

**Optimized Scan**

```shell
Ripcord88x@htb[/htb]$ sudo nmap 10.129.2.0/24 -F -oN tnet.minrate300 --min-rate 300 <SNIP> Nmap done: 256 IP addresses (10 hosts up) scanned in 8.67 seconds
```

