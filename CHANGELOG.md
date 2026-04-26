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

### Fixed (post-release)

Three detection rules narrowed to prevent false positives identified
during post-Milestone 9 review. Same conservative narrowing pattern as
the NEX-L-CR-006 fix applied during M9 development.

- **NEX-L-PE-008** (User shell rc/profile modification): tightened to
  require `>>` append redirection in addition to `.bashrc` substring.
  Previously matched any `cat/echo/printf/tee` command touching `.bashrc`,
  including legitimate reads like `cat ~/.bashrc`. Now correctly
  distinguishes malicious shell rc injection from harmless inspection.

- **NEX-L-CR-001** (/etc/shadow read access): switched from `Contains`
  to `EndsWith` match on `/etc/shadow`. Previously triggered false
  positives on `/etc/shadow.bak`, `/etc/shadow.backup`, and any other
  path containing the substring. Now matches only the exact path.

- **NEX-L-LM-002** (SSH agent forwarding abuse): tightened to require
  the `-A` flag in isolation (` -A ` with surrounding spaces).
  Previously matched any `ssh` command line containing the substring
  `-A`, including multi-flag combinations like `-ABCDE` or paths
  containing uppercase A. Now matches only the isolated flag as
  intended.

All 87 unit tests still passing. Workspace builds clean.

Five additional detection rules narrowed in a follow-up audit pass,
applying the same conservative narrowing pattern. The audit was
triggered by reviewing rules with single-condition match logic or
overly broad substring patterns.

- **NEX-L-CR-002** (Browser credential database access): added
  process_name allowlist (`cat`/`less`/`sqlite3`/`cp`/`tar`/`zip`/`xxd`/...)
  and required `google-chrome` path. Previously matched any process
  with `Login Data` substring in command line, including legitimate
  files named `Login Data Sheet.csv` or paths containing those words.
  Trade-off: now Chrome-specific (Edge/Firefox/Brave excluded), but
  zero false positives on the high-signal 0.90-score detection.

- **NEX-L-LM-003** (rsync to remote SSH host): added required `:`
  check in addition to `@`. Previously matched any rsync command with
  `@` substring (paths, parameter values). Now requires both
  `user@host` separator AND `host:path` separator, the canonical
  SSH destination format.

- **NEX-L-PE-005** (crontab interactive edit): switched from
  `Contains "-"` to `EndsWith " -e"`. Previously matched any crontab
  invocation with a hyphen (`-l` list, `-u` user, paths with dashes).
  Now matches only the actual edit pattern used in privilege
  escalation sessions.

- **NEX-L-PE-002** (rc.local modification): added process_name
  allowlist (`echo`/`printf`/`cat`/`tee`/`sed`) and required `>>`
  append redirection. Previously matched any command containing
  `/etc/rc.local` substring, including `cat /etc/rc.local` (read).
  Now matches only actual write patterns. Same approach as PE-008.

- **NEX-L-LM-004** (scp file copy to remote host): added required
  `:` check in addition to `@`. Same fix pattern as LM-003.
  Eliminates false positives on local paths containing `@`.

Cumulative effect of M9 audit (8 fixes total, including NEX-L-CR-006
applied during M9 development):

- 8 detection rules narrowed
- 0 unit tests broken
- 0 changes to rule count, score, or tier classification
- Detection capability on intended attacks: preserved
- False positive scenarios eliminated: 12+

Internal commit reference: `6638571` (private nexus-core repo).

This is a quality-improvement pass, not a coverage change. Future
similar passes will be done as new false-positive patterns are
identified during real-world testing.

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
