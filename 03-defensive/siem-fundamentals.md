\# SIEM Fundamentals



\## What it is

\*\*SIEM\*\* = Security Information and Event Management. A platform that:

1\. \*\*Collects\*\* logs and events from everywhere (endpoints, servers, network gear, cloud, apps),

2\. \*\*Normalizes\*\* them into a common format,

3\. \*\*Correlates\*\* events across sources to detect threats,

4\. \*\*Alerts\*\* analysts on suspicious patterns,

5\. \*\*Stores\*\* logs for investigation and compliance.



If ATT\&CK is the map of what attackers do, the SIEM is where you \*see\* them

doing it. It's the SOC analyst's home base.



\## The mental model

```mermaid

flowchart LR

&#x20;   subgraph Sources\["Log Sources"]

&#x20;       EP\[Endpoints<br/>Sysmon, EDR]

&#x20;       Net\[Network<br/>Firewall, DNS, IDS]

&#x20;       Cloud\[Cloud<br/>CloudTrail, GCP audit]

&#x20;       Apps\[Apps<br/>Auth, web, DB]

&#x20;   end

&#x20;   Sources --> Ingest\[Ingestion<br/>Agents, syslog, APIs]

&#x20;   Ingest --> Parse\[Parse \& Normalize<br/>Common schema]

&#x20;   Parse --> Store\[(Store \& Index)]

&#x20;   Store --> Detect\[Detection Rules<br/>Correlation]

&#x20;   Detect --> Alert\[Alert]

&#x20;   Alert --> Analyst\[Analyst / SOAR]

&#x20;   Store --> Hunt\[Threat Hunting<br/>Ad-hoc queries]

```



\## The pipeline in detail



\*\*1. Collection.\*\* Agents on hosts (Sysmon, osquery, Elastic Agent, Splunk UF), syslog from network gear, API pulls from cloud (CloudTrail, Entra ID logs), webhooks from SaaS.



\*\*2. Parsing / normalization.\*\* A raw log like `Aug 12 10:23:45 host1 sshd\[1234]: Failed password for root from 1.2.3.4` becomes structured fields: `event.action=login\_failure, user.name=root, source.ip=1.2.3.4, service=sshd`. Common schemas: \*\*ECS\*\* (Elastic Common Schema), \*\*OCSF\*\* (Open Cybersecurity Schema Framework), Splunk CIM.



\*\*3. Enrichment.\*\* Add context: threat-intel lookups on IPs, GeoIP, asset info, user role, past behavior. A raw IP becomes "known Tor exit node, unusual for this user."



\*\*4. Storage.\*\* Hot (fast, expensive) for recent data; warm/cold (cheap object storage) for older data. Retention driven by compliance (PCI = 1 year, some regs longer).



\*\*5. Detection.\*\* Rules run over incoming data. Two families:

\- \*\*Signature / rule-based\*\* — "SSH failed login from same IP, ≥10 times in 60s."

\- \*\*Anomaly / behavior-based\*\* — "This user usually logs in from Nairobi at 9am, now signing in from Moscow at 3am." UEBA (User and Entity Behavior Analytics) sits here.



\*\*6. Alert triage.\*\* Analyst reviews, gathers context, decides: \*\*true positive\*\* (real attack), \*\*false positive\*\* (benign), \*\*benign true positive\*\* (real activity, not malicious). Good detections have low false-positive rates.



\*\*7. Response.\*\* Manual (analyst) or automated via \*\*SOAR\*\* (Security Orchestration, Automation, and Response) — playbooks like "isolate host, disable user, open ticket."



\## Log sources that matter most

Prioritize these when standing up a SIEM — most attacks show up here first:



\- \*\*Authentication\*\* — Windows Security 4624/4625, Linux `auth.log`, cloud IdP (Entra, Okta, Cognito).

\- \*\*Endpoint process activity\*\* — Sysmon (Windows), auditd (Linux), EDR telemetry.

\- \*\*DNS\*\* — attackers love DNS for C2 and data exfil. Unusually long domains, huge query volume, weird TLDs.

\- \*\*Web proxy / firewall\*\* — outbound connections. C2 lives here.

\- \*\*Cloud audit\*\* — AWS CloudTrail, GCP Audit Logs, Azure Activity. IAM changes especially.

\- \*\*PowerShell / shell\*\* — script block logging on Windows, bash history + auditd on Linux.

\- \*\*VPN\*\* — logins from impossible locations, off-hours access.



Common trap: logging everything and detecting nothing. Better to log the

right sources well than everything badly.



\## SIEM vs SOAR vs XDR vs EDR (interview question)

| | What it does |

|---|---|

| \*\*EDR\*\* | Endpoint Detection \& Response — deep telemetry + response on endpoints (CrowdStrike, SentinelOne, Defender for Endpoint). |

| \*\*SIEM\*\* | Aggregates + correlates logs across the whole environment. Log-centric. |

| \*\*XDR\*\* | Extended Detection \& Response — vendor-integrated stack (endpoint + email + network + cloud) with correlation baked in. Overlap with SIEM. |

| \*\*SOAR\*\* | Automates the \*response\* — playbooks, ticket integration, host isolation, user disable. Sits on top of SIEM. |

| \*\*UEBA\*\* | Behavior analytics — anomalies for users and entities. Usually a SIEM feature. |



Modern reality: lines blur. Splunk + Phantom (SOAR). Elastic bundles SIEM + EDR + SOAR. Sentinel has Logic Apps for SOAR. Vendors sell it all as "one platform."



