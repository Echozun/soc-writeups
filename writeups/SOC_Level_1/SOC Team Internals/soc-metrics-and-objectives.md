# SOC Metrics and Objectives

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-06-17

## Key Concepts

**Core metrics.** The numbers an L1 analyst needs to know cold: Alert Count, False Positive Rate, Alert Escalation Rate, and Threat Detection Rate.

**Triage metrics.** SOC Availability, Mean Time to Detection (MTTD — time from the actual event to the alert firing), Mean Time to Acknowledge (MTTA — time from alert creation to an analyst picking it up), and Mean Time to Respond (MTTR — time from alert generation to the end of the required response, whether that's a phone call or full remediation).

**Why these matter beyond the dashboard.** These metrics aren't just operational health checks — they're also used to evaluate individual analysts at both L1 and L2. A metric trending the wrong way can mean the difference between catching a real attack in time and missing it.

## Scenario: Unhappy Customer

**Tests:** Mean Time to Respond.

**Situation:** A major customer's CFO had their email and Entra ID account breached, and it took close to 6 hours to remove the attacker from the mailbox — more than enough time for emails to be exfiltrated.

**Reasoning:** A 5+ hour gap just to reset credentials and enforce MFA is an MTTR failure, not a detection failure — the alert fired, the response was just too slow. The fix isn't more alerting, it's documenting the credential-rotation procedure so the next analyst handling a similar incident doesn't lose hours figuring out the steps.

## Scenario: Delayed Alert

**Tests:** Mean Time to Detection.

**Situation:** Analysts spent the first 20 minutes watching the screen waiting for an alert to appear, despite having adequate staff on hand.

**Reasoning:** A 20-minute gap between the actual attack starting and the alert firing is an MTTD problem — the detection rule itself was too slow to catch it, not a staffing issue. The fix is having SOC engineers tighten the detection rule's execution schedule (e.g., running every 5 minutes instead of a longer interval).

## Scenario: Tired Analysts

**Tests:** False Positive Rate.

**Situation:** Across an 8-hour shift, L1 analysts closed 760 alerts, 95% of which were noise from internal IT systems or automation scripts.

**Reasoning:** A 95% false positive rate means analysts can't realistically stay vigilant — that volume of noise is the direct path to alert fatigue and missed real threats. The fix is working with SOC engineers to exclude known-legitimate automation/IT activity from the detection rules generating that noise.

## Takeaway

All three scenarios trace back to the same idea: a bad metric almost never means "try harder," it means something specific in the pipeline — response procedure, detection rule timing, or rule tuning — needs to change. These exact metrics were tracked as part of my own evaluations at Reliaquest, so the scenarios aren't hypothetical — they're the kind of thing that shows up in a real performance review.
