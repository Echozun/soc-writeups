# Phishing Analysis Tools

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-02

## Overview

This room covers the tools and workflow an analyst uses to go deeper on the artifacts pulled from a phishing email — header analysis, OSINT enrichment, link extraction, and attachment sandboxing — then applies that workflow to a live spoofed-Netflix sample.

## Key Concepts

**Identifying artifacts:** the first step in any phishing investigation is separating what's available into two buckets. Header artifacts — sender address, sender IP, subject line, recipient address, reply-to address, date/time — come straight from the message headers. Body artifacts vary by email but generally include URLs/hyperlinks, attachment names, and attachment hashes.

**Email header analysis:** dedicated tools exist for this (Google Admin Toolbox Messageheader, Message Header Analyzer), but in practice it's usually just as fast to open the raw message source directly and pull what you need by hand. From there, the header artifacts go into OSINT lookups — IPs against VirusTotal or IPinfo, URLs against URLScan.io — to check public reputation. Using at least three sources to confirm a verdict is worth building as a habit rather than trusting a single tool's flag. One caveat: public VPN IPs will almost always show some flags from some source since they've likely been abused by someone at some point, so a flag on a VPN exit node isn't automatically damning the way the same flag would be on a dedicated host.

**Email body analysis:** the same reputation tools apply to links, but obfuscated or embedded links need extraction first — CyberChef or a dedicated URL-extraction tool handles that. The hard rule here is to never open an attachment or click a link on the analysis machine itself; that only happens in a lab or sandbox. Once you have a file hash, VirusTotal and Cisco Talos can check its reputation without opening the file at all. Unlike an IP flag, a hit on a file hash is close to a guarantee of malicious intent — but a clean result doesn't guarantee the file is safe, just that nothing's flagged it yet.

**Malware sandboxes:** for any attachment that needs to actually be detonated to see what it does, tools like Any.Run, Hybrid Analysis, and JOESandbox let you interact with it in an isolated environment and observe its behavior without putting a real system at risk.

**PhishTool:** a purpose-built platform that automates most of the above — header parsing, OSINT enrichment, link/attachment analysis — into one tool with built-in reporting. It's useful as a force-multiplier for volume, but the underlying skill it's automating is the same manual process covered above: phishing emails are largely analyzable with tools already built into a browser, no dedicated platform required to reach a verdict.

## Scenario: Your Account is on Hold (spoofed Netflix)

**Tests:** applying the header/body artifact-extraction workflow above to a live sample, working entirely from the raw message source in a mail client (Thunderbird).

**Reasoning:** The email impersonates Netflix, using an urgency-driven "account on hold" premise to push the recipient toward clicking an "Update Account" button. Working through `View → Message Source` in Thunderbird surfaces the artifacts one at a time:

- **Received-from IP:** `10.197.37.234` — the originating IP the header claims the message came from.
- **Return-Path:** resolves to `etekno[.]xyz`, a domain with no connection to Netflix and the clearest single indicator that this isn't a legitimate Netflix message — Return-Path is often overlooked in favor of the more visible "From" field, but it's harder for an attacker to spoof convincingly and worth checking every time.
- **"UPDATE ACCOUNT NOW" button:** rather than linking anywhere near a Netflix domain, it resolves to a shortened URL — `https[://]t[.]co/yuxfZm8KPg?amp=1` — which would need to be resolved or sandboxed before ever being followed, per the "never click on the analysis machine" rule above.

Between the impersonated brand, the mismatched Return-Path domain, and a shortened link masking the actual destination, this has all three of the core phishing tells covered in the header/body analysis concepts above — no exotic tooling required, just working the message source methodically.

## Takeaway

The tools in this room — VirusTotal, IPinfo, URLScan, CyberChef, sandboxes, PhishTool — all exist to answer the same handful of questions faster: does this sender/IP/domain/file have a bad reputation, and where does this link or attachment actually go? The Netflix sample is a good demonstration that the manual version of that process, done by hand in a mail client, gets you to the same verdict just as reliably. PhishTool and similar platforms earn their keep on volume and reporting overhead, not on doing anything a browser and a raw message source can't already do.
