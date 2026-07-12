# Wireshark: Traffic Analysis

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-12

## Key Concepts

### Detecting Nmap Scans

Nmap is the industry-standard tool for network mapping, host discovery, and service enumeration — which also makes recognizing its traffic signatures a baseline analyst skill. The three most common scan types leave distinct fingerprints in Wireshark:

- **TCP connect scans** complete the full three-way handshake (since the underlying OS socket API requires it), and typically show a TCP window size larger than 1024 bytes because a real connection expects to exchange data.
- **SYN scans** ("half-open" scans) never complete the handshake and are only available to privileged users. They typically carry a window size of 1024 bytes or less.
- **UDP scans** don't involve a handshake at all — there's no positive signal for an open port, but a closed port returns an ICMP "port unreachable" error (type 3, code 3).

The base display filters for probing this behavior:

![TCP flag reference table mapping flag combinations (SYN, ACK, RST, FIN) to their Wireshark filter syntax](../../images/wireshark-traffic-analysis/Pasted%20image%2020260712144533.png)

```
TCP Connect scan:  tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024
SYN scan:          tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024
UDP scan:          icmp.type==3 and icmp.code==3
```

### ARP Poisoning & Man-in-the-Middle

Address Resolution Protocol (ARP) is what lets devices resolve IP addresses to MAC addresses on a local network. ARP poisoning abuses this by sending forged ARP packets to a target (often the default gateway), corrupting its IP-to-MAC mapping table so the attacker's machine ends up in the traffic path — a classic man-in-the-middle setup.

The tell in Wireshark is a **conflict**: two different ARP responses claiming the same IP address. Wireshark flags the second (conflicting) response, but doesn't tell the analyst which one is legitimate — that call requires knowing the network's actual architecture and inspecting the traffic timeline around the conflict.

![Two stacked Wireshark captures showing a normal ARP request/reply exchange — "Who has 192.168.1.1? Tell 192.168.1.25" followed by the reply "192.168.1.1 is at 50:78:b3:f3:cd:f4"](../../images/wireshark-traffic-analysis/Pasted%20image%2020260712150456.png)

![ARP filter reference table — opcode 1 (request) vs opcode 2 (reply), plus hunt filters for ARP scanning, poisoning detection, and flooding](../../images/wireshark-traffic-analysis/Pasted%20image%2020260712151518.png)

Useful filters: `arp.opcode == 1` / `== 2` to split requests from replies, and `arp.duplicate-address-detected or arp.duplicate-address-frame` to jump straight to the conflicting packets.

### Identifying Hosts and Users: DHCP, NetBIOS, and Kerberos

Being able to tie traffic back to a specific host or user is a core investigative skill. Most enterprise networks follow a predictable naming convention for hosts and accounts — which cuts both ways: it makes legitimate identification easy, but it also gives an adversary a simple pattern to imitate and blend in with. Three protocols carry this identifying information:

**DHCP** — DHCP Request packets carry the hostname, ACK packets confirm accepted leases, and NAK packets show denials. Option 53 (request type) has fixed values worth filtering on directly; useful sub-fields include Option 12 (hostname), Option 50 (requested IP), Option 51 (lease time), Option 61 (client MAC), and Option 56 (NAK rejection reason).

![DHCP protocol overview and filter reference table — Request/ACK/NAK message types and their relevant option fields](../../images/wireshark-traffic-analysis/Pasted%20image%2020260712152714.png)

**NetBIOS (NBNS)** — resolves names between applications on different hosts on the local network. Query packets carry the queried name, TTL, and IP address details.

![NetBIOS Name Service protocol overview and filter reference (nbns.name contains "keyword")](../../images/wireshark-traffic-analysis/Pasted%20image%2020260712152849.png)

**Kerberos** — the default authentication protocol for Windows domains, proving identity between two hosts over an untrusted network. The `CNameString` field carries the username — with one wrinkle: some Kerberos packets put a hostname in that same field, so filtering out values ending in `$` (the hostname convention) is necessary to isolate real usernames. Other useful fields: `pvno` (protocol version), `realm` (domain name for the ticket), and `SNameString` (service name — `krbtgt` marks a Ticket Granting Ticket request).

![Kerberos protocol overview and filter reference — CNameString username filtering, and pvno/realm/sname fields](../../images/wireshark-traffic-analysis/Pasted%20image%2020260712152903.png)

### Tunneling Traffic: DNS and ICMP

Tunneling hides one protocol's traffic inside another, and in an attacker's hands it becomes a way to smuggle command-and-control (C2) traffic or exfiltrated data past network controls that would otherwise catch it.

**ICMP** is built for diagnosing network issues (ping, traceroute), but it's also abused for denial-of-service and as a covert channel for C2 or exfiltration. Because legitimate ICMP traffic is normally low-volume and uniformly sized, ICMP tunneling tends to show up as an anomaly that starts appearing after a malware execution or exploitation event — a sudden spike in ICMP volume, or packets with unusual/inconsistent sizes, are the indicators to look for.

**DNS tunneling** follows a similar pattern and similar timing (it typically starts after initial compromise). The adversary controls a domain configured as a C2 channel; the compromised host's malware or post-exploitation commands get encoded into DNS queries rather than sent as normal application traffic. The giveaway is that these aren't real address lookups — they're abnormally long queries for crafted subdomains that are actually encoded data, e.g. `encoded-commands.maliciousdomain.com`.

### Cleartext Protocol Analysis: FTP

FTP was designed for simplicity, not security — everything, including credentials, goes over the wire in plaintext, which makes it a good protocol to practice cleartext traffic analysis on.

![FTP response code reference — x1x (information), x2x (connection), x3x (authentication) series with example codes](../../images/wireshark-traffic-analysis/Pasted%20image%2020260712170320.png)

![FTP response codes continued (230/231/331/430/530) plus request commands (USER/PASS/CWD/LIST) and advanced brute-force/password-spray detection filters](../../images/wireshark-traffic-analysis/Pasted%20image%2020260712170335.png)

The response code series map cleanly to a phase of the exchange — x1x for information requests, x2x for connection status, x3x for authentication — and `200` always means the command succeeded. Filtering `ftp.request.command == "USER"` and `== "PASS"` pulls the credential exchange directly out of the capture, and `ftp.response.code == 530` combined with a repeated username is a straightforward brute-force signal, while a single repeated password across many usernames flags a password-spray attempt.

## Takeaway

This room is a tour of the signatures — Nmap scans, ARP conflicts, DHCP/NetBIOS/Kerberos identity fields, DNS/ICMP tunneling anomalies, and cleartext FTP — that show up in raw packet captures during network traffic analysis. In a real SOC, purpose-built tools (NDR/IDS platforms, SIEM correlation) usually surface these patterns automatically; Wireshark's value is in the deep-dive case where an analyst needs to verify or drill into what an alert is actually seeing at the packet level.
