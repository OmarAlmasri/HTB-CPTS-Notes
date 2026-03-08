# Determine Firewalls and Their Rules

We already know that when a port is shown as filtered, it can have several reasons. In most cases, firewalls have certain rules set to handle specific connections. The packets can either be `dropped`, or `rejected`. The `dropped` packets are ignored, and no response is returned from the host.

This is different for rejected packets, which elicit an explicit response. TCP packets are returned with an RST flag, while ICMP can contain different types of error codes.

Such errors include:

- Net Unreachable
- Net Prohibited
- Host Unreachable
- Host Prohibited
- Port Unreachable
- Proto Unreachable

Nmap's TCP ACK scan (`-sA`) method is much harder to filter for firewalls and IDS/IPS systems than regular SYN (`-sS`) or Connect scans (`sT`) because they only send a TCP packet with only the `ACK` flag. When a port is closed or open, the host must respond with an `RST` flag. Unlike outgoing connections, all connection attempts (with the `SYN` flag) from external networks are usually blocked by firewalls. However, the packets with the `ACK` flag are often passed by the firewall because the firewall cannot determine whether the connection was first established from the external network or the internal network.
