# MITRE
**Source:** TryHackMe — SOC Level 1 Path, Cyber Defense Frameworks | **Date:** 2026-06-26 | **Difficulty:** Easy

## Overview
A walkthrough of three MITRE frameworks used in defensive security: ATT&CK, the Cyber Analytics Repository (CAR), and D3FEND. MITRE is a not-for-profit organization whose cybersecurity research underpins much of the common vocabulary and tooling used across the industry.

## Key Concepts

### ATT&CK Framework
MITRE ATT&CK (Adversarial Tactics, Techniques, and Common Knowledge) is a globally-accessible knowledge base of adversary tactics and techniques built from real-world observations of how threat actors operate. The framework organizes adversary behavior into a matrix of Tactics — the high-level goal (e.g., Persistence, Privilege Escalation, Defense Evasion) — and Techniques, the specific method used to achieve that goal. Each technique carries a unique ID, so T1053 always means Scheduled Task/Job regardless of who's talking about it.

The core value in a SOC environment is a shared language. When an analyst identifies behavior as T1053, that label is immediately understood by the threat intel team, the Tier 2 analyst receiving the escalation, and the IR team writing the post-incident report — no ambiguity, no translation. In practice, L1 analysts don't spend daily triage time navigating the ATT&CK matrix directly; the detection content already mapped to technique IDs in your SIEM or EDR does that work. Where ATT&CK becomes directly useful is when you're trying to think one step ahead: if you're seeing Scheduled Task creation (Persistence), the logical next question is what Privilege Escalation technique might follow — and the framework helps you anticipate that pivot rather than only react to what's already in the alert queue.

ATT&CK is also central to defending against APTs, which routinely reuse the same TTPs across campaigns. A threat intel report on a given group translates directly to ATT&CK technique IDs, giving defenders a concrete list of what to detect for rather than a vague behavioral description.

### Cyber Analytics Repository (CAR)
MITRE CAR is a knowledge base of detection analytics built directly on top of ATT&CK. Where ATT&CK describes what adversaries do, CAR provides ready-made detection logic for catching them doing it. Each analytic includes a data model, pseudocode representation, and in many cases direct implementations for specific tools — Splunk queries, EQL rules — that can be deployed without writing detection content from scratch.

CAR closes the gap between knowing which technique to detect and having working detection content for it. A team mapping their ATT&CK coverage can cross-reference CAR to see whether validated analytics already exist for a given technique before spending time building their own.

### D3FEND
D3FEND (Detection, Denial, and Disruption Framework Empowering Network Defense) is MITRE's framework for the defensive side of the equation. Where ATT&CK catalogs what attackers do, D3FEND maps the countermeasures — detection, denial, and disruption techniques that security controls can implement to address them. It establishes a common language for describing how defensive controls actually function, making it useful for coverage analysis: given a set of ATT&CK techniques you're trying to defend against, D3FEND helps identify which defensive techniques apply and where gaps remain.

![D3FEND framework overview](../../images/mitre/Pasted%20image%2020260626112326.png)
