\# Nmap Techniques



\## What it is

\*\*Nmap\*\* (Network Mapper) is the standard tool for network discovery and

port scanning. It answers: \*what hosts are up, what ports are open, what

services are running, and what OS is behind them?\*



It's the first tool in almost every real reconnaissance workflow — offensive

(pentest, CTF) and defensive (attack-surface inventory). Understanding what

each scan actually sends on the wire separates people who \*use\* Nmap from

people who \*understand\* it.



\*\*Ethics rule:\*\* only scan hosts you own or have written permission to

scan. Unauthorized port scans are illegal in many jurisdictions.

Practice on: `scanme.nmap.org` (Nmap's own permission-granted target),

your own machines, TryHackMe/HTB labs, or a local VM.



\## The mental model

```mermaid

flowchart LR

&#x20;   Discovery\[Host Discovery<br/>Who's up?] --> PortScan\[Port Scan<br/>What's open?]

&#x20;   PortScan --> Service\[Service Detection<br/>What's running?]

&#x20;   Service --> Version\[Version Detection<br/>Which version?]

&#x20;   Version --> OS\[OS Fingerprint<br/>Which OS?]

&#x20;   OS --> Scripts\[NSE Scripts<br/>Vulns / enum]

```



Every real scan is layered: find hosts → find ports → identify services →

identify versions → run targeted scripts. Each layer is noisier than the last.



\## Essential commands (memorize these)



| Command | What it does |

|---|---|

| `nmap 192.168.1.0/24` | Basic scan of a subnet (top 1000 ports) |

| `nmap -sn 192.168.1.0/24` | \*\*Ping scan\*\* — host discovery only, no ports |

| `nmap -sS -p- <target>` | \*\*SYN scan\*\*, all 65535 ports |

| `nmap -sV <target>` | Service + version detection |

| `nmap -O <target>` | OS fingerprint |

| `nmap -A <target>` | Aggressive: version + OS + scripts + traceroute |

| `nmap -sU -p 53,161 <target>` | UDP scan (slow — always specify ports) |

| `nmap -Pn <target>` | Skip host discovery (target blocks ping) |

| `nmap -oA scan\_output <target>` | Save results in all formats (normal, XML, grep) |

| `nmap --script vuln <target>` | Run all "vuln" category NSE scripts |



\## The scan types (what actually goes on the wire)



\- \*\*`-sS` SYN scan (default when root)\*\* — sends `SYN`, if `SYN-ACK` back → open, if `RST` → closed. Never completes the handshake. Called "half-open" or "stealth." Fastest reliable scan.

\- \*\*`-sT` TCP connect scan\*\* — full 3-way handshake. Slower and noisier, but works without root. What non-root users get by default.

\- \*\*`-sU` UDP scan\*\* — no handshake in UDP, so Nmap must send app-layer probes and interpret ICMP responses. \*\*Very slow.\*\* Always scope with `-p`.

\- \*\*`-sn` Ping scan\*\* — host discovery only, skips port scan. Great for mapping which hosts are live in a subnet.

\- \*\*`-sA` ACK scan\*\* — used to map firewall rules (differentiates filtered vs unfiltered).

\- \*\*`-sN` / `-sF` / `-sX` Null / FIN / Xmas\*\* — sends TCP packets with weird flag combos. Sometimes bypasses old stateless firewalls, useless against modern stacks.



\## Port states — what each one means

\- \*\*open\*\* — service is actively accepting connections.

\- \*\*closed\*\* — host is up, port is reachable, nothing listening.

\- \*\*filtered\*\* — Nmap can't tell (firewall dropped the packet). Try `-Pn`, different scan type, or timing.

\- \*\*unfiltered\*\* — reachable but state can't be determined (rare, seen with ACK scan).

\- \*\*open|filtered\*\* — Nmap isn't sure (common with UDP).



Interviews love the distinction between \*\*closed\*\* and \*\*filtered\*\*. Closed = host responded with RST. Filtered = no response at all.



\## Timing templates — the speed dial

`-T0` (paranoid, IDS-evading) through `-T5` (insane, LAN only).

Defaults to `-T3`. Use `-T4` on modern networks for a good speed/accuracy

tradeoff. `-T5` will drop packets and give unreliable results.



\## Common workflows



\*\*Quick discovery of a network:\*\*

```bash

nmap -sn 192.168.1.0/24

```



\*\*Full attack-surface enumeration of one host:\*\*

```bash

sudo nmap -sS -sV -O -p- -T4 -oA full\_scan target.com

```



\*\*Fast top-100 ports with versions:\*\*

```bash

nmap -sV --top-ports 100 target.com

```



\*\*Web-focused enum:\*\*

```bash

nmap -sV -p 80,443,8080,8443 --script "http-title,http-headers,http-enum" target.com

```



\*\*Vuln sweep (NSE vuln scripts):\*\*

```bash

nmap -sV --script vuln target.com

```



\## NSE — the Nmap Scripting Engine

`--script` lets you run Lua scripts for enum, brute force, vulns, discovery,

etc. Categories: `default`, `discovery`, `safe`, `intrusive`, `vuln`,

`exploit`, `brute`, `malware`, `dos`, `auth`.



Useful ones:

\- `smb-enum-shares` — enumerate SMB shares

\- `http-title`, `http-headers`, `http-enum` — web recon

\- `ssh-auth-methods` — what auth SSH accepts

\- `ssl-enum-ciphers` — TLS cipher audit

\- `dns-brute` — subdomain guessing



List scripts: `ls /usr/share/nmap/scripts/`. Read the script comments — they

tell you what it does and what category it's in.



\## Output formats

\- `-oN file` — normal (human-readable)

\- `-oX file` — XML (parseable, feed into tools)

\- `-oG file` — grep-friendly

\- `-oA prefix` — all three at once



Save your scans. Pipe XML into tools like \*\*Metasploit\*\* (`db\_import`) or

convert with `xsltproc` to HTML for reports.



\## Common gotchas

\- \*\*`-sS` needs root/admin.\*\* Without it, you silently fall back to `-sT` (slower, logged by target apps).

\- \*\*Default scan is top 1000 ports, not all.\*\* Use `-p-` for all 65535 when it matters.

\- \*\*UDP is painful.\*\* Don't `-sU` an entire subnet with all ports and expect results this century.

\- \*\*`-Pn` is often needed.\*\* Modern hosts (Windows, cloud VMs) drop ICMP, and Nmap will report "host down" incorrectly without it.

\- \*\*Aggressive scans get you noticed / banned.\*\* `-A` is fingerprintable in seconds by any decent IDS. Use `-T0`/`-T1` if stealth matters.

\- \*\*NAT / cloud loadbalancers.\*\* Scanning `example.com` may hit a CDN edge, not the actual origin. Interpret results accordingly.

\- \*\*`-O` needs at least one open and one closed port\*\* to fingerprint accurately.

\- \*\*Version scan (`-sV`) can crash fragile services.\*\* Rare, but real. Don't run on production without a change window.



\## Nmap on Windows

Runs fine (install from nmap.org). Some scan types (raw sockets) need admin

cmd/PowerShell. The GUI is called \*\*Zenmap\*\* — good for learning; on the job

everyone uses the CLI.



\## Related tools worth knowing

\- \*\*masscan\*\* — faster than Nmap for huge ranges, less clever. Use for initial sweep, then Nmap for detail.

\- \*\*rustscan\*\* — fast port discovery, pipes ports into Nmap for the deep dive.

\- \*\*naabu\*\* — ProjectDiscovery's port scanner, integrates with their recon suite.

\- \*\*netcat / ncat\*\* — banner grabbing and manual probing.



\## What I want to try

\- Run `nmap scanme.nmap.org` (their explicitly-permitted target) and read every line of output.

\- Scan my own machine: `sudo nmap -sS -sV -p- 127.0.0.1` — see what I'm actually running.

\- Set up a \*\*Metasploitable\*\* VM and enumerate it fully with Nmap — walk through each open port.

\- Compare `-sS` vs `-sT` vs `-sA` output on the same target and understand the difference.

\- Write a small NSE script (Lua) — the easiest way to internalize how the scripting engine works.

\- Run `masscan` on a `/16` and pipe results into `nmap` for detailed follow-up.



\## Sources

\- \[nmap.org](https://nmap.org/) — official docs

\- `man nmap` — thorough

\- "Nmap Network Scanning" — Gordon Lyon (the author, free chapters online)

\- HackTricks Nmap page

\- TryHackMe: \*\*Nmap\*\* room (free)



