# Changelog

All notable changes to NorthNarrow are documented here.
Format inspired by [Keep a Changelog](https://keepachangelog.com).

## [v0.4.0] — Milestones 10–14.3: eBPF telemetry stack + brand transition
**Released**: 2026-04-29

### Added — kernel-side telemetry (Milestones 10–14.3)
- **M10 — eBPF process tracepoint**: BPF tracepoint on
  `sched/sched_process_exec` (kernel 5.7+) replacing the M3 sysinfo polling
  fallback as the primary process telemetry source. Full `ProcessInfo` capture
  (PID, PPID, comm, executable path) with sub-millisecond latency. Universal
  sysinfo fallback retained for hosts without BPF.
- **M11 — BPF LSM process exec enforcement** (5/6 sub-phases shipped, ALERT
  mode complete):
  - LSM hook on `bprm_check_security` with kernel-side path resolution via
    `bpf_d_path` (LSM BTF allowlist).
  - Allowlist / denylist BPF policy maps, sub-microsecond lookup.
  - Schema v4 with structured `enforcement` field on every event.
  - Cascading-oracle fast-path: skip both heuristic and AI evaluation on
    Allowlisted (suppress) and Denied (high-confidence verdict).
  - M11.6 BLOCK mode deferred to a VM-only session (a bug in BLOCK mode can
    render a host unbootable).
- **M12 — Self-protection**: watchdog supervisor (systemd `Restart=always` +
  custom fork-execv pattern with async-signal-safe `_exit`), KB SHA-256
  verification at boot, executable self-hash for tamper-detection telemetry.
- **M13 — Network telemetry**: TCP outbound capture via kprobe on
  `tcp_v4_connect` (M13.1), PID + remote tuple correlation through the
  CorrelationEngine (M13.2), DNS observability with raw query capture, Shannon
  entropy on QNAMEs for tunneling detection, and per-PID rate limiting via
  token bucket (M13.3).
- **M14 — File Integrity Monitoring** (in progress):
  - M14.1 ✅ Design doc + scaffolding (`FileEvent`, `FileEventKind` types).
  - M14.2 ✅ Five BPF LSM file hooks attached: `file_open`, `inode_unlink`,
    `inode_rename`, `inode_create`, `inode_setattr`. Path-aware `file_open`;
    `inode_*` events emit kind discriminator + PID/UID/comm with `path_len=0`.
  - M14.3 ✅ Userspace consumer + agent integration: unified `BpfLsmSource`
    drains both `EVENTS` (M11 exec) and `FILE_EVENTS` (M14 FIM) from a single
    loaded BPF object; events flow as `EcsEvent` with the new `event.action`
    ECS field set to `file_open` / `file_unlink` / `file_rename` /
    `file_create` / `file_setattr`.
  - M14.4 🚧 Correlation rules + dentry-based path resolution for the four
    `inode_*` hooks (in progress).

### Added — schema
- `EcsEvent.event_action: Option<String>` mapped to ECS `event.action`. A
  free-form fine-grained operation label (e.g. `file_open`, `process_exec`,
  `network_connect`), distinct from the coarse `event_type` enum.

### Changed — branding
- Project renamed **NEXUS AIEDR → NorthNarrow** across all public-facing
  documentation. GitHub repository moved from
  `github.com/nexus-aiedr/nexus-aiedr` to `github.com/northnarrow/northnarrow`
  (public docs) and from `github.com/nexus-aiedr/nexus-core` to
  `github.com/northnarrow/northnarrow-private` (source). GitHub auto-redirects
  from old URLs for a limited time.
- Status label switched from "pre-release" to **"alpha / pre-customer"** to
  more accurately reflect the project stage.

### Removed
- `nexus-hive` placeholder crate (was 100% empty: 2-line `lib.rs` with a
  single const, 5 unused dependencies, never imported). The P2P-mesh concept
  remains on the long-term roadmap; it will be designed with concrete
  requirements when a deployment partner needs it, not pre-allocated as an
  empty crate.
- Userspace `aya-log` dependency from `northnarrow-bpf-lsm` and
  `northnarrow-bpf-net`. Kernel-side `aya-log-ebpf` retained.
- Orphan `BpfLsmEnforcement::new` constructor (post-audit dead code).

### Test coverage
- 163 tests passing (up from 100 in v0.3.0).

---

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

Internal commit reference: `6638571` (private northnarrow-private repo).

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
- Workspace skeleton: `northnarrow-private`, `northnarrow-agent` crates
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

## Roadmap (forward-looking)

| Milestone | Focus | Status |
|-----------|-------|--------|
| M14.4    | FIM correlation rules + dentry-based path resolution for `inode_*` LSM hooks | 🚧 In progress |
| M-AI-x   | RAG-LEGIT (benign baselines) + RAG-NOLEGIT (malicious patterns) retrieval-augmented inference | 🔜 Design |
| M-AI-y   | Adversarial multi-agent training loop (5+ agents, red/blue) for verdict robustness | 🔜 Research |
| Future   | Windows-native agent (ETW consumer) | 🔜 Planned |
| Future   | Management UI (Leptos / Yew) | 🔜 Planned |
| Future   | P2P mesh for multi-host IoC propagation (mDNS + Ed25519-signed beacons) | 🔜 Planned, no crate allocated yet |
| Future   | License enforcement (Ed25519 key file) | 🔜 Planned |
