#  Network Enumeration

This document provides essential commands for enumerating network interfaces, IP addresses, DNS configurations, routing tables, and active connections in a Windows environment. Proper network enumeration is critical for identifying potential attack vectors and misconfigurations.

- Displays details about all network interfaces, including IP addresses, DNS servers, and adapters.

> Using `ipconfig`:

```bash
ipconfig /all
```

> Using PowerShell:

```powershell
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
```

> List DNS servers:

```powershell
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```


- Useful for managing IP addresses, DNS caches, and network settings.

> List all network interfaces:

```bash
ipconfig
```

> Display full network configuration:

```bash
ipconfig /all
```

> Release the IPv4 address for the specified adapter:

```bash
ipconfig /release
```

> Release the IPv6 address for the specified adapter:

```bash
ipconfig /release6
```

> Renew the IPv4 address for the specified adapter:

```bash
ipconfig /renew
```

> Renew the IPv6 address for the specified adapter:

```bash
ipconfig /renew6
```

> Display the contents of the DNS resolver cache:

```bash
ipconfig /displaydns
```

> Purge the DNS resolver cache:

```bash
ipconfig /flushdns
```

> Refresh all DHCP leases and register DNS names:

```bash
ipconfig /registerdns
```


- Checks connectivity with other systems on the network.

> Ping a host:

```bash
ping 192.168.1.7
```

> Continuous ping:

```bash
ping 192.168.1.7 -t
```

> Send a specific number of echo requests:

```bash
ping -n 1 192.168.1.7
```

> Adjust the size of the ping packet:

```bash
ping -n 1 -l 65500 192.168.1.7
```

- Traces the path that packets take to reach a specific host.

```bash
tracert 192.168.1.7
```

- Combines the functions of `ping` and `tracert`, providing network latency and loss details.

```bash
pathping 192.168.1.7
```

- Displays the ARP cache, which maps IP addresses to MAC addresses.

```bash
arp -a
```

- Queries DNS servers to resolve domain names and IP addresses.

> Lookup an IP address:

```bash
nslookup 192.168.1.33
```

> Lookup a domain name:

```bash
nslookup armour.com
```

- Displays the current routing table and network routes.

```bash
route print
```

- Displays active network connections, listening ports, and protocol statistics.

> Show basic network connection stats:

```bash
netstat
```

> Display active connections numerically:

```bash
netstat -n
```

>Show all active connections and listening ports:

```bash
netstat -an
```

>Show active connections with process IDs (PID):

```bash
netstat -ano
```

>Show TCP-specific connections:

```bash
netstat -anop tcp
```

> Show UDP-specific connections:

```bash
netstat -anop udp
```


## Usage Tips:

- `ipconfig` and `netstat` are useful for quick network troubleshooting.

- Use `tracert` and `pathping` to identify network latency or misrouting issues.

- `arp -a` helps in identifying MAC/IP conflicts or ARP spoofing attempts.

- `nslookup` can help test DNS resolution issues or identify rogue DNS servers.

- `netstat -ano` helps correlate connections with running processes using the PID.

## Related
- [User Enumeration](User-Enumeration.md) — companion host enumeration
- [Windows Version and Configuration](Windows-Version-and-Configuration.md) — companion system fingerprinting
- [Privilege Escalation Tools](Privilege-Escalation-Tools.md) — automate enumeration
- Lateral Movement — pivot using discovered hosts and routes
