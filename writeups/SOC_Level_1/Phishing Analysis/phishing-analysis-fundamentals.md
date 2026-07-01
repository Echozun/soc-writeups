# Phishing Analysis Fundamentals
**Source:** TryHackMe — SOC Level 1 Path, Phishing Analysis | **Date:** 2026-06-30 | **Difficulty:** Easy

## Overview
Phishing and spam have long been the most common form of social engineering. Spam is generally low risk, but phishing — and especially spear-phishing — can cause serious damage depending on how much effort the attacker puts into the pretext. This room covers the fundamentals of triaging a phishing alert: what to look at, in what order, and why.

## Key Concepts

### The Email Address
Every phishing triage starts at the basics: the email address. Addresses are always structured as `username@domain`, and that structure gives an analyst two separate things worth checking rather than one. The domain can be checked against threat intel or reputation lists of known-bad domains — a fast first filter for obvious impersonation or previously-flagged infrastructure. The username/local part is worth OSINT in its own right: does it match a real employee at the claimed organization, is it a lookalike of a legitimate address, or is it a throwaway/newly-registered identity with no footprint at all.

### Email Delivery Protocols
Several protocols work behind the scenes to move a message from sender to recipient, and knowing which one is in play changes where evidence actually lives. SMTP (Simple Mail Transfer Protocol) is the protocol that sends mail. On the receiving end, a mailbox uses either POP3 or IMAP, and the two behave very differently for an investigation. POP3 downloads mail to a single device and typically removes it from the server afterward — sent mail is stored only on the device it was sent from, so if the compromised endpoint isn't the source of evidence, the mail may not exist anywhere else. IMAP keeps mail on the server and syncs it across every connected device, with sent mail also stored server-side — meaning the mailbox itself (not just one endpoint) is the authoritative source, and evidence generally survives even if a single device is wiped or unavailable.

### Email Headers
An email splits into two main parts: the header and the body. The header carries the metadata — From (sender), To (recipient), Reply-To (where responses actually route, which is not always the same address as From and is a common spoofing tell when it isn't), Subject, and Date. None of that is fully visible in a rendered inbox view; getting the real picture means pulling the message source or the raw .eml file. That's where the deeper artifacts live: the originating source IP, and the results of DKIM and SPF checks, which are the authentication mechanisms used to catch a sender spoofing a domain they don't actually control.

### Email Body
The body carries the actual message, sent either as plaintext or as HTML (which is what supports images, styling, and embedded links). Attachments — documents, images, or other file types — ride along with the body and are one of the two most common delivery mechanisms for a payload, alongside embedded links.

### Types of Phishing
Not all unsolicited or deceptive email is the same threat:
- **Spam** — unsolicited bulk email sent to many recipients; low risk on its own, but spam carrying malicious content is reclassified as **malspam**.
- **Phishing** — email impersonating a trusted entity to trick a recipient into revealing sensitive information.
- **Spear phishing** — a targeted variant aimed at a specific individual or organization, usually built around personal information about the target to make the pretext convincing.
- **Whaling** — a spear-phishing attack that specifically targets high-value individuals such as executives, aimed at sensitive data or financial access.
- **Smishing** — phishing conducted over SMS/text.
- **Vishing** — phishing conducted over voice calls.

## Takeaway
The through-line here is that a rendered email hides almost everything an analyst actually needs: which delivery protocol was in play determines where evidence persists, and the header fields that matter most (Reply-To, source IP, DKIM/SPF) only surface once you go to the message source. Triage starts at the address, but the verdict comes from the parts of the message a normal inbox view never shows you.
