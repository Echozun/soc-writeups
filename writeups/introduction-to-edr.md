# Introduction to EDR

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-06-18

## Key Concepts

**What is an EDR?** Three core features: visibility, detection, and response. Visibility means high-resolution insight into what's happening on an endpoint — process activity, registry changes, file/folder changes, user actions. Detection combines signature-based matching (known-bad indicators) with behavior-based detection (machine learning flagging activity that deviates from normal). Response gives the analyst the ability to act directly on what's found.

**Beyond the antivirus.** Traditional AV is signature-based and built to catch basic, known threats — it doesn't have the visibility or behavioral analysis needed to catch anything more advanced. EDR exists to cover that gap.

**How an EDR works.** Agents deployed on each endpoint collect the data needed to build an alert, send it to a central console, and the console runs the detection logic against it. EDR is powerful but not all-encompassing — an analyst still needs to pull in other tools, not rely on the EDR alone.

**EDR telemetry.** The raw data the agents collect. More telemetry generally means better-informed judgment calls when triaging.

**Detection and response capabilities.** On the detection side: behavioral detection, anomaly detection, IOC matching, MITRE ATT&CK mapping, and ML-driven analysis. On the response side: isolate the host, terminate the process, quarantine, remote access, and artifact collection.

## Investigation

The room's practical section drops the analyst into an EDR console at a fictional company ("TECH THM") with several detections sitting in the queue across three endpoints.

**DESKTOP-HR01:** A macro-enabled document (`invoice.docm`) opened in Word, which spawned `CMD.EXE`, which launched `cURL` to pull a payload from an external domain. The file was dropped to `C:\Users\Public\install.exe` but never actually ran — the EDR caught and quarantined it before execution. Classic malware-staging behavior: get the payload onto disk first, execute later.

**WIN-ENG-LAPTOP03:** An unsigned binary, `syncsvc.exe`, running out of a user's temp directory, accessed `lsass.exe` memory — a known credential-dumping technique — and attempted to exfiltrate the result to an external upload URL. This one is a genuine compromise: process location, the unsigned binary, the LSASS access, and the outbound exfil attempt all point the same direction.

**DESKTOP-DEV01:** `UpdateAgent.exe` made an outbound connection, which is what triggered the alert in the first place — but threat intel context identified it as a known internal IT utility. Without that context, the outbound connection alone would look identical to the previous two cases. This is the same lesson as Alert Triage: the verdict comes from context, not just the raw activity.

## Takeaway

This room's value isn't the conceptual material — none of it was new — it's the reminder that EDR telemetry only matters if the analyst actually uses it to build the full process chain before calling a verdict. Two of the three endpoints here looked similarly suspicious at a glance; what separated a real compromise from a false alarm was tracing the process parentage and checking outbound destinations against what's actually expected for that host.
