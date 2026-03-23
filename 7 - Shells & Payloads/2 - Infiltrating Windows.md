**How to determine if the host is a windows machine?**

When can detect from TTL while using ICMP to determine if the host is up.
A typical response from windows hosts will either be `32` or `128`.

> [!resources]
> TTL by OS table:
> [Default TTL (Time To Live) Values of Different OS - Subin's Blog](https://subinsb.com/default-device-ttl-values/)

**PowerShell or CMD? Which one to use?**

Use `CMD` when:

- You are on an older host that may not include PowerShell.
- When you only require simple interactions/access to the host.
- When you plan to use simple batch files, net commands, or MS-DOS native tools.
- When you believe that execution policies may affect your ability to run scripts or other actions on the host.

Use `PowerShell` when:

- You are planning to utilize cmdlets or other custom-built scripts.
- When you wish to interact with .NET objects instead of text output.
- When being stealthy is of lesser concern.
- If you are planning to interact with cloud-based services and hosts.
- If your scripts set and use Aliases.

