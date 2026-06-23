# Pyramid of Pain
**Source:** TryHackMe — SOC Level 1 Path, Cyber Defense Frameworks **Difficulty:** Easy

## Key Concepts
### Hash Values (Trivial)
A hash value is the output of running a file through a hashing algorithm — a fixed-length alphanumeric string that uniquely identifies that exact data. Analysts can run a file's hash through online lookup tools to check its reputation. The catch is that hashing is brittle by design: appending even a single byte to a known-malicious file (as simple as `echo`-ing a string onto the end of it) produces a completely different hash, instantly defeating any hash-based detection.

In practice, hash-blocking is generally implemented through threat intel feeds and curated "bad hash" lists. It's not a catch-all — any attacker who bothers to modify their payload slips past it — but it's still worth doing, because it reliably filters out the lazy or unsophisticated attackers reusing known, unmodified tooling.

### IP Address (Easy)
An IP address identifies a device on a network. When a malicious IP turns up in an investigation, blocking it at the firewall is usually trivial. Attackers get around this with techniques like fast flux — using a rotating pool of previously compromised hosts as proxies, so the apparent source IP keeps changing and the real origin stays hidden.

In an L1 seat, the practical reality is that you rarely get deep enough into an investigation to definitively distinguish fast flux from ordinary IP churn (a host moving between addresses for entirely legitimate reasons). What matters at that tier isn't pinning down which one it is — it's recognizing the pattern and flagging it as something worth escalating or watching, rather than trying to resolve the distinction yourself.

### Domain Names (Simple)
A domain name maps an IP to a readable string. Domains have to be registered, but many DNS providers have loose verification standards and expose APIs that make registering and rotating new domains easy. Attackers also disguise domains to look legitimate — punycode lets a domain like `adıdas.de` visually pass for `adidas.de` while actually resolving to `http://xn--addas-o4a.de/`, and standard URL shorteners (`bit.ly`, `tinyurl.com`, `goo.gl`) obscure the real destination of a link entirely.

This isn't a theoretical edge case — it's a constant in phishing, which makes up a large share of what an L1 analyst actually triages. Punycode lookalikes and shortened links both show up regularly in real alert queues, not just in training rooms.

### Host Artifacts (Annoying)
These are the traces an attack leaves on a system: registry values, suspicious process execution, recognizable attack patterns, or files dropped by malicious tooling. Changing these forces an attacker to rework their actual tradecraft, not just swap a file or IP — which is why they sit higher on the pyramid.

For an L1 analyst, host artifacts are primarily how you reconstruct what actually happened on a machine — they're the evidence trail used to identify the attacker's TTPs and piece together what was done, not just confirm that something happened.

### Network Artifacts (Annoying)
The network-side equivalent: user-agent strings, C2 communication patterns, and the URI structure of HTTP POST requests tied to the attack. Like host artifacts, changing these means changing the fundamentals of how the attack communicates, making them similarly costly for an attacker to alter.

The value to an L1 analyst is the same as host artifacts — these are used to build out the picture of what occurred and tie activity back to known TTPs, rather than being an end in themselves.

### Tools (Challenging)
At this tier, simply changing a file or IP isn't enough — the attacker has to build new tooling, find better tooling, or get measurably more proficient with what they're already using. If a defender can detect and block the tool itself, the attacker is stopped outright.

From an L1 seat, this tier mostly isn't visible in the moment. You're not generally in a position to recognize "this is the same actor as last time, now using a different tool" — every alert that lands in the queue gets investigated and resolved on its own merits. Attribution to a specific actor or campaign isn't the job at this stage; the alert is the alert regardless of who's behind it.

### TTPs (Tough)
Tactics, Techniques, and Procedures cover the full lifecycle of an attack, typically organized through the MITRE ATT&CK Matrix. Detecting and responding at the TTP level leaves an attacker with essentially nowhere left to go — short of rebuilding their entire approach from scratch.

The real value of TTPs and MITRE ATT&CK isn't just technical categorization — it's a shared language. Because MITRE is written in a way almost anyone can follow, it becomes the most useful framework for communicating what happened during an incident to non-technical stakeholders or executives, not just to other analysts.
