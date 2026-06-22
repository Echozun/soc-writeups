# Introduction to SOAR
**Source:** TryHackMe — SOC Level 1 Path, Core SOC Solutions **Difficulty:** Medium

## Key Concepts
- **Traditional SOC and challenges** — a SOC's core job spans monitoring/detection, recovery/remediation, threat intel, and cross-team communication. In practice, several recurring problems make that job harder: alert fatigue (too many alerts and false positives overwhelm analysts), disconnected tooling (separate, non-integrated tools make investigating a single alert slower instead of easier), manual processes (repetitive steps done by hand that drive up Mean Time to Respond), and a talent shortage (the field moves fast enough that finding analysts who can keep up is its own ongoing problem).
- **Overcoming SOC challenges with SOAR** — SOAR (Security Orchestration, Automation, and Response) exists to connect the SOC's disconnected tools, threat intel feeds, and processes into a single system, so the same data and actions don't have to be manually shuttled between platforms.

  ![SOAR orchestration diagram: SIEM, TI feeds, firewalls/IDS, and IAM feeding into a central SOAR, which drives automated response, reduced alert fatigue, and consistent workflows](images/introduction-to-soar/soar-orchestration-diagram.png)

- **Building SOAR playbooks** — most of what a SOC does day to day is repetitive enough to automate, and the unit of that automation is a playbook: an "if this, then that" structure that defines exactly what the system does when a given condition is met.

## Situation
The room's case management practical gives you a list of case ticket settings (creation, updates, notifications, deletion, etc.) and asks you to flip on automation for the ones that should be automated.

## Decision
Every setting gets automated except "Delete Case Ticket." Ticket creation, updates, and notifications are routine, rule-based actions with no judgment call attached — exactly what a playbook should own. Deletion is the one action left manual, because it's destructive and hard to reverse: an unintentional automated deletion means losing case information outright, which is a worse outcome than the time saved by automating it.
