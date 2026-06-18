# SOC L1: Alert Triage

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-06-14

## Overview

This room hands you a queue of three alerts from a SOC dashboard and asks you to triage them the way an L1 analyst would: pick them in priority order (severity first, then oldest first), assign each to yourself before investigating, work the alert details down to a verdict, and justify that verdict with evidence rather than a guess.

## Key Concepts

**Alert lifecycle:** an alert starts as an event on a system — a login, a process launch, a file download. The OS, firewall, or cloud provider logs that event, the log gets shipped to a SIEM or EDR for processing, and rule logic in that tool decides whether to fire an alert. This is the actual pipeline I worked against every day for two years, not a simplified version of it.

**Alert properties:** every alert carries the same core fields regardless of platform — time, rule name, severity, status, assignee, verdict, description, and the specific fields tied to that rule. Knowing these cold matters because triage speed depends on going straight to the fields that matter for that alert type instead of reading the whole thing top to bottom.

**Prioritization:** filter the queue, then sort by severity, then by age within each severity band — work the oldest critical before a newer one, and don't pick up an alert someone else already has in progress. That's genuinely how the SOCs I worked in ran the queue, not a textbook simplification.

## Alert: Potential Data Exfiltration

**Details:** Critical severity, 2025-03-21 13:30. Source IP `192.168.45.66`, source network `UK04/MEETINGROOM`, destination `*.zoom.us`. 5.8 GB sent, 5.2 GB received.

**Investigation:** Worked this one first since it was the highest severity in the queue. The source network being a meeting room and the destination being Zoom's domain space pointed toward expected behavior right away — large bidirectional transfers are normal for video conferencing. In a real SOC with SIEM access, I'd pull the logs on the source device and any associated users to rule out anything riding alongside the legitimate traffic. That tooling wasn't available in this room, so the verdict rests on the network/destination context alone.

**Verdict:** False positive — authorized and expected Zoom traffic from a meeting room.

## Alert: Double-Extension File Creation

**Details:** High severity, 2025-03-21 13:58. Host `LPT-HR-009`, user `S.Conway`, process `chrome.exe`, target file `C:\Users\S.Conway\Downloads\cats2025.mp4.exe`, downloaded from `https://freecatvideoshd.monster/cats2025.mp4.exe`, MD5 `14d8486f3f63875ef93cfd240c5dc10b`.

**Investigation:** The real extension being `.exe` while masquerading as an `.mp4` was the immediate red flag — classic double-extension phishing. Ran the file hash through OSINT (VirusTotal): 49 of 69 engines flagged it as a trojan under the alias `h0t.exe`. That detection rate was enough on its own to escalate. In a real environment I'd still pivot on the host and user to confirm whether the file was actually executed or got quarantined first, and check for related activity on both — but for this alert, the hash result was sufficient evidence to call it.

**Verdict:** True positive — confirmed malicious file (trojan), downloaded from an untrusted domain disguised as a video file.

## Alert: Download from GitHub Repository

**Details:** Low severity, 2025-03-21 13:02. Accessed URL `https://github.com/facebook/react`, user `G.Chandler`, host `LPT-IT-063`, source network `VPN/DEVELOPERS`.

**Investigation:** Two things lined up here: the repo itself (React, maintained by Meta — a known, trusted source) and the source network being the developer VPN segment. Both point toward routine developer activity rather than anything suspicious. In a real case I'd confirm with identity management that the user is actually provisioned as a developer, plus the standard pivot for unrelated suspicious activity on the host — but the repo legitimacy and network context were enough to close this one out.

**Verdict:** False positive — authorized developer activity, legitimate repository.

## Takeaway

Alert triage was the entire job for two years, not a piece of it — this room is the job description compressed into three tickets. The thing worth naming explicitly: two of these three closed as false positives, and both times the call came down to *context* (which network, which user role, which destination) rather than anything technical about the traffic itself. The one true positive turned on a single concrete artifact — a hash that came back dirty. That's the actual shape of L1 triage: most of the queue is legitimate activity that looks alarming out of context, and the job is telling the difference fast without missing the one that isn't.
