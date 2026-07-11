# Wireshark: Packet Operations

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-11

## Key Concepts

### Statistics — Summary and Overview Tools
Wireshark's Statistics menu gives analysts a way to step back from individual packets and see the shape of a capture as a whole — the scope of the traffic, which protocols are present, and which endpoints and conversations are involved. A few options are especially useful for getting oriented in a new capture:

- **Resolved Addresses** lists every IP address in the capture alongside its resolved DNS name where one is available, giving a quick inventory of the hosts involved without hunting through individual packets.
- **Protocol Hierarchy** breaks down every protocol present in the capture into a tree view, showing packet counts and percentages for each — a fast way to see what's dominating the traffic.
- **Conversations** shows the traffic exchanged between two specific endpoints.
- **Endpoints** is closely related but reports information for a single field individually (e.g., everything tied to one IP) rather than a pairwise exchange between two.

### Statistics — Protocol-Specific Details
Most of the general Statistics views mix IPv4 and IPv6 traffic together by default. Wireshark's **IPvX Statistics** option (Statistics → IPvX Statistics) narrows the view down to a single IP version, letting an analyst isolate everything tied to that version in one window rather than filtering it out packet by packet.

Two more targeted views sit alongside it: a **DNS** breakdown, which decomposes every DNS packet in the capture into a tree view by packet count and percentage, and an equivalent **HTTP** breakdown that does the same for HTTP traffic — both useful for a fast read on protocol-specific activity without building a custom filter first.

### Packet Filtering Principles
Wireshark distinguishes between two filter types: **capture filters**, which limit what gets captured in the first place, and **display filters**, which narrow what's shown from an already-captured set without discarding anything. Standard practice is to capture broadly and filter afterward, since a capture filter that's too narrow risks missing the very traffic under investigation.

Capture filter syntax operates at a lower level — byte offsets, hex values, and masks combined with boolean operators. Display filters are Wireshark's more powerful and far more commonly used tool: they support roughly 3,000 protocols and allow packet-level searches against any field in the protocol breakdown, with the official [Display Filter Reference](https://www.wireshark.org/docs/dfref/) documenting the full set.

Display filters are built from comparison operators and logical expressions:

![Comparison operators table — eq, ne, gt, lt, ge, le with C-like equivalents and examples](../../images/wireshark-packet-operations/Pasted%20image%2020260711143248.png)

![Logical expressions table — and, or, not with C-like equivalents and examples](../../images/wireshark-packet-operations/Pasted%20image%2020260711143303.png)
