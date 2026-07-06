# Snapped Phish-ing Line

**Source:** TryHackMe — SOC Level 1 path
**Date:** 2026-07-06

## Overview

Multiple employees at SwiftSpend Financial reported a suspicious email, and some had already submitted their credentials. This is a capstone practical tying together the last several phishing-analysis rooms — the task is to trace the campaign from the initial email all the way to the attacker's own infrastructure: the redirect chain, the fake login page, and — because this particular attacker left it exposed — the phishing kit itself.

## Key Concepts

**Open-directory exposure on attacker infrastructure:** phishing kits are usually deployed by just uploading a set of files to a compromised or rented web server, and that server's directory listing isn't always locked down. When it isn't, the entire kit — source code, credential logs, everything — is sitting there crawlable by anyone who checks. That's an attacker mistake, not something to expect on every case.

**Hashing and threat-intel lookup on a phishing kit itself:** a phishing kit is usually distributed as a packaged archive (ZIP, in this case), and running the standard malware-triage steps — `sha256sum` the file, check the hash against VirusTotal — works exactly the same way against a phishing kit as it does against any other malicious file. It came back flagged by 33 of 65 vendors under threat categories `trojan`, `phishing`, and `hacktool`, with a popular threat label of `trojan.phishmailer/phishingms` — meaning this exact kit is a known, previously-catalogued piece of commodity tooling, not something custom-built for this one campaign.

**Reading a kit's backend code to find the exfiltration destination:** the credential-harvesting form itself is only half the kit — the PHP file that actually processes the submission (`$_POST['email']`, `$_POST['password']`) is what shows where the stolen data actually goes. In this case that file builds a formatted message (captured email, password, IP, browser user-agent, GeoIP-derived country) and sends it to a hardcoded address, `m3npat@yandex.com` — the real collection point behind the campaign, as opposed to anything visible from the phishing page itself.

**Kit branding as an attribution signal:** the credential log and kit code both carry the tag "Created BY Real Carder" — a phishing-kit builder brand that shows up repeatedly in leaked/resold criminal kits. Recognizing a kit's branding is a fast way to tell "mass-produced, resold tool" from "bespoke campaign built for this target," which changes what kind of actor you're likely dealing with.

**Real-world scope judgment:** digging this deep into an attacker's infrastructure is not standard Tier 1 practice at an MSSP. Once a case is confirmed as credential-harvesting phishing, the response is to block the domain and force a password reset for anyone who submitted — not to go excavate the attacker's kit, credential log, and backend code. It's worth checking whether an attacker left something this exposed, since it's cheap to glance at, but building a habit of going this deep on every phishing ticket doesn't scale and isn't the L1 analyst's job.

## Investigation

The first email, sent to William McClean, impersonated "Group Marketing Online" with a fake "Quote for Services Rendered" — generic vendor-invoice pretext, sent from `Accounts.Payable@groupmarketingonline.icu`:

![Phishing email impersonating Group Marketing Online, sent to William McClean](../../images/snapped-phishing-line/Pasted%20image%2020260706150339.png)

The same sender targeted other employees with a different pretext. The email to Zoe Duncan claimed to be a "Direct Credit Advice" and carried an HTML attachment rather than an in-body link:

![Phishing email to Zoe Duncan with a "Direct Credit Advice" HTML attachment](../../images/snapped-phishing-line/Pasted%20image%2020260706150535.png)

Reading the attachment's source rather than just opening it showed it wasn't a document at all — it was a meta-refresh redirect, hardcoded to immediately forward the victim to an external URL with their email address already appended as a query parameter:

![Terminal output of the HTML attachment showing a meta-refresh redirect to kennaroads.buzz](../../images/snapped-phishing-line/Pasted%20image%2020260706150821.png)

The redirect's root domain was `kennaroads.buzz`. Following it in an isolated VM browser landed on a fake Microsoft login page, pre-populated with the victim's real email address to make the prompt look personalized and legitimate:

![Fake Microsoft/Office365 login page pre-filled with the victim's email](../../images/snapped-phishing-line/Pasted%20image%2020260706150922.png)

Rather than stopping at the fake login page, checking whether the same site exposed anything else at its root turned up an open directory listing at `kennaroads.buzz/data/` — the attacker had left Apache's default directory browsing enabled, exposing the entire phishing kit as a downloadable archive:

![Open directory listing on kennaroads.buzz exposing the phishing kit archive](../../images/snapped-phishing-line/Pasted%20image%2020260706151011.png)

Downloading and hashing that archive (`sha256sum`) and checking the hash against VirusTotal confirmed it as a known, previously-seen phishing kit — 33 of 65 vendors flagged it malicious, categorized under `trojan`, `phishing`, and `hacktool`, with a popular threat label of `trojan.phishmailer/phishingms`:

![VirusTotal detection results showing 33/65 vendors flagging the kit as malicious](../../images/snapped-phishing-line/Pasted%20image%2020260706151141.png)

The details tab showed the archive contained 49 files — a mix of PHP, images, and supporting directories — consistent with a full, functional phishing-kit deployment rather than a single standalone page:

![VirusTotal details tab showing the archive's 49 contained files](../../images/snapped-phishing-line/Pasted%20image%2020260706151236.png)

The same open directory also exposed the kit's live credential-harvesting log, showing real captured Office365 logins in plaintext — including one employee, `michael.ascot@swiftspend.finance`, who had submitted credentials twice:

![Exposed credential-harvesting log showing captured Office365 logins](../../images/snapped-phishing-line/Pasted%20image%2020260706151337.png)

Extracting the kit and reading its backend validation script showed exactly what happens to a submitted credential: it's assembled into a formatted message along with the victim's IP, browser user-agent, and GeoIP-resolved country, then emailed straight to the attacker's collection address, `m3npat@yandex.com`:

![Phishing kit's PHP backend showing the credential message format and the attacker's collection email](../../images/snapped-phishing-line/Pasted%20image%2020260706151534.png)

The same directory also held a `flag.txt`, containing a Base64-encoded, reversed string:

![flag.txt containing an encoded secret value](../../images/snapped-phishing-line/Pasted%20image%2020260706151808.png)

Running it through CyberChef with a From-Base64 step followed by a character-reverse recovered the plaintext flag:

![CyberChef recipe decoding the flag to plaintext](../../images/snapped-phishing-line/Pasted%20image%2020260706152005.png)

## Verdict

Confirmed credential-harvesting phishing campaign. Multiple SwiftSpend employees were targeted with vendor/finance-themed pretexts leading to a fake Office365 login page; several had already submitted real credentials before the attack was escalated to the SOC. The attacker's own infrastructure — kit archive, credential log, and backend collection script — was independently exposed and confirmed the same conclusion the front-end investigation had already reached.

## Takeaway

The email and redirect chain alone were enough to call this phishing with confidence — mismatched sender domain, a redirect hidden inside an attachment rather than a visible link, and a fake login page pre-filled with the victim's own email to look personalized. Digging into the attacker's exposed `/data` directory was worth the two minutes it took to check, but it isn't something to build into a standard L1 workflow: once credential-harvesting phishing is confirmed, the actual response is blocking the domain and forcing a password reset for anyone who submitted, not reverse-engineering the attacker's kit. That's useful depth for a DFIR-flavored investigation, but it's more thoroughness than a Tier 1 SOC ticket calls for.
