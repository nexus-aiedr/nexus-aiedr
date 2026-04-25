# NEXUS AIEDR — Threat Intelligence References

NEXUS detection rules are **not invented in isolation**. Every rule is
grounded in published research, documented attacks, or vulnerability
disclosures. This document tracks the sources.

---

## CVE References (rules backed by specific vulnerabilities)

| CVE | Vulnerability | Year | NEXUS Rule(s) |
|-----|---------------|-----:|---------------|
| CVE-2021-4034 | PwnKit — polkit pkexec local privesc | 2021 | NEX-L-PR-005 |
| CVE-2022-0847 | DirtyPipe — Linux kernel pipe write | 2022 | NEX-L-PR-008 |
| CVE-2023-2640 | OverlayFS / GameOver(lay) | 2023 | NEX-L-PR-008 |
| CVE-2024-21626 | runc container escape "Leaky Vessels" | 2024 | NEX-L-IA-004 |
| CVE-2024-4577 | PHP-CGI Windows argument injection | 2024 | NEX-L-IA-014 |
| CVE-2019-10149 | Exim mail server RCE | 2019 | NEX-L-IA-003 |
| CVE-2021-44228 | Log4Shell (Apache Log4j) | 2021 | NEX-L-IA-001 |
| CVE-2022-22965 | Spring4Shell (Spring Framework) | 2022 | NEX-L-IA-001 |

---

## APT Campaigns Referenced

### TeamPCP — CanisterWorm (March 2026)
- **Vector**: npm supply chain
- **C2 channel**: ICP (Internet Computer) canisters
- **Self-propagation**: automated `npm publish` from infected hosts
- **NEXUS coverage**: NEX-L-0011 through NEX-L-0015 (5 rules)
- **References**: JFrog, Socket, Aikido security advisories

### Outlaw / Kinsing (ongoing)
- **Vector**: Linux server cryptomining campaigns
- **Target**: Redis, Hadoop, Docker exposed instances
- **NEXUS coverage**: NEX-L-0004 (cryptominer detection)

### TeamTNT (ongoing)
- **Vector**: Cloud-native attacks (AWS, Docker, Kubernetes)
- **Tactics**: IMDS credential theft, container escape
- **NEXUS coverage**: NEX-L-BIZ-001 through NEX-L-BIZ-006

### RotaJakiro (2021–ongoing)
- **Vector**: Linux backdoor with kernel-thread masquerading
- **NEXUS coverage**: NEX-L-PRO-003 (kernel thread name fakes)

### XorDDoS (long-running)
- **Vector**: Linux DDoS botnet with rootkit features
- **NEXUS coverage**: NEX-L-ENT-002 (bind-mount rootkit)

### Symbiote (2022)
- **Vector**: Linux stealth malware via LD_PRELOAD + bind mounts
- **NEXUS coverage**: NEX-L-PE-010, NEX-L-ENT-002

### APT29 / Cozy Bear (Russia, ongoing)
- **Tactics**: Supply chain, cloud, custom backdoors
- **NEXUS coverage**: NEX-L-ENT-001 (timestomping anti-forensics)

### Turla (Russia, long-running)
- **Linux variant tactics**: SSH backdoors, kernel modules
- **NEXUS coverage**: NEX-L-PRO-001 (backdoored OpenSSH)

### APT41 / Double Dragon (China, ongoing)
- **Cross-platform**: Windows + Linux
- **Tactics**: Process injection, ptrace abuse
- **NEXUS coverage**: NEX-L-ENT-003 (ptrace injection)

### Lazarus Group (DPRK, ongoing)
- **Linux variants**: financial, supply chain
- **NEXUS coverage**: NEX-L-ENT-003, NEX-L-PRO-004 (memfd_create)

---

## Knowledge Frameworks Used

### MITRE ATT&CK Enterprise Matrix v15
Primary framework for technique mapping. Every NEXUS rule has a
`mitre_technique` field populated with the corresponding T-number
(e.g., T1059.004 for Unix Shell execution).

### GTFOBins
Reference for "Living Off the Land" binaries on Linux. NEX-L-ENT-004
(LOLBin chain) maps directly to documented GTFOBins patterns.

### PayloadsAllTheThings
Reference for shell payloads, reverse shell variants, and obfuscation
techniques. Multiple rules in the Free and Pro tiers reflect
documented patterns from this collection.

### CIS Docker Benchmark
Source for container security rules (privileged containers, Docker
socket exposure, capability abuse). Rules in Business tier follow
this benchmark's recommendations.

### OWASP Container Security Top 10
Reference for K8s-specific rules (pod escape, service account theft,
metadata service access).

### NIST SP 800-53
Compliance framework consideration for Enterprise tier reporting.

### MITRE D3FEND
Used for response action mapping (the 5-level adaptive ladder).

---

## Public Security Research Cited

The following organizations and researchers have published work that
informs NEXUS rules. Crediting them is the right thing to do.

- **JFrog Security Research** — supply chain attack disclosures
- **Socket** — npm ecosystem security
- **Aikido Security** — npm/PyPI threat intelligence
- **Trail of Bits** — container security research, audit reports
- **Aqua Security** — cloud-native attack patterns, Tracee project
- **Sysdig Threat Research** — Falco rules, real-world attack analyses
- **Mandiant / Google Cloud Threat Intelligence** — APT campaign tracking
- **Microsoft Threat Intelligence** — supply chain, identity attacks
- **CrowdStrike Intelligence** — adversary tracking (public reports)
- **Red Canary** — detection engineering blog posts
- **The DFIR Report** — incident reports with detailed IOCs
- **vx-underground** — historical malware archives

---

## How NEXUS Adds Value

NEXUS does **not** invent new threat intelligence. The work is in:

1. **Curating** — selecting which patterns are worth turning into rules
2. **Translating** — converting research findings into precise
   `MatchCondition` predicates that minimize false positives
3. **Tier-classifying** — deciding which patterns are common (Free)
   vs specialized (Enterprise)
4. **Validating** — ensuring rules don't trigger on legitimate activity
   in real Linux environments
5. **Integrating** — wiring rules into the Cascading Oracle so AI can
   handle the ambiguous cases

The detection rules are NEXUS's distinctive contribution. The
underlying threat research is shared with the global security community.

---

## Contributing Threat Intelligence

When the project goes public, we will accept rule contributions through
PRs. Each contributed rule must include:

- A clear research reference (CVE, blog post, paper, public IOC)
- A specific MITRE ATT&CK technique
- Unit tests demonstrating both true positive and absence of false
  positive on benign activity

For now, while in pre-release, please reach out via the contact
information in the main README.

---

*Last updated: April 2026 (Milestone 9 — v0.3.0)*
