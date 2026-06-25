# Unified Kill Chain
**Source:** TryHackMe — SOC Level 1 Path, Cyber Defense Frameworks **Difficulty:** Easy

## Overview
The Unified Kill Chain is an 18-phase attack framework designed to model a modern intrusion from first contact through final objectives — a more complete and realistic alternative to earlier kill chain models that only covered part of the attack lifecycle.

## Key Concepts
### Kill Chain
A term borrowed from military doctrine describing the full sequence of stages required to complete an attack. In cybersecurity, it's used to map out the methodological path an attacker takes from initial planning through achieving their objective. Understanding a kill chain gives defenders a predictive edge: if you know what phase an attacker is in, you can anticipate what comes next and identify the earliest feasible point to intervene — rather than only reacting after damage is done.

### Threat Modelling
A structured process for improving the security posture of a system through four steps: identifying what systems and applications need to be secured and what role they serve in the environment, assessing the vulnerabilities and weaknesses those systems carry, creating an action plan to address the identified risks, and implementing policies to prevent recurrence where possible. The goal is to move from reactive, after-the-fact patching toward deliberate, prioritized defense — knowing not just that a vulnerability exists, but which ones matter most given what an attacker would actually want to achieve against that environment.

### The Unified Kill Chain
The UKC expands on earlier frameworks by covering all 18 phases of a modern intrusion in a single model. Where something like Lockheed Martin's Cyber Kill Chain stops at payload delivery and execution, the UKC continues through lateral movement, privilege escalation, and final objectives — reflecting how real intrusions actually unfold rather than stopping at initial compromise. Its 18 phases are organized under three goals:

![Unified Kill Chain overview diagram](../../images/unified-kill-chain/Pasted%20image%2020260625111642.png)

#### In: Initial Foothold
The "In" goal covers everything an attacker does to gain their first presence inside a target environment. This includes reconnaissance, weaponization, social engineering or delivery, initial exploitation, and early persistence and defense evasion. The emphasis at this stage is preparation — attackers want to complete as much groundwork as possible before touching anything that could generate alerts, which means passive OSINT and planning happens well before any payload is delivered.

![In: Initial Foothold phases](../../images/unified-kill-chain/Pasted%20image%2020260625114035.png)

#### Through: Network Propagation
Once inside, the attacker needs to expand their reach. The "Through" goal covers pivoting to new systems, privilege escalation, internal discovery, credential access, and evading defenses within the internal network. The attacker is rarely satisfied with their initial foothold — the systems they need access to in order to achieve their real objectives are typically deeper in the environment, which means lateral movement and privilege escalation are almost always necessary steps.

![Through: Network Propagation phases](../../images/unified-kill-chain/Pasted%20image%2020260625114252.png)

#### Out: Actions on Objectives
The "Out" goal represents what the attacker actually came for — data collection and exfiltration, destruction, impact, or establishing long-term persistence for future operations. Everything in the "In" and "Through" goals was in service of reaching this stage. From a defender's perspective, this is also frequently the stage where an intrusion finally surfaces in the alert queue: exfiltrating data in volume or causing impact produces noise that earlier, quieter reconnaissance and lateral movement stages didn't.

![Out: Actions on Objectives phases](../../images/unified-kill-chain/Pasted%20image%2020260625114535.png)

## Practical Application
The room's practical exercise presents five statements describing attacker behaviors and asks you to categorize each into the correct UKC phase.

| Statement | Phase |
|---|---|
| The attacker uses tools to gather information about a system. | Reconnaissance |
| The attacker installs a malicious script to allow them remote access at a later date. | Persistence |
| The hacked machine is being controlled from an attacker's own server. | Command & Control |
| The attacker uses the hacked machine to access other servers on the same network. | Pivoting |
| The attacker steals a database and sells this to a 3rd party. | Exfiltration |

**Flag:** `THM{UKC_SCENARIO}`
