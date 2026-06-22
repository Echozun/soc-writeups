# Introduction to SOAR
**Source:** TryHackMe — SOC Level 1 Path, Core SOC Solutions **Difficulty:** Medium

## Key Concepts
- **Traditional SOC and challenges** — a SOC's core job spans monitoring/detection, recovery/remediation, threat intel, and cross-team communication. In practice, several recurring problems make that job harder: alert fatigue (too many alerts and false positives overwhelm analysts), disconnected tooling (separate, non-integrated tools make investigating a single alert slower instead of easier), manual processes (repetitive steps done by hand that drive up Mean Time to Respond), and a talent shortage (the field moves fast enough that finding analysts who can keep up is its own ongoing problem).
- **Overcoming SOC challenges with SOAR** — SOAR (Security Orchestration, Automation, and Response) exists to connect the SOC's disconnected tools, threat intel feeds, and processes into a single system, so the same data and actions don't have to be manually shuttled between platforms.

  ![SOAR orchestration diagram: SIEM, TI feeds, firewalls/IDS, and IAM feeding into a central SOAR, which drives automated response, reduced alert fatigue, and consistent workflows](images/introduction-to-soar/soar-orchestration-diagram.png)

- **Building SOAR playbooks** — most of what a SOC does day to day is repetitive enough to automate, and the unit of that automation is a playbook: an "if this, then that" structure that defines exactly what the system does when a given condition is met.

## Situation
The room's threat intel practical hands you a checklist for an email-based threat intel workflow and asks which steps can be automated. The workflow: generate file hashes for extracted attachments, run those hashes and any extracted URLs through VirusTotal, fall back to manual analyst review only if VirusTotal returns nothing, and on a malicious verdict (automated or manual) delete the email, notify the organization, and update the incident ticket with the IOCs and results.

## Decision
Everything in that chain is automatable except the final delete action. Hash generation, the VirusTotal lookup, the notification, and the ticket update are all deterministic, rule-based steps with no judgment call attached — exactly the kind of repetitive work a playbook should own. The one step that should stay gated behind a human is deleting the email, since that's a destructive, hard-to-reverse action and the one place in the workflow where a wrong automated call has real cost. Manual analyst review only enters the picture as a fallback when VirusTotal itself doesn't return a verdict.
