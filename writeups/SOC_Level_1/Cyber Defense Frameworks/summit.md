# Summit

**Source:** TryHackMe — SOC Level 1 Path, Cyber Defense Frameworks  
**Date:** 2026-06-29  
**Difficulty:** Medium

## Overview

Summit is a purple-team simulation that walks through the Pyramid of Pain from bottom to top. Playing as a defender against a simulated attacker ("Sphinx"), each round requires detecting the current sample and blocking it at the appropriate level — forcing the attacker to adapt and escalate until they give up entirely.

## Key Concepts

The full framework is covered in my [Pyramid of Pain writeup](pyramid-of-pain.md). This room is its practical companion — each of the six samples maps directly to one level:

| Level | Applied Block |
|-------|--------------|
| Hash Values (Trivial) | File hash of sample1.exe added to blocklist |
| IP Address (Easy) | C2 IP 154.35.10.113:4444 blocked at the firewall |
| Domain Names (Simple) | Domain emudyn.bresonicz.info blocked |
| Host Artifacts (Annoying) | Registry key disabling Windows Defender real-time protection |
| Network Artifacts (Annoying) | Beaconing pattern: 97-byte outbound connections every 30 minutes |
| TTPs (Tough) | Exfiltration method: file creation at %temp%/exfiltr8.log |

## Sample 1 — Hash Block

**Details:** The attacker announces the filename in advance: sample1.exe. Scanning it in the malware sandbox confirms it as malicious.

**Investigation:** With the executable available, the file hash is the most direct block. Sandbox verdict is clear; hash goes on the blocklist.

**Verdict:** Blocked by file hash. Lowest effort to implement, and trivially bypassed by any file modification — but it reliably catches attackers reusing unmodified tooling.

## Sample 2 — IP Block

**Details:** A modified payload with a different hash (defeating the previous block). Sandbox shows a network GET request to 154.35.10.113:4444.

**Investigation:** The changed hash means the blocklist doesn't catch it. The C2 IP is the next available artifact — block outbound traffic to that IP.

**Verdict:** Blocked by IP. The attacker still has a pool of IPs to rotate through, but this forces the swap.

## Sample 3 — Domain Block

**Details:** Another modified payload. The attacker has multiple IPs available now, making an IP block ineffective. Sandbox shows consistent DNS resolution to emudyn.bresonicz.info regardless of which IP is in use.

**Investigation:** The domain is the stable identifier across all the IP variants. Blocking it catches any IP the domain resolves to, no matter how often the attacker rotates.

**Verdict:** Blocked by domain. Registering new infrastructure costs more than rotating IPs — starts to push real pain onto the attacker.

## Sample 4 — Host Artifact Block

**Details:** sample4.exe. No single-IP or domain dependency. Sandbox shows a registry write to `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection` disabling real-time monitoring, and a spawned `backdoor.exe` process.

**Investigation:** The registry modification is the broader signal — any variant of this malware that disables real-time protection the same way will leave the same artifact. Blocking on that registry key catches the behavior rather than the specific file.

**Verdict:** Blocked by host artifact. Forces the attacker to change their actual technique, not just swap a network indicator.

## Sample 5 — Network Artifact (Sigma Rule)

**Details:** No executable this time — instead, 12 hours of outbound connection logs. Consistent connections to 51.102.10.19:443, all with exactly 97 bytes, every 30 minutes.

**Investigation:** The regularity is the signature. Fixed byte size, fixed interval — this is C2 beaconing: the compromised host checking in on a timer. A Sigma rule written against outbound connections matching 97-byte payload at 30-minute intervals catches this pattern regardless of destination IP, because the attacker can change the IP but not the communication timing without rebuilding the implant.

**Verdict:** Behavioral Sigma rule on the beaconing pattern. This is the first block that targets how the attack behaves rather than what it touches.

## Sample 6 — TTP Block (Sigma Rule)

**Details:** No executable. Instead, commands.log — the full command history from the previous sample. The attacker repeatedly writes to %temp%/exfiltr8.log throughout.

**Investigation:** The attacker's exfiltration TTP is staging data to a predictable temp path before sending it. Blocking file creation at that path shuts down the exfiltration method entirely — any payload using the same staging approach gets caught, regardless of what the file looks like or where it calls home.

**Verdict:** TTP-level Sigma rule on file creation at %temp%/exfiltr8.log. Highest pain: changing this requires redesigning the exfiltration approach from scratch.

## Takeaway

The escalation in this room is the lesson — each block at a lower pyramid level is easy to implement and just as easy for the attacker to route around. The practical caveat is that in a real L1 seat, you're not tracking a single attacker across six rounds with context from each previous block. You're clearing a queue of independent alerts, each investigated on its own merits. The individual detections in this room are all realistic: a beaconing pattern, a registry key modification, and a suspicious temp-path write all stand out in a log. The narrative of a single escalating pursuit is more thought experiment than real workflow — but the detection methods themselves hold up.
