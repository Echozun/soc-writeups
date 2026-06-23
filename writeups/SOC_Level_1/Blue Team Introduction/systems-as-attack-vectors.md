# Systems as Attack Vectors

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-06-13

## Key Concepts

**Systems as targets.** When the human element is hardened — staff trained, security hygiene up to date — attackers don't have a choice but to go after the systems directly. Every system has a different value to a threat actor depending on what it's used for.

**Three paths into a system.** Human-led attacks (a malicious USB left somewhere it'll get plugged in, malware downloads, weak passwords), software vulnerabilities, and supply chain compromise.

**The vulnerability lifecycle.** Every piece of software has flaws, and some sit dormant for years before anyone finds them. If an attacker finds one before anyone else does, that's a zero-day. Once it's public, it gets a CVE number, and from there it's a race — attackers building exploits while defenders rush to patch. The answer to a CVE is always a patch.

**Misconfigurations.** A different flavor of human error — weak passwords, improper permissions — that opens a system up. The fix is a full configuration audit, but realistically, an analyst usually doesn't know about a misconfiguration until it's already been exploited.

## What was actually new

Nothing about the individual concepts was new, but the framing was worth keeping: attackers don't treat "human hacking" and "system hacking" as separate categories, they just take whichever path is open. That means equal effort has to go toward both fronts — a SOC that's strong on phishing detection but weak on patch cadence is still exposed.
