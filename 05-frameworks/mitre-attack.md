\# MITRE ATT\&CK



\## What it is

\*\*MITRE ATT\&CK\*\* (Adversarial Tactics, Techniques, and Common Knowledge) is a

free, curated knowledge base of how real-world attackers behave — mapped from

public incident reports, threat intel, and red-team research.



It answers the question: \*"What do attackers actually do, in what order, and

how do we detect and stop each step?"\* Every SOC, threat intel team, red team,

and detection engineer references it constantly. When someone says "TTPs"

(Tactics, Techniques, Procedures), they're speaking ATT\&CK.



\## The mental model

```mermaid

flowchart LR

&#x20;   Matrix\[ATT\&CK Matrix] --> Tactic\[Tactic: the WHY]

&#x20;   Tactic --> Technique\[Technique: the HOW]

&#x20;   Technique --> SubT\[Sub-technique: specific variant]

&#x20;   Technique --> Procedure\[Procedure: real observed use]

```



\- \*\*Tactic\*\* = the attacker's goal at that step (e.g. "get initial access").

\- \*\*Technique\*\* = a general method to achieve it (e.g. "phishing").

\- \*\*Sub-technique\*\* = a specific variant (e.g. "spearphishing attachment").

\- \*\*Procedure\*\* = how a specific group actually did it (e.g. "APT29 sent a Word doc with a malicious macro on X date").



\## The 14 Enterprise tactics (in typical order)

```mermaid

flowchart LR

&#x20;   R\[Reconnaissance] --> RD\[Resource Development]

&#x20;   RD --> IA\[Initial Access]

&#x20;   IA --> E\[Execution]

&#x20;   E --> P\[Persistence]

&#x20;   P --> PE\[Privilege Escalation]

&#x20;   PE --> DE\[Defense Evasion]

&#x20;   DE --> CA\[Credential Access]

&#x20;   CA --> D\[Discovery]

&#x20;   D --> LM\[Lateral Movement]

&#x20;   LM --> C\[Collection]

&#x20;   C --> CC\[Command \& Control]

&#x20;   CC --> EX\[Exfiltration]

&#x20;   EX --> I\[Impact]

```



1\. \*\*Reconnaissance\*\* — gather info about the target (before touching it).

2\. \*\*Resource Development\*\* — set up infrastructure (register domains, buy accounts).

3\. \*\*Initial Access\*\* — get a foothold (phishing, valid accounts, exploit public app).

4\. \*\*Execution\*\* — run adversary-controlled code (PowerShell, scripting).

5\. \*\*Persistence\*\* — survive reboots and credential resets (scheduled tasks, run keys, service creation).

6\. \*\*Privilege Escalation\*\* — gain higher permissions (kernel exploit, token manipulation, sudo abuse).

7\. \*\*Defense Evasion\*\* — avoid detection (disable logging, obfuscate, use signed binaries — "LOLBins").

8\. \*\*Credential Access\*\* — steal creds (dumping LSASS, keyloggers, Kerberoasting).

9\. \*\*Discovery\*\* — map the environment from inside (`net user`, `whoami`, network scan).

10\. \*\*Lateral Movement\*\* — move to other hosts (RDP, SSH, PsExec, WMI).

11\. \*\*Collection\*\* — gather target data (screenshots, browser data, files).

12\. \*\*Command \& Control (C2)\*\* — communicate with attacker infrastructure (HTTPS, DNS, Slack).

13\. \*\*Exfiltration\*\* — get data out (over C2, to cloud storage, DNS tunneling).

14\. \*\*Impact\*\* — the endgame (ransomware, wipers, defacement, DoS).



Not every attack hits every tactic. Not always in this order. But the sequence

is a solid mental map.



\## Real example — a phishing-to-ransomware chain, mapped

| Step | Tactic | Technique |

|---|---|---|

| User clicks malicious link | Initial Access | T1566.002 Spearphishing Link |

| Malicious doc runs macro | Execution | T1204.002 Malicious File |

| Scheduled task added | Persistence | T1053.005 Scheduled Task |

| UAC bypass | Privilege Escalation | T1548.002 Bypass UAC |

| Disable Windows Defender | Defense Evasion | T1562.001 Disable Tools |

| Dump LSASS | Credential Access | T1003.001 LSASS Memory |

