# NetworkMiner

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-13

## Key Concepts

**NetworkMiner in forensics.** The goal of network forensics is to surface enough information from network traffic to detect malicious activity, security breaches, and anomalies. NetworkMiner supports that by giving an analyst quick, high-level hints on where to start an investigation rather than requiring a deep manual dig first: context on captured hosts (IP, MAC, hostname, OS), a list of potential attack indicators or anomalies (traffic spikes, port scans), and identification of tools or toolkits used to carry out an attack (e.g., Nmap).

**Two operating modes.** NetworkMiner works two ways. It has a sniffing feature for capturing live traffic, but that mode is Windows-only and, by most reports, less reliable than dedicated sniffing tools — it's not really what the tool is built for. Its real strength is packet parsing/processing: pointing it at an existing PCAP to get a fast overview of what's in the capture. It's explicitly a quick-overview tool, not an in-depth analysis tool.

![NetworkMiner capability table — traffic sniffing, PCAP parsing, protocol analysis, OS fingerprinting, file extraction, credential grabbing, and cleartext keyword parsing](../../images/networkminer/Pasted%20image%2020260713115356.png)

Set against Wireshark, the tradeoff is direct: NetworkMiner does OS fingerprinting and host categorization that Wireshark doesn't, and its parameter/keyword discovery is automatic where Wireshark's is manual — but it gives up protocol analysis, payload analysis, statistical analysis, and most filtering/decoding depth to get there.

![Feature comparison table between NetworkMiner and Wireshark across purpose, sniffing, PCAP handling, OS fingerprinting, keyword discovery, credential discovery, file extraction, filtering, packet decoding, protocol/payload/statistical analysis, cross-platform support, host categorization, and ease of management](../../images/networkminer/Pasted%20image%2020260713115824.png)

It's not always the right call to reach for the most powerful tool available. Wireshark can bury an analyst in information and force a slow sift through it, where a simpler tool like NetworkMiner surfaces the low-hanging fruit — host inventory, obvious anomalies, an OS fingerprint — much faster. Knowing when the fast, shallow tool is the correct first move (and when it isn't enough) is its own skill.
