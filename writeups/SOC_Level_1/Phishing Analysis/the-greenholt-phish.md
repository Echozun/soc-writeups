# The Greenholt Phish

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-05

## Overview

A sales executive at Greenholt PLC escalated a suspicious email to the SOC: a generic greeting, an unexpected request for a money transfer, and an unsolicited attachment, none of which matched how this "known customer" normally communicated. This room is a capstone practical pulling together the last few phishing-analysis rooms — the task is to work a single raw email sample down to a verdict using header analysis, DNS authentication records, and attachment forensics.

## Key Concepts

**Sender field mismatch:** display name, actual sending address, and reply-to address are three separate fields that don't have to agree, and a legitimate sender has no reason for them not to. Any email where all three point to different places is worth treating with suspicion before any tooling gets involved.

**Received-chain / message-source analysis:** the visible "From" field is attacker-controlled and proves nothing on its own. The `Received:` headers in the raw message source record every hop the message actually took, and reading that chain (or running it through a header-parsing tool) surfaces the true originating server — not the one the message claims to be from.

**SPF (Sender Policy Framework):** a DNS TXT record listing which mail servers are authorized to send for a domain. A domain can have a well-formed SPF record and still have a message from it fail SPF, if the actual sending server isn't one of the ones listed.

**DMARC:** builds on SPF/DKIM by declaring what a receiving server should do with mail that fails authentication — none, quarantine, or reject. A domain publishing a strict DMARC policy is not the same as that policy actually being enforced at the point of delivery; the record states intent, it doesn't guarantee the receiving side acted on it.

**Attachment/file-type verification:** a filename and its extension are just a label the attacker chose — they're not evidence of what the file actually is. Confirming real file type and checking the file's hash against threat intelligence (VirusTotal) is what actually establishes whether an attachment is dangerous, not the name it was given.

## Investigation

The email's subject line referenced a specific "Transfer Reference Number" to make the message look like it belonged to an existing, expected transaction:

![Fake wire-transfer confirmation email referencing a Transfer Reference Number](../../images/the-greenholt-phish/Pasted%20image%2020260705105643.png)

Before any tooling came into play, the three identity fields already didn't line up: the display name read "Mr. James Jackson," the actual sending address was `info@mutawamarine[.]com`, and replies were routed to a third address, `info.mutawamarine@mail[.]com`. A genuine, established correspondent has no reason to split across three different addresses like that.

Running the message source through Google's Admin Toolbox Messageheader tool surfaced the full `Received:` chain and pointed to the true originating IP, `192.119.71.157` — separate from anything in the visible From header. The same tool also flagged the message as an outright SPF fail with an unrecognized sending IP, and DMARC as unresolved at delivery time, despite the domain publishing real records for both:

![Message header analysis showing the Received chain, SPF fail, and unresolved DMARC](../../images/the-greenholt-phish/Pasted%20image%2020260705110159.png)

WHOIS/VirusTotal attribution on that originating IP traced it to HostPapa, a budget commodity hosting provider — infrastructure with no plausible tie to a marine services company's legitimate mail servers.

An SPF lookup run directly against the Return-Path domain returned a record that looks legitimate on its face:

![SPF record lookup for the Return-Path domain](../../images/the-greenholt-phish/Pasted%20image%2020260705111042.png)

```
v=spf1 include:spf.protection.outlook.com -all
```

That record points at Outlook's mail protection service — plausible-looking — but the actual sending IP wasn't one of the servers it authorizes, which is exactly why the header tool flagged a hard fail rather than a pass. Having a well-formed SPF record doesn't help if the mail didn't come from anywhere that record actually covers.

A DMARC lookup on the same domain returned:

![DMARC record lookup for the Return-Path domain](../../images/the-greenholt-phish/Pasted%20image%2020260705111118.png)

```
v=DMARC1; p=quarantine; fo=1
```

The domain's own policy says mail failing authentication should be quarantined, with forensic reports generated on failure. That the message still reached an inbox is the practical lesson here: a domain stating the right policy is not the same as that policy being enforced by whatever received this particular message.

The attachment itself, `SWT_#09674321____PDF__.CAB`, was named to echo the same transfer reference number from the subject line and dressed up to look like a PDF receipt. Hashing it with `sha256sum` and checking that hash against VirusTotal confirmed the real file type was a RAR archive, not a PDF or CAB:

![VirusTotal details tab confirming the true file type as a RAR archive](../../images/the-greenholt-phish/Pasted%20image%2020260705111614.png)

More significantly, 50 of 63 security vendors flagged the file outright malicious, attributed to the Loki trojan family and categorized under trojan/ransomware/downloader:

![VirusTotal detection results — 50/63 vendors flagging the file as malicious](../../images/the-greenholt-phish/Pasted%20image%2020260705111526.png)

That result reframes the mismatched extension: it wasn't sloppy formatting, it was active cover for a live malware payload.

## Verdict

Phishing — confirmed malicious. No single artifact carried the whole case: the sender/reply-to mismatch, an SPF fail against a domain that otherwise has a legitimate-looking record, an originating IP with no real connection to the sender's claimed business, and an attachment VirusTotal confirms is a live trojan all point the same direction independently of each other.

## Takeaway

The formatting was the first thing that would stand out in the wild — dressed up, but not in a way that reads as genuine professional correspondence, which alone is usually enough to start pulling the thread. Stacked with the authentication failures and a disguised attachment that comes back as confirmed malware, there's no ambiguity in this one: it's phishing, not a borderline call.
