\# OWASP Top 10 (2021)



\## What it is

The OWASP Top 10 is a consensus list of the most critical web application

security risks, published by the Open Worldwide Application Security Project.

It's updated every 3–4 years based on real-world data from thousands of apps.

The 2021 edition is the current version; 2025 is expected soon.



Not a checklist — a \*\*starting point for threat modeling.\*\* Real apps have

risks that don't make the list.



\## The mental model

```mermaid

flowchart LR

&#x20;   Design\[A04: Insecure Design] --> Build

&#x20;   Build\[Code + Config] --> A01\[A01: Broken Access Control]

&#x20;   Build --> A02\[A02: Crypto Failures]

&#x20;   Build --> A03\[A03: Injection]

&#x20;   Build --> A07\[A07: AuthN Failures]

&#x20;   Build --> A08\[A08: Data Integrity]

&#x20;   Deploy\[Deploy] --> A05\[A05: Misconfiguration]

&#x20;   Deploy --> A06\[A06: Vulnerable Components]

&#x20;   Runtime\[Runtime] --> A09\[A09: Logging Failures]

&#x20;   Runtime --> A10\[A10: SSRF]

```



\## The list



\### A01 — Broken Access Control

Users doing things they shouldn't be allowed to. The #1 risk since 2021.

\- \*\*Examples:\*\* IDOR (`/api/user/123` when you're user 456), path traversal, missing role checks, tampering JWTs to elevate privileges.

\- \*\*Defense:\*\* deny by default, enforce authorization server-side on \*every\* request, log access failures.

\- \*\*Test for it:\*\* open two accounts, try to access account B's resources from account A.



\### A02 — Cryptographic Failures

Data protected weakly or not at all. Formerly "Sensitive Data Exposure."

\- \*\*Examples:\*\* plaintext passwords, HTTP for sensitive pages, MD5/SHA-1 for passwords, hardcoded keys, weak TLS.

\- \*\*Defense:\*\* TLS everywhere, bcrypt/argon2 for passwords, AES-256-GCM for data at rest, rotate keys.

\- \*\*Rule:\*\* never invent your own crypto. Use libraries.



\### A03 — Injection

Untrusted input interpreted as code. Includes SQLi, NoSQLi, LDAPi, command injection, ORM injection.

\- \*\*Defense:\*\* parameterized queries (prepared statements), input validation, output encoding, safe APIs (`subprocess` with list args, not `shell=True`).

\- \*\*XSS\*\* now lives under Injection (it was A07 in 2017).



\### A04 — Insecure Design

Flaws in the \*design\* itself, not just the implementation. Introduced in 2021.

\- \*\*Examples:\*\* password reset that emails the actual password, business logic that lets you refund yourself, unlimited login attempts.

\- \*\*Defense:\*\* threat modeling before you build, secure design patterns, misuse cases in requirements.

\- Can't be fixed with a code patch — you have to redesign.



\### A05 — Security Misconfiguration

Everything is configurable and most defaults are insecure.

\- \*\*Examples:\*\* default admin creds, S3 buckets public, verbose error messages, unnecessary services enabled, missing security headers.

\- \*\*Defense:\*\* hardened base images, config as code, automated scanning (Checkov, tfsec), least-privilege by default.



\### A06 — Vulnerable and Outdated Components

You're using a library with a known CVE.

\- \*\*Examples:\*\* Log4Shell, Struts (Equifax), old jQuery with known XSS.

\- \*\*Defense:\*\* SBOM (software bill of materials), Dependabot / Renovate, Trivy/Snyk in CI, remove unused deps.

\- Reality check: your app is mostly other people's code. Patch it.



\### A07 — Identification and Authentication Failures

Broken login and session management.

\- \*\*Examples:\*\* no rate limiting on login, weak password policy, session IDs in URLs, no logout, permitting known-compromised passwords.

\- \*\*Defense:\*\* MFA everywhere, strong session management (rotate on privilege change), check passwords against HaveIBeenPwned, lockouts with CAPTCHA (not just "3 tries then locked").



\### A08 — Software and Data Integrity Failures

Trusting things you shouldn't. Includes insecure deserialization and supply-chain attacks.

\- \*\*Examples:\*\* loading plugins from untrusted sources, CI/CD pipeline compromise (SolarWinds), deserializing user-controlled data, npm typosquatting.

\- \*\*Defense:\*\* signed artifacts (Sigstore, cosign), verified plugin repositories, pinned dependency hashes, protected CI/CD secrets.



\### A09 — Security Logging and Monitoring Failures

You got breached and didn't notice for 6 months.

\- \*\*Examples:\*\* no logs on auth events, no alerting on anomalies, logs stored where the attacker can wipe them.

\- \*\*Defense:\*\* centralized logging (SIEM), immutable/append-only storage, alert on auth failures + privilege changes + high-value actions, run tabletop exercises.

\- Industry stat: mean time to detect a breach is \*hundreds\* of days.



\### A10 — Server-Side Request Forgery (SSRF)

Attacker makes the \*server\* fetch a URL of their choosing.

\- \*\*Why it matters:\*\* the server is often inside a trusted network. Requests to `http://169.254.169.254/` (AWS metadata) can leak credentials. Capital One breach, 2019.

\- \*\*Defense:\*\* allowlist outbound destinations, block private IP ranges, disable HTTP redirects, IMDSv2 on AWS.



\## What changed from 2017

\- New: Insecure Design (A04), Software/Data Integrity Failures (A08), SSRF (A10)

\- Merged: XSS folded into Injection

\- Renamed: "Sensitive Data Exposure" → "Cryptographic Failures" (focus on cause, not symptom)

\- Broken Access Control moved from #5 to #1



\## Common gotchas

\- \*\*The Top 10 is not a compliance checklist.\*\* Passing it doesn't mean you're secure.

\- \*\*Order doesn't strictly mean severity in your app\*\* — SSRF might be trivial in a static site, catastrophic in a cloud API.

\- \*\*Categories overlap.\*\* A misconfigured S3 bucket is A01 (access) + A02 (crypto if unencrypted) + A05 (misconfig). Real incidents rarely fit one box.

\- \*\*Injection isn't just SQL.\*\* Log injection, XML injection, prompt injection in LLM apps — all the same family.

\- \*\*"Insecure Design" is often the root cause\*\* of the more visible issues below it.



\## What I want to try

\- Deploy \*\*OWASP Juice Shop\*\* or \*\*DVWA\*\* in Docker and walk through each Top 10 category hands-on.

\- Pick an open-source web app and audit it against the Top 10 — write up findings.

\- Add SAST (Semgrep) + dep scanning (Trivy) + secret scanning (Gitleaks) to a project's CI to catch a few of these automatically.

\- Read the actual OWASP Top 10 PDF — it's short and worth it.



\## Sources

\- \[OWASP Top 10 (2021)](https://owasp.org/Top10/)

\- OWASP Cheat Sheet Series

\- PortSwigger Web Security Academy (free labs)