\## The major players

\- \*\*Commercial:\*\* Splunk, Microsoft Sentinel, IBM QRadar, Google Chronicle, Exabeam, Sumo Logic.

\- \*\*Open-source / free tier:\*\* \*\*Wazuh\*\* (great for home lab), \*\*Elastic Security\*\* (SIEM built on the ELK stack), \*\*Graylog\*\*, \*\*SecurityOnion\*\* (SIEM + IDS + full network monitoring, prebuilt).



For learning: \*\*Wazuh\*\* or \*\*Security Onion\*\* in a VM. Free, real, industry-recognized.



\## Detection rules — the vocabulary

\- \*\*Sigma\*\* — vendor-neutral detection rule format (YAML). Write once, convert to Splunk SPL, Elastic KQL, Sentinel KQL, etc. `git clone` the SigmaHQ repo — thousands of rules.

\- \*\*YARA\*\* — pattern matching for files/memory. Malware analysis.

\- \*\*Snort / Suricata\*\* — network IDS rules. Match packet patterns.

\- \*\*KQL\*\* — Kusto Query Language (Sentinel, Defender).

\- \*\*SPL\*\* — Search Processing Language (Splunk).

\- \*\*EQL\*\* — Event Query Language (Elastic, ATT\&CK-friendly sequences).



Every good detection is tagged with the \*\*ATT\&CK technique(s)\*\* it covers.

That's how you map detection coverage to threat coverage.



\## Example: a SSH brute-force detection (Sigma)

```yaml

title: SSH Brute Force

id: 3d3f6e1a-6c4a-4e9a-9c1f-abc

status: experimental

description: Detects many failed SSH logins from a single source IP.

logsource:

&#x20; product: linux

&#x20; service: sshd

detection:

&#x20; selection:

&#x20;   message|contains: 'Failed password'

&#x20; timeframe: 60s

&#x20; condition: selection | count() by source\_ip > 10

falsepositives:

&#x20; - Legit user typing password many times

level: medium

tags:

&#x20; - attack.credential\_access

&#x20; - attack.t1110.001

```

That last block — `attack.t1110.001` — is why yesterday's Nmap/OWASP + ATT\&CK notes matter. Detections \*speak\* ATT\&CK.



\## The SOC analyst workflow (mental model)

```mermaid

sequenceDiagram

&#x20;   SIEM->>Analyst: Alert fires

&#x20;   Analyst->>SIEM: Pivot on user, host, IP

&#x20;   Analyst->>EDR: Check endpoint process tree

&#x20;   Analyst->>ThreatIntel: Reputation lookup on IP/hash

&#x20;   Analyst->>Analyst: Decide: TP / FP / BTP

&#x20;   alt True Positive

&#x20;       Analyst->>SOAR: Trigger playbook

&#x20;       SOAR->>Endpoint: Isolate host

&#x20;       SOAR->>IdP: Disable user

&#x20;       Analyst->>Ticketing: Open incident

&#x20;   else False Positive

&#x20;       Analyst->>SIEM: Tune rule

&#x20;   end

```



\## Common gotchas

\- \*\*Alert fatigue is the #1 SOC problem.\*\* 200 alerts/day, most are false positives, real attacks get lost. Tune ruthlessly.

\- \*\*"Ingesting the log" ≠ "detecting on it."\*\* You have to write rules. Vendors show impressive dashboards, but detections don't build themselves.

\- \*\*Time sync (NTP) matters.\*\* Correlating events across sources fails if clocks drift.

\- \*\*Log integrity.\*\* If attackers can wipe your logs, they will. Forward to append-only storage.

\- \*\*Compliance ≠ security.\*\* PCI/SOX/HIPAA drive log \*collection\* but rarely mandate good \*detection\*.

\- \*\*PII in logs.\*\* Auth logs, HTTP logs, and stack traces leak personal data. GDPR/POPIA/CCPA apply. Redact.

\- \*\*Storage costs balloon fast.\*\* Splunk's licensing (per GB/day) is famously expensive. Tiered storage and log filtering are essential.

\- \*\*The most-tuned rule in the world can't fix bad log coverage.\*\* Fix collection before you fix detection.



\## What I want to try

\- Deploy \*\*Wazuh\*\* in a VM, install its agent on my laptop, watch real events flow in.

\- Spin up \*\*Security Onion\*\* in VirtualBox and generate traffic against a \*\*Metasploitable\*\* VM — see what the SIEM catches.

\- Write my first \*\*Sigma\*\* rule from scratch (e.g. "detect new local admin created") and convert it to KQL.

\- Take a public ATT\&CK-mapped threat report and map it to detections I'd write.

\- Explore \*\*Microsoft Sentinel's free tier\*\* with a demo tenant, run KQL over the sample logs.

\- Read the "Detection Engineering" chapter of Splunk's docs — free, and better than most books.



\## Sources

\- \[Wazuh docs](https://documentation.wazuh.com/) — the best free SIEM to learn on

\- \[SigmaHQ rule repo](https://github.com/SigmaHQ/sigma)

\- \[MITRE ATT\&CK](https://attack.mitre.org/) — already covered in `05-frameworks/mitre-attack.md`

\- "Blue Team Handbook: SOC, SIEM, and Threat Hunting" — Don Murdoch

\- SANS SEC450 / SEC555 course syllabi (topics only, for the pro track)

