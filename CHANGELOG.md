# Changelog

All notable changes to NEXUS AIEDR are documented here.
Format inspired by [Keep a Changelog](https://keepachangelog.com).

## [v0.3.0] — Milestone 9: Tier-Aware Knowledge Base
**Released**: April 2026

### Added
- **Commercial tier system**: 4 tiers (Free / Pro / Business / Enterprise)
  following the GitLab/Elastic single-binary licensing model
- **22 new detection rules** (78 → 100 total):
  - Free tier (+6): ransomware staging, history clearing, hidden home
    executables, tmpfs mount, worm replication, shell expansion abuse
  - Pro tier (+6): SSH backdoor, LD_LIBRARY_PATH hijack, kernel thread
    masquerade, memfd_create fileless, /bin/sh redirect, tee obfuscation
  - Business tier (+6): K8s pod escape, Docker socket abuse, cloud IMDS,
    cgroup release_agent, privileged container, K8s service account theft
  - Enterprise tier (+4): APT timestomping, bind-mount rootkit,
    ptrace injection, LOLBin chain
- `RuleTier` enum with `includes()` method (lower tiers always included)
- `RuleOs` enum for cross-platform classification
- `KnowledgeBase::filter_by_tier()` and `filter_active_rules()` methods
- 7 new unit tests for tier-aware filtering (87 unit tests total)

### Changed
- `DetectionRule` struct extended with `tier` and `os` fields
  (backward-compatible via `#[serde(default)]`)
- README: added "Detection Methodology" section with honest discussion
  of polling-based capture limitations and the eBPF migration plan

### Verified
- 60-second idle test: **0 false positives**
- All 96 tests passing (87 unit + 9 integration)
- Cascading Oracle live demo: heuristic + AI enrichment validated

---

## [v0.2.0] — Milestone 8: Detection Coverage Expansion
**Released**: April 2026

### Added
- **Detection KB grown 6x**: from 13 to 78 rules
- 6 new strategic rule families:
  - CanisterWorm Suite (5 rules) — Supply chain attack patterns
  - Initial Access (15 rules) — Web/DB/mail RCE, container escape
  - Persistence (12 rules) — systemd, cron, profile injection
  - Privilege Escalation (10 rules) — PwnKit, DirtyPipe, OverlayFS
  - Credential Access (10 rules) — /etc/shadow, browser DBs, cloud creds
  - Lateral / C2 / Exfiltration (13 rules) — SSH, Tor, DNS tunneling

### Fixed
- LocalOracle: empty-reasoning fallback for benign events (avoids
  unhelpful AI verdicts when Gemma returns no malicious indicators)
- NEX-L-CR-006: tightened `/proc/<pid>/maps` rule to require dump-tool
  process name AND `/proc/` AND `/maps` (prevents false positives on
  any process touching `/proc/` paths during normal operation)

---

## [v0.1.0] — Milestones 1–7: Foundations
**Released**: 2025–2026

### Architecture
- Workspace skeleton: `nexus-core`, `nexus-agent`, `nexus-hive` crates
- ECS-compliant event schema (Elastic Common Schema 8.11)
- Ed25519 signed envelope for KB updates (`crypto.rs`)
- Process telemetry via `sysinfo` polling at 10 Hz
- Per-host correlation engine with sliding windows
- 5-level adaptive response ladder (LOG → ISOLATE)
- LocalOracle integration with Gemma 4 E4B via llama.cpp HTTP
- Cascading Oracle architecture (heuristic primary + AI secondary)

### Initial detection set
- 3 Windows core rules (PowerShell encoded, hidden window, Office macro)
- 10 Linux core rules (reverse shells, cryptominers, scripting attacks)

---

## Roadmap

| Milestone | Focus | Status |
|-----------|-------|--------|
| M10 | eBPF kernel telemetry (replace sysinfo polling) | 🚧 Planning |
| M11 | nexus-hive P2P mesh (Ed25519 IoC propagation) | 🔜 Planned |
| M12 | License enforcement (Ed25519 key file) | 🔜 Planned |
| M13 | Windows-native agent (ETW consumer) | 🔜 Planned |
| M14 | Management UI (Leptos / Yew) | 🔜 Planned |
