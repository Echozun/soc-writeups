# SOC L1: Alert Reporting

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-06-15

## Key Concepts

**Alert funnel.** All incoming alerts hit L1 first, where the job is to triage and eliminate false positives so only confirmed true positives move up. L2 picks those up and escalates further to DFIR if active incident response is needed. Writing a full report at each handoff is what makes that funnel actually work — without it, every escalation just shifts the investigation burden upward instead of building on what L1 already found.

**Reporting guide.** L1 reports exist for three reasons: they give the next tier context so they're not starting from zero, they create a record of what was found, and writing them sharpens the analyst's own investigation skills. The structure that matters most is the 5Ws — who, what, when, where, why.

**Escalation guide and threshold.** After reaching a verdict, the question is whether it needs to go further: is this an indicator of a larger attack, does it need remediation or communication after the verdict, or is it something the analyst doesn't fully understand and needs senior input on? Having a clear threshold for "this escalates" matters because it keeps L2 from getting flooded with noise that L1 should have resolved.

**SOC communication.** Escalating correctly means bringing the right people into the loop in the right order — not just kicking a ticket upstairs and hoping someone picks it up.

**Why FP logging matters.** Correctly labeling something a false positive isn't just closing a ticket — that history is the data used to tune detection rules, keep the board from clogging with repeat noise, and serves as a reference the next time a similar alert fires.

## Incident: Email Marked as Phishing After Delivery

At 19:25 on 2025-03-27, an alert fired for an email marked as phishing after delivery. The sender, "Microsoft Support <support@microsoft.com>," sent a message titled "Important Update: Microsoft Teams Pricing Increase," using urgency-driven language ("600% price increase," "urgent notice," "download the report") and an attached file, `REPORT.rar`, to the recipient Eddie Huffman, IT Manager. Both SPF and DKIM checks failed on the message.

Between the urgent language, the implausible 600% pricing claim, the `.rar` attachment, and the failed authentication checks, this was assessed as a phishing attempt and escalated to L2 for further action.

## Incident: Spike of Domain Discovery Commands

At 19:56 the same day, an alert fired for a spike in domain discovery commands. The local system account `NT AUTHORITY\SYSTEM` on host `DMZ-MSEXCHANGE-2013` (Windows Server 2012 R2) was observed running `dir`, `hostname`, `whoami /priv`, `net group "Domain Admins" /domain`, and `nltest /dclist:tryhackme.thm` — with a parent process of `C:\Users\Public\revshell.exe` and a grandparent process of `C:\Windows\System32\inetsrv\w3wp.exe`.

That process chain — a webshell-style parent spawning domain enumeration commands — points to a successful reverse shell followed by post-exploitation discovery. Escalated as a true positive for further remediation.

## Takeaway

The two incidents land on opposite ends of the same funnel: one closes with a clean escalation decision from message-level indicators, the other from a process tree that tells the story on its own. Both get the same treatment — a report built on the 5Ws, not a gut call — because that's what makes the handoff to L2 actually useful instead of just passing the problem along.
