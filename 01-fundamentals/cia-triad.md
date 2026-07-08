\# The CIA Triad



\## What it is

The three foundational goals of information security: \*\*Confidentiality,

Integrity, Availability.\*\* Every security control ultimately protects one

or more of these. If you can name the triad \*and\* explain the tradeoffs

between them, you're already ahead of most entry-level candidates.



\## The mental model

```mermaid

flowchart TB

&#x20;   Sec\[Information Security]

&#x20;   Sec --> C\[Confidentiality]

&#x20;   Sec --> I\[Integrity]

&#x20;   Sec --> A\[Availability]

&#x20;   C --> C1\[Only authorized parties<br/>can read data]

&#x20;   I --> I1\[Data is accurate and<br/>not tampered with]

&#x20;   A --> A1\[Data and systems are<br/>accessible when needed]

```



\## The three goals

\- \*\*Confidentiality\*\* — only authorized parties can access the data.

&#x20; Controls: encryption (at rest and in transit), access control, MFA, network segmentation.

\- \*\*Integrity\*\* — data hasn't been altered by unauthorized parties, accidentally or on purpose.

&#x20; Controls: hashing (SHA-256), digital signatures, checksums, version control, immutable logs.

\- \*\*Availability\*\* — systems and data are accessible when needed by authorized users.

&#x20; Controls: redundancy, backups, DDoS protection, load balancing, disaster recovery.



\## Attacks that break each one

\- \*\*C\*\* — data breach, sniffing, shoulder surfing, weak encryption, misconfigured S3 bucket.

\- \*\*I\*\* — MITM tampering, unauthorized DB writes, ransomware encrypting files, log tampering.

\- \*\*A\*\* — DDoS, ransomware locking systems, hardware failure, misconfigurations that crash prod.



Ransomware is famous for hitting \*\*all three\*\* at once.



\## The tradeoffs (this is the interview question)

The three goals \*\*conflict\*\* with each other, and choosing which to prioritize

is a real engineering decision:



\- Encrypting a database (C↑) makes recovery harder if you lose the key (A↓).

\- Read-only immutable logs (I↑) mean you can't fix a bad entry (A↓).

\- Aggressive backups (A↑) create more copies of sensitive data to secure (C↓).

\- Locking accounts after 3 failed logins (C↑) enables trivial account-lockout DoS (A↓).



The right answer is almost never "maximize all three." It's "understand the

business, pick which one dominates, mitigate the others."



\## Extended models

Some frameworks extend the triad:

\- \*\*AAA\*\* — Authentication, Authorization, Accounting

\- \*\*DAD\*\* — Disclosure, Alteration, Destruction (the \*attack\* triad, mirror of CIA)

\- \*\*Parkerian Hexad\*\* — CIA + Authenticity, Possession/Control, Utility



CIA is by far the most commonly referenced. Know the others exist.



\## Common gotchas

\- \*\*Confidentiality ≠ privacy.\*\* Confidentiality is a control; privacy is a right/policy.

\- \*\*Integrity ≠ correctness.\*\* Integrity means \*unaltered\*; the original data can still be wrong.

\- \*\*Availability isn't just uptime\*\* — it's \*authorized\* users getting \*timely\* access. A slow system can violate availability.

\- Ransomware = C + I + A hit at once. Not just an availability issue.



\## What I want to try

\- Pick a system I know (e.g. a GitHub repo, a home router) and map its controls to CIA.

\- Design a small system where two of the three goals are in real tension, and explain the tradeoff I'd make.

\- Read the write-up of a real breach and classify what failed.



\## Sources

\- NIST SP 800-33 (foundational security concepts)

\- "Security Engineering" — Ross Anderson (free online)

\- OWASP glossary