| Scan the network | Discovery | T1046 Network Service Discovery |

| Spread via SMB | Lateral Movement | T1021.002 SMB/Admin Shares |

| Beacon over HTTPS | C2 | T1071.001 Web Protocols |

| Encrypt files | Impact | T1486 Data Encrypted for Impact |



Every real incident report can be flattened into a chain like this. That

flattening is the whole point of ATT\&CK.



\## The matrices

\- \*\*Enterprise\*\* — Windows, Linux, macOS, cloud (AWS/Azure/GCP/SaaS), containers, network.

\- \*\*Mobile\*\* — iOS, Android.

\- \*\*ICS\*\* — Industrial Control Systems (SCADA, PLCs). Different tactics because impact means physical damage.



Most people mean \*Enterprise\* when they say "ATT\&CK."



\## What to actually do with it



\### Blue team uses

\- \*\*Detection engineering\*\* — write detections mapped to techniques. Every rule tagged with `T####`.

\- \*\*Coverage mapping\*\* — a heatmap of which techniques you can detect vs which you can't.

\- \*\*Threat hunting\*\* — pick a technique with no coverage, go hunt for it.

\- \*\*Purple teaming\*\* — red emulates a technique, blue tries to detect, gaps get filled.



\### Red team uses

\- \*\*Adversary emulation\*\* — mimic a real group's TTPs (e.g. "emulate APT29") instead of random pentesting.

\- \*\*Reporting\*\* — findings mapped to ATT\&CK make results actionable for the blue team.



\### Threat intel uses

\- \*\*Group profiles\*\* — each named actor (APT28, Lazarus, FIN7) has a page listing techniques they've used.

\- \*\*Vocabulary\*\* — everyone in the room means the same thing when they say "T1078."



\## Related MITRE projects

\- \*\*D3FEND\*\* — defensive counterpart. Maps defensive countermeasures to ATT\&CK techniques.

\- \*\*CAR (Cyber Analytics Repository)\*\* — reference detection analytics.

\- \*\*ATT\&CK Navigator\*\* — free web tool to build coverage heatmaps, layer sub-techniques, plan hunts. Learn this one.

\- \*\*Caldera\*\* — MITRE's open-source adversary emulation platform.

\- \*\*Atomic Red Team\*\* — small, focused scripts that execute individual techniques for testing detections (by Red Canary, not MITRE, but universally used alongside).



\## Common gotchas

\- \*\*ATT\&CK is descriptive, not prescriptive.\*\* It tells you what attackers do; it doesn't tell you what to \*do\* about it. That's D3FEND / your own engineering.

\- \*\*Coverage ≠ security.\*\* "We detect 200 techniques" is meaningless without knowing which matter for your environment.

\- \*\*Not every technique is equally relevant.\*\* A SaaS shop doesn't care about ICS techniques. Prioritize by threat model.

\- \*\*Sub-techniques were added in 2020.\*\* Old material references numeric IDs without them. Convert when you see one.

\- \*\*ATT\&CK vs Kill Chain vs Cyber Kill Chain (Lockheed).\*\* Kill Chain is 7 higher-level phases; ATT\&CK is more granular and non-linear. Both are useful. They're not competitors.

\- \*\*"Cyber Kill Chain" is Lockheed's trademarked term.\*\* ATT\&CK doesn't use it.



\## What I want to try

\- Open \*\*ATT\&CK Navigator\*\* and build a coverage layer for a hypothetical small business.

\- Pick one technique (e.g. T1078 Valid Accounts) and write a detection idea + a log source needed.

\- Run one \*\*Atomic Red Team\*\* test on my own machine (safe, self-contained) and see what it looks like in the logs.

\- Read a threat report (e.g. Mandiant on APT29) and map its findings to ATT\&CK techniques myself.

\- Deploy Wazuh or Elastic SIEM in a home lab and enable ATT\&CK-tagged rules.



\## Sources

\- \[attack.mitre.org](https://attack.mitre.org/)

\- \[MITRE ATT\&CK Navigator](https://mitre-attack.github.io/attack-navigator/)

\- \[Atomic Red Team](https://atomicredteam.io/)

\- "MITRE ATT\&CK: Design and Philosophy" (whitepaper, free)

\- SANS FOR578 course topics (for the pro track)

