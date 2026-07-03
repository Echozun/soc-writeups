# Phishing Prevention

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-03

## Overview

This room covers the DNS- and cryptography-based standards mail providers use to authenticate legitimate senders and stop spoofing before it reaches an inbox, then applies that context to a packet capture of live SMTP traffic — reading server response codes and pulling apart a malicious email attachment at the protocol level.

## Key Concepts

**Sender Policy Framework (SPF):** a DNS TXT record listing the IP addresses authorized to send mail on behalf of a domain. When a message arrives, the receiving server checks the sending IP against that list. A match returns Pass/Neutral/None and the message goes through; a SoftFail or PermError flags it as suspicious but still allows it through; a Fail or TempError gets it rejected outright.

![SPF authentication flow — sender to DNS lookup to accept/reject](../../images/phishing-prevention/Pasted%20image%2020260703114521.png)

**DomainKeys Identified Mail (DKIM):** signature-based authentication. The sending server signs the message with a private key; the receiving server pulls the corresponding public key from the domain's DNS DKIM record and verifies the signature. Unlike SPF, a DKIM signature survives forwarding, which makes it useful for catching spoofing that SPF alone would miss.

![DKIM signing and verification flow — private key sign, public key match](../../images/phishing-prevention/Pasted%20image%2020260703115137.png)

**Domain-Based Message Authentication, Reporting, and Conformance (DMARC):** ties SPF and DKIM together through alignment — it checks that the domain in the visible "From" address actually matches the domains SPF and DKIM verified, closing the gap where a message could pass both checks individually but still come from a sender that doesn't match what the recipient sees.

**Secure/Multipurpose Internet Mail Extensions (S/MIME):** a public-key-cryptography standard for digitally signing and encrypting email content directly, rather than authenticating the sending server. Two components: a digital signature (integrity/authentication of the message itself) and encryption (confidentiality of the content).

![S/MIME encryption flow between two mail clients using public/private key pairs](../../images/phishing-prevention/Pasted%20image%2020260703121136.png)

## Scenario: Analyzing SMTP Responses

**Tests:** reading and filtering SMTP server response codes directly out of a packet capture in Wireshark, rather than relying on a mail client's summary of what happened to a message.

**Reasoning:** filtering on `smtp.response.code` isolates every server response in the capture, which turns a large PCAP into a workable list of pass/fail outcomes per message. From there it's a matter of counting: 19 packets carried a `220 Service ready` response (successful connection setup), and a `553` response marked a message rejected specifically because `spamhaus.org` — a real-time blocklist — flagged it (`Requested action not taken: mailbox name not allowed`). A separate check for `552` responses turned up 6 messages blocked for presenting a security issue distinct from the blocklist rejection. The value of this over a mail client's inbox/spam view is that the response code tells you *why* the server made its decision, not just that it did.

## Scenario: Inspecting Emails and Attachments

**Tests:** tracing a malicious attachment through raw SMTP traffic rather than trusting file names or extensions at face value.

**Reasoning:** filtering the same capture for `imf` (Internet Message Format) surfaces email-specific fields Wireshark parses out of the SMTP stream, including which client sent the message — in this case Microsoft Outlook Express 6.00.2600.0000, a detail that's useful for building a picture of the sender's environment. Out of 512 SMTP packets total, packet 270 carried an attachment named `document.zip`, but the message tied to it was undeliverable because the destination host IP (`212.253.25.152`) wasn't responding. The attachment itself was encoded as base64, which is standard for binary data over SMTP but also exactly how a malicious payload — in this case a disguised `attachment.scr` — gets carried without tripping plain-text content filters. The lesson here is that a `.zip` or `.scr` extension on its own means nothing without checking what's actually inside the encoded payload.

## Takeaway

SPF, DKIM, and DMARC are the prevention layer — they're designed to stop a spoofed message from ever landing cleanly in an inbox, and S/MIME extends that to the content of the message itself once it's there. But prevention isn't perfect, and this room's second half is the reason SOC analysts still need packet-level skills: when a message gets through anyway, or when you're investigating after the fact, the SMTP response codes and raw message fields in a capture tell you exactly what the mail servers decided and why, independent of whatever a mail client's UI chooses to show. Knowing the standards and knowing how to read the traffic they generate are two different skills, and this room is really testing both at once.
