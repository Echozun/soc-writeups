# Wireshark: The Basics

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-09

## Key Concepts

**Tool overview.** Wireshark is an open-source, cross-platform packet analyzer capable of both sniffing live traffic and inspecting existing packet captures (PCAPs). It's built for three general purposes: troubleshooting network problems (load failures, congestion), spotting security anomalies (rogue hosts, abnormal port usage, suspicious traffic patterns), and investigating protocol details (response codes, payload data) during an active analysis.

The GUI opens to one all-in-one page with five sections worth knowing at a glance:

![Wireshark GUI layout table listing Toolbar, Display Filter Bar, Recent Files, Capture Filter and Interfaces, and Status Bar](../../images/wireshark-the-basics/Pasted%20image%2020260709110412.png)

![Wireshark's actual startup screen with each of the five sections labeled and pointed out](../../images/wireshark-the-basics/Pasted%20image%2020260709110424.png)

Once a capture is loaded, the packet view itself splits into three panes: the **packet list pane** (a one-line summary per packet — source/destination address, protocol, brief info), the **packet details pane** (a full protocol breakdown of whichever packet is selected), and the **packet bytes pane** (the raw hex and decoded ASCII for that packet). Wireshark also color-codes packets by protocol and condition so anomalies stand out at a glance — those colors are configurable under the View tab — and live capture is one click away via the "shark fin" button.

**Packet dissection.** Wireshark natively supports a long list of protocols for dissection, and custom dissection scripts can be written and added on top of that native support. Every packet breaks down into 5-7 layers based on the OSI model:

![Expanded packet detail pane showing the Frame, Ethernet II, IPv4, TCP, reassembled TCP segments, and HTTP layers of a single packet](../../images/wireshark-the-basics/Pasted%20image%2020260709111429.png)

- **Frame (Layer 1)** — shows what frame or packet is being looked at, plus details specific to the physical layer.
- **Source/Destination MAC (Layer 2)** — shows the source and destination MAC address, from the data link layer.
- **Source/Destination IP (Layer 3)** — shows the source and destination IPv4 addresses, from the network layer.
- **Protocol (Layer 4)** — shows the details of the protocol in use and the source/destination ports, from the transport layer.
- **Protocol Errors** — a continuation of Layer 4, showing the specific TCP segments that needed to be reassembled.
- **Application protocol (Layer 5)** — shows the details specific to whichever application-layer protocol is in play.
- **Application Data** — an extension of Layer 5 that can show the application-specific data itself.

**Packet navigation.** Once a capture has more than a handful of packets, finding the right one matters as much as understanding it. Wireshark gives several ways to do that: packet numbers (a unique index assigned to every packet in the capture); "Go to packet," which includes jumping to the next packet in the same conversation; Find, which searches by string, hex, display filter, or regex; mark/unmark, which flags specific packets black regardless of their normal color so they stand out during an investigation; packet comments, for annotating packets directly; export packets, to split a subset out into its own file; export objects, to pull files that were transferred over the wire back out of the capture; and adjustable time display formats for whatever the investigation calls for.

**Packet filtering.** Wireshark has two distinct filter types that are easy to conflate: **capture filters**, which limit what gets captured in the first place, and **display filters**, which limit what's shown from an already-captured set without discarding anything. The fastest way to build a display filter is right-clicking a field in the packet details pane and choosing "Apply as Filter" to filter on that exact value. A related option, "Conversation Filter," filters instead for every packet in that specific back-and-forth exchange rather than just packets matching one field's value. Beyond these, the filter bar supports much more granular queries for narrowing down to exactly what's relevant.
