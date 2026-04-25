# NEXUS AIEDR — Tier System & Commercial Model

This document explains how NEXUS implements commercial tiers and
why the model was chosen.

---

## The Single-Binary, Feature-Flagged Model

NEXUS follows the **single-binary commercial tier model** pioneered
by GitLab and adopted by Elastic, JetBrains (sort of), Tailscale, and
many modern open-core companies.

### How it works

- One Rust binary is built, signed, and shipped to all customers
- A signed license file unlocks features and detection rules at boot
- Higher tiers see more rules and features; lower tiers see a subset
- Features above the licensed tier **never load into memory**

### Why this model

| Approach | Single binary | Multiple binaries |
|----------|:---:|:---:|
| Build complexity | Low | High |
| Distribution | Simple | Complex (per-tier artifacts) |
| Patch deployment | Universal | Per-tier coordination |
| Security audit | Single surface | Multiple surfaces |
| Customer upgrade | License swap | Full reinstall |

The single-binary approach **simplifies everything except the licensing
logic itself**, which is concentrated in one well-tested module.

---

## The Four Tiers

### 🆓 Free (Community)

**Target audience**: hobbyists, home labs, students, security researchers

**Includes**:
- Core detection rules (54 in v0.3.0)
- HeuristicOracle (deterministic engine)
- Single-host monitoring
- Local logging only
- Manual KB updates

**Does NOT include**:
- LocalOracle / AI reasoning
- nexus-hive P2P mesh
- Compliance reports
- Commercial support

**License model (planned)**: AGPLv3
**Price**: Free

---

### 🥉 Pro

**Target audience**: freelancers, small dev teams, individual professionals

**Includes** (everything in Free, plus):
- 21 additional rules (75 total)
- LocalOracle / Gemma 4 AI integration
- Auto-update KB (signed feed)
- Email-based alerting
- Basic dashboard
- Email support (48h response)

**License model (planned)**: Commercial (annual)
**Indicative price**: €49 per host per year

---

### 🥈 Business

**Target audience**: SMEs, MSPs, mid-market security teams

**Includes** (everything in Pro, plus):
- 16 additional rules (91 total)
- nexus-hive P2P mesh (multi-host coordination)
- Compliance reports (ISO 27001, NIS2, GDPR)
- SIEM integration (syslog/CEF to Elastic, Splunk, etc.)
- Threat intel feed (signed by NEXUS team)
- Phone support (24h response)

**License model (planned)**: Commercial (annual)
**Indicative price**: €149 per host per year

---

### 🥇 Enterprise

**Target audience**: governments, banks, critical infrastructure,
regulated industries

**Includes** (everything in Business, plus):
- All 100 rules (full catalog)
- Custom rule development support
- Air-gapped deployment certification
- SLA 99.9% uptime guarantee
- Dedicated incident response
- Compliance attestation (audit reports)
- White-label option

**License model (planned)**: Commercial (annual, negotiated)
**Indicative price**: from €399 per host per year

---

## Tier Inclusion Rules

Higher tiers always include all features of lower tiers. The `RuleTier`
enum implements an `includes()` check:

| License | Free rules | Pro rules | Business rules | Enterprise rules | Total visible |
|---------|:---:|:---:|:---:|:---:|:---:|
| Free | ✅ | ❌ | ❌ | ❌ | 54 |
| Pro | ✅ | ✅ | ❌ | ❌ | 75 |
| Business | ✅ | ✅ | ✅ | ❌ | 91 |
| Enterprise | ✅ | ✅ | ✅ | ✅ | 100 |

---

## Pricing Philosophy

NEXUS pricing is intentionally lower than US-centric competitors:

| Solution | Indicative price (per host/year) |
|----------|----------------------------------|
| CrowdStrike Falcon | $50–$200 |
| SentinelOne | $50–$150 |
| Microsoft Defender for Endpoint | $30–$60 (with Microsoft 365) |
| Wazuh | Free (community) / commercial support varies |
| **NEXUS Pro** | €49 |
| **NEXUS Business** | €149 |
| **NEXUS Enterprise** | from €399 |

The reasoning:

- Italian/EU market has lower budget tolerance than US enterprise
- PMI italiane (small/mid Italian businesses) are a primary target
- Sovereignty-conscious buyers (gov, regulated) pay for compliance,
  not for raw feature count
- Open-source Free tier serves as the on-ramp for paid tiers

---

## License Enforcement (Milestone 12, planned)

The current state (v0.3.0) does **not enforce licenses**. The agent runs
with the full Enterprise tier hardcoded as default, for development.

Milestone 12 will introduce:

- **Ed25519-signed license file** loaded at boot
- **Offline validation** (no phone-home required — air-gapped friendly)
- **Tier extracted from signed claims** (subject, expiry, hosts, features)
- **Grace period**: expired licenses downgrade to Free, no service interruption
- **Revocation list** (CRL): for stolen/compromised licenses

Until M12, all features are available regardless of license state.

---

## Legal Note

Tier names, prices, and features documented here represent the **current
plan**. They may change before commercial release. This document is not
an offer of sale.

For commercial inquiries, please contact the project maintainers.

---

*Last updated: April 2026 (Milestone 9 — v0.3.0)*
