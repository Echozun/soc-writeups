# Cyber Kill Chain
**Source:** TryHackMe — SOC Level 1 Path, Cyber Defense Frameworks **Difficulty:** Easy

## Overview
Lockheed Martin's Cyber Kill Chain models an intrusion as a sequence of stages an attacker has to move through in order, borrowed from the military concept of a "kill chain." The framework's defensive value is that interrupting any single stage breaks the whole chain — a defender doesn't need to stop every stage, just one.

## Key Concepts
### Reconnaissance
The research and planning phase. OSINT is usually the starting point, since publicly available information gives an attacker a free foundation to build on before spending any effort on direct contact. Recon splits into two modes: passive (no direct interaction with the target — WHOIS lookups, social media scraping, reviewing breach data, scraping job postings for the tech stack a company runs) and active (direct contact — social engineering, port scanning, banner grabbing, probing for open services). Passive recon is effectively undetectable from the target's side, since nothing ever touches their infrastructure; active recon starts generating logs and alerts the moment it begins, which is why attackers try to extract as much value as possible from passive methods first.

### Weaponization
Once an attacker has enough recon, they convert that raw information into an actual attack tool — crafting malware or packaging an exploit into a deliverable payload. This is also where the payload gets tailored to what recon turned up: if recon revealed a specific vulnerable application version, the exploit gets built to target that exact version rather than something generic. Weaponization happens entirely on the attacker's side, away from the target, so a defender has effectively no visibility into this stage as it's happening.

### Delivery
The mechanism for getting the weaponized payload to the target. Common vectors include phishing emails, malicious attachments or links, watering-hole attacks (compromising a site the target is known to visit), and physical drops (e.g. a USB left where someone will plug it in). Delivery is the first stage where the attacker's activity is actually visible to the defender — it's the point where email gateways, web proxies, and endpoint controls get their first real shot at stopping the chain before anything executes.

### Exploitation
The stage where the payload actually executes — the point where the attacker's code runs on the target system, typically by abusing a vulnerability in an application, OS, or a user action (like enabling a malicious macro). This is the hinge point of the whole chain: everything before it is preparation, and everything after it depends on this step succeeding.

### Installation
After gaining initial execution, the attacker's priority is establishing persistence so they don't lose access if the initial foothold closes (a reboot, a patch, a user closing the malicious document). Typical methods: a webshell, modifying Windows services, or adding "run keys" in the Registry or Startup folder so the payload survives a reboot. This stage is also where a single point of execution turns into a durable presence on the system — the difference between a one-off compromise and an actual foothold.

### Command & Control (C2)
The attacker stands up a C2 channel to remotely control the compromised system. The most common approach is blending in with normal traffic over HTTP/HTTPS (ports 80/443), since that traffic looks legitimate to a firewall and to most network monitoring that isn't doing deep packet inspection. DNS-based C2 is another common channel — the infected machine makes constant DNS requests to a server the attacker controls, using the responses themselves to smuggle commands or data, which can slip past controls that only watch HTTP/HTTPS traffic. C2 is also where an attacker usually expands their reach within the environment — issuing commands for lateral movement, additional recon on internal systems, or pulling down more tooling — rather than just sitting on the one compromised host.

### Actions on Objectives
Whatever the attacker actually came for: data exfiltration, data corruption, further reconnaissance, or privilege escalation. This is the payoff stage — everything before it was setup. It's also frequently the stage where an intrusion finally surfaces to a defender, since exfiltrating large volumes of data or destroying systems tends to produce noise (unusual outbound traffic volume, file integrity changes) that earlier, quieter stages didn't.

## Practical Application: The 2013 Target Breach
The room's practical exercise maps a real intrusion onto the kill chain rather than a hypothetical: the November 2013 Target breach, which ultimately exposed roughly 40 million credit and debit card accounts.

- **Reconnaissance:** the attackers didn't go after Target directly first — they researched and targeted a third-party HVAC vendor with network access into Target's environment, a classic supply-chain angle.
- **Weaponization:** the attackers built out PowerShell-based tooling to use once they had a foothold.
- **Delivery:** a spearphishing attack against the vendor was the entry vector, used to harvest the vendor's credentials.
- **Exploitation:** those stolen vendor credentials were used to pivot into Target's network and reach point-of-sale systems.
- **Installation:** persistence was maintained on the compromised systems so the access survived past the initial intrusion.
- **Command & Control:** the attackers kept fallback channels available, so even when some of their primary access got blocked, they still had a path to keep exfiltrating.
- **Actions on Objectives:** the objective was cardholder data — the attackers exfiltrated payment card information harvested from the point-of-sale systems.

What makes this exercise useful over a purely abstract one is the third-party angle: it's a reminder that "the target" in an attack chain isn't always the org being breached — Target's own perimeter wasn't the weak point, a vendor's was.
