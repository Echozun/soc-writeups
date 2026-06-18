# Junior Security Analyst Intro

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-06-10

## Key Concepts

**The job is constant learning.** A SOC analyst's day isn't just monitoring and investigating alerts — the threat landscape moves fast enough that keeping up with it is part of the actual workload, not a side task.

**A SOC is not a one-person job.** It runs on a stack of roles working together: managers, engineers, analysts, incident responders, and more. An alert that lands on an L1 analyst's queue is the start of a chain, not the whole chain.

**A day in the life is mostly tickets and alerts**, with ongoing learning layered on top to keep pace with how attacks evolve.

## Situation

Tasked with reviewing alerts tied to the IP `221.181.185.159`. Two alerts fired five minutes apart: first, "Unauthorized login attempts from IP address 221.181.185.159 to port 22," then "Successful SSH login from the suspicious IP address 221.181.185.159."

## Decision

Ran the IP through the built-in OSINT tool: it traced back to China, showed involvement in 4 prior cyber attacks, and came back flagged as malicious. That sequence — failed SSH attempts followed by a successful login from an IP with an existing malicious flag and an attack history — was enough to call it real. Escalated to L2 for further investigation and remediation, and blocked the IP given the suspicious activity and the malicious flag from OSINT.
