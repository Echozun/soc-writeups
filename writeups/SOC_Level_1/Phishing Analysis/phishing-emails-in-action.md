# Phishing Emails in Action

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-01

## Overview

This room walks through four real phishing samples, each built around a different delivery method — a fake receipt, a fake shipping notice, a multi-stage credential-harvesting redirect, and a malicious attachment — and asks you to work each one down to a verdict using the artifacts in the email itself.

## Key Concepts

**Spoofed sender address:** the display name and the actual sending address are two different fields, and a phishing email exploits that gap — showing a trusted name (PayPal, a shipping carrier, Netflix) while the underlying address resolves to a domain that has nothing to do with that company.

**URL shortening and link manipulation:** a shortened link (`is.gd`, `bit.ly`, etc.) or a hyperlink whose display text doesn't match its actual `href` both serve the same purpose — hiding the real destination behind something that looks legitimate or innocuous. When a mail client disables hover-to-preview (Yahoo does this by default on suspected spam), the only way to recover the true destination is to pull the message source and read the raw HTML.

**Pixel tracking:** a 0x0 or otherwise invisible image embedded in the email body that silently notifies the attacker's server when the message is opened, confirming a live, monitored inbox — independent of whether the recipient clicks anything.

**Brand impersonation and branded HTML:** copying a company's logo, color scheme, and layout to borrow its credibility. This can stack across multiple hops in a single campaign — a fake OneDrive page linking to a fake Adobe page, each one impersonating a different trusted brand to keep the victim moving toward the final credential-harvesting step.

**Artificial urgency:** a deadline, an expiring order, an account on hold — designed to push the recipient into acting before they slow down and check the sender or the link.

**Malicious attachments:** not every phishing email relies on a link. An attachment (PDF, in this case) can carry its own embedded link, so the credential-harvesting step happens once the victim opens the file rather than clicking anything in the email body itself.

## Alert: Cancel Your Order (fake PayPal receipt)

**Details:** Subject "Your Receipt for Payment to Amazing Stuff." Sender displays as `service@paypal.com` but the actual address is a `sultanbogor.com` domain. Recipient is an unrelated Yahoo mail-relay address, not a normal personal inbox. Body impersonates a PayPal gift-card purchase receipt with a "Cancel the Order" button.

![Sender/recipient mismatch on the fake PayPal receipt](../../images/phishing-emails-in-action/Pasted%20image%2020260701141935.png)
![Full receipt body impersonating a PayPal gift-card purchase](../../images/phishing-emails-in-action/Pasted%20image%2020260701141950.png)

**Investigation:** The subject line is written to grab attention immediately. The sender address doesn't match PayPal's real domain space, and the recipient address is a format PayPal wouldn't actually be sending to. Reviewing the message source for the "Cancel the Order" button shows it points to a shortened URL, `https://is.gd/6oCJ4m?#-u7g?ah7Je1`, rather than anything on `paypal.com`.

![Message source showing the shortened URL behind "Cancel the order"](../../images/phishing-emails-in-action/Pasted%20image%2020260701142001.png)

**Verdict:** Phishing — spoofed sender, mismatched recipient, and a shortened link hiding the real destination are enough to call this one with confidence.

## Alert: Track your Package (fake shipping notification with pixel tracking)

**Details:** Subject uses a fake tracking number to create urgency. Display name reads "Distribution Center," but the actual sending address is `contact@beginpro.club`. The body links the tracking number itself as a hyperlink.

![Display name vs. actual sender address on the fake shipping email](../../images/phishing-emails-in-action/Pasted%20image%2020260701143343.png)

**Investigation:** Yahoo had disabled both images and links on this message, which meant hovering over the tracking-number link wasn't an option — the destination had to be pulled from the raw message source instead. That source showed the tracking-number hyperlink actually pointing to a `devret.xyz` domain, completely unrelated to any shipping carrier, and a separate embedded `Tracking.png` image loading from the same domain — a tracking pixel that would notify the attacker's server the moment the email was opened, regardless of whether the link was ever clicked.

![Message source revealing the real link destination and the embedded tracking pixel](../../images/phishing-emails-in-action/Pasted%20image%2020260701143355.png)

**Verdict:** Phishing — spoofed sender, a disguised link destination, and an active tracking pixel confirming attacker visibility into the mailbox.

## Alert: Download Document Here (multi-stage credential-harvesting redirect)

**Details:** Email formatted as a Citrix "you have a fax" notification with a "Download Document Here" button, sent with a tight expiration date to create urgency.

![Fax-notification email with expiration date and download button](../../images/phishing-emails-in-action/Pasted%20image%2020260701145702.png)

**Investigation:** Following the link chain rather than trusting any single page in isolation was the key move here. The first click landed on a page styled to look like OneDrive, but the URL bar showed a domain (`app.poqt.in`) with no connection to Microsoft. That page's "Get Document" button led to a second impersonation page, styled as Adobe Document Cloud, on yet another unrelated domain (`bdkmotorsport.com`) — brand impersonation stacked twice in one chain, each hop borrowing more credibility than the last.

![Fake OneDrive and fake Adobe Document Cloud pages, both on unrelated domains](../../images/phishing-emails-in-action/Pasted%20image%2020260701145715.png)

That Adobe-styled page then presented an email-provider login prompt (Outlook/Office 365/other). Testing it with throwaway credentials returned an "Invalid Credentials" error rather than access to any document — consistent with a form built to capture whatever is typed rather than actually authenticate anything.

![Fake Outlook login prompt rejecting a submitted credential](../../images/phishing-emails-in-action/Pasted%20image%2020260701145726.png)

**Verdict:** Phishing — a multi-stage redirect chain, each step impersonating a different trusted brand, ending in a credential-harvesting form on infrastructure with no connection to any of the companies it claims to represent.

## Alert: Your Account is on Hold (spoofed Netflix, malicious PDF attachment)

**Details:** Subject "Netflix ID Suspended." Display name reads "Netlix billing" (typo, easy to miss at a glance) from `z99@musacombi.online` — no relation to any Netflix domain. Body claims a billing problem and instructs the recipient to open an attached PDF to update payment information.

![Spoofed Netflix sender with a typo'd display name](../../images/phishing-emails-in-action/Pasted%20image%2020260701151004.png)

**Investigation:** Unlike the previous three samples, the malicious payload here isn't a link in the email body — it's the attached PDF. The email itself has several supporting red flags: an irregular, non-standard phone number format, and a genuine `help.netflix.com` link included specifically to build false trust alongside the fraudulent instructions. Opening the attachment shows it carries its own embedded link to a domain with no connection to Netflix, meaning the credential-harvesting step only fires once the victim opens the file rather than clicking anything in the message itself.

![Email body with irregular phone number and the malicious PDF attachment](../../images/phishing-emails-in-action/Pasted%20image%2020260701151014.png)

**Verdict:** Phishing — spoofed sender, brand impersonation, and a malicious link delivered via attachment rather than an in-body link.

## Takeaway

Across all four samples, the same two tactics show up every time: a spoofed sender address and an artificial sense of urgency. What changes is the delivery mechanism — a shortened link, a tracking pixel, a multi-hop redirect chain, or a malicious attachment. None of these require advanced tooling to catch; in every case the verdict came down to checking the actual sender address against the display name, and pulling the message source when a link's true destination wasn't visible at a glance. That's a repeatable process, not a one-off trick, which is exactly what it needs to be against a threat that shows up in every inbox, not just SOC-monitored ones.
