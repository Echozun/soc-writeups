# SOC Workbooks and Lookups

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-06-16

## Key Concepts

**Assets & Identities.** Most alerts cite a system (asset) and a user (identity), and neither one means much on its own — the same alert can be routine or a real problem depending on who triggered it and what access they're supposed to have. Companies maintain inventories describing what each system and employee is for, and pulling that context is usually the first real step in triage.

**Network Diagrams.** For network-related alerts, a diagram is what lets an analyst tell normal traffic from abnormal and trace an IP through NAT translation (public-facing address mapped to an internal one) to find out what actually happened on the network.

**Workbook Theory.** With the sheer range of alert types a SOC sees, no analyst remembers every step for every rule. A workbook is a step-by-step guide for a specific alert type that ensures the analyst gathers full context before reaching a verdict (TP or FP) — it's what keeps triage consistent across different analysts and different shifts.

## Scenario: Email Analysis Workbook

**Tests:** the procedure for triaging a suspicious email.

**Procedure:** take ownership of the alert, then use the identity inventory to get context on the recipients. Investigate the email itself — SPF/DKIM results, content, sender domain reputation — through an EML analyzer. Move to the attachment: sandbox any binaries, manually review any scripts. If the activity turns out malicious, gather the triage evidence (recipient list, sandbox report, EML analyzer results, other indicators), write the report for L2 with that evidence attached, and escalate. If it's not malicious, write a short comment explaining why it's safe and expected, and close as FP.

## Scenario: PowerShell Analysis Workbook

**Tests:** the procedure for triaging a suspicious PowerShell execution.

**Procedure:** assign the alert, pull context on the affected machine from the asset inventory. Run threat intel against any static URL involved and analyze the executable. Build a process tree to find the parent process and the account that started PowerShell. If malicious, save the process tree and login timeline SIEM searches, write the L2 report with evidence and assumptions attached, and escalate. If not malicious, write a short comment explaining the verdict and loop in SOC engineers to tune the detection rule.

## Scenario: Network Analysis Workbook

**Tests:** the procedure for triaging a suspicious network scan.

**Procedure:** assign the alert, pull IP context from the network map and asset inventory. List the ports scanned and what services could be running on them. If the source is a known corporate scanner (Nessus, Zabbix, etc.), verify the scanning pattern matches what's expected. If malicious, collect the evidence (attack time range, scanned ports, IP context), write the report for L2, and escalate. If not malicious, write a comment explaining the verdict and reach out to SOC engineers to tune the rule.

## Takeaway

All three workbooks follow the same shape: get context first (identity/asset/network), investigate with the right tool for that alert type, then branch into either "build the evidence package and escalate" or "explain the verdict and close." That consistency is the point — it's not about memorizing three different processes, it's one process applied to three different alert types. Used this exact pattern repeatedly during my time at Reliaquest.
