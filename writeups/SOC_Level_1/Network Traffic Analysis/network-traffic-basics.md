# Network Traffic Basics

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-08

## Overview

Network Traffic Analysis (NTA) is the practice of capturing, inspecting, and analyzing data as it moves across a network. This room covers the foundational model for what traffic is actually visible at different points in a network, and works through a practical exercise on where to place a monitoring tap to capture a specific type of malicious activity.

## Key Concepts

**Purpose of Network Traffic Analysis.** NTA is generally used to monitor network performance, check for abnormalities, and inspect the content of suspicious communication both internally and externally. From a SOC perspective specifically, it supports three things: detecting suspicious or malicious activity, reconstructing attacks during incident response, and verifying and validating alerts that come in from other sources.

**What traffic can be observed — the TCP/IP stack.** The clearest way to understand what's visible at any given point is the TCP/IP stack itself, since it's implemented on nearly every network-connected device:

![TCP/IP stack encapsulation diagram](../../images/network-traffic-basics/Pasted%20image%2020260708094114.png)

- **Application layer** — holds the application header and the actual payload (the data itself).
- **Transport layer** — segments the application data and wraps it with a transport header, almost always TCP or UDP.
- **Internet layer** — adds its own header (source/destination IP, TTL are the fields most commonly logged here); if a segment exceeds the MTU, it gets fragmented and each fragment gets its own header.
- **Link layer** — adds addressing information at the lowest level, including source and destination MAC address.

Each layer down the stack wraps the layer above it, which is why a capture taken at, say, the link layer still contains everything above it — encapsulated.

**Traffic sources and flows.** A corporate network has predictable sources and directions of traffic. Sources split into **intermediary** (the devices traffic passes through — firewalls, switches, web proxies, IDS/IPS, routers, access points) and **endpoint** (where traffic originates — servers, hosts, IoT devices, printers, lab machines, cloud resources, mobile phones). Flows split into **North-South** (LAN↔WAN — HTTPS, DNS, SSH, VPN, SMTP, RDP; both inbound and outbound streams pass through the firewall) and **East-West** (traffic that stays inside the LAN, and in practice gets monitored a lot less closely than North-South traffic does).

## Scenario: Tap Placement for a Malicious HTTP Download

**Tests:** applying the traffic-flow model to pick the single most efficient point to observe a specific type of traffic — in this case, North-South web traffic.

**Setup:** a user on a random workstation clicks a phishing link, triggering an HTTP request that downloads a malicious PowerShell script. The question is where a tap would need to sit to actually capture that web traffic — not everywhere would show it, only the most efficient location would.

![Network topology diagram used for the tap-placement exercise](../../images/network-traffic-basics/Pasted%20image%2020260708122241.png)

**Reasoning:** placing the tap between the firewall and the web proxy is the right call, because that's the choke point all web traffic passes through regardless of which internal host initiated it. Capturing there surfaced the HTTP response carrying the download of the `.ps1` file, which contained the flag.

## Scenario: Tap Placement for DNS-Based C2

**Tests:** the same tap-placement reasoning applied to a different protocol and a different traffic pattern — DNS instead of HTTP.

**Setup:** a workstation is compromised, and command-and-control instructions are being smuggled in through DNS TXT records rather than a normal HTTP channel. Again, the question is where to place the tap to actually see that DNS traffic.

**Reasoning:** the tap needs to sit between the DNS server and the switch it connects to — that's the point where all DNS queries and responses for the environment converge, regardless of which workstation is asking. Reviewing the traffic captured there for suspicious external connections turned up a lookup to `c2.tryhackrne.thn`, an external domain with no legitimate reason to be receiving TXT-record traffic from an internal host, which contained the flag.

## Takeaway

The two scenarios are really the same exercise twice: identify the protocol in play, then find the single point in the topology where all traffic of that type is guaranteed to converge, rather than a point where it might. Tapping at a random switch or endpoint can miss the traffic entirely depending on network layout — the firewall/proxy boundary and the DNS server's uplink are the "guaranteed convergence" points for web and DNS traffic respectively. That same logic is the first thing to check before trying to explain "why didn't we see this in the capture" during a real investigation — wrong tap placement, not a missing signature, is often the actual answer.
