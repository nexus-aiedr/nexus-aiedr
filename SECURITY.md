# Security Policy

## Reporting a Vulnerability

NorthNarrow takes security seriously. If you believe you have found a
security vulnerability in any NorthNarrow component (architecture, detection
logic, or future released code), please follow responsible disclosure.

## How to Report

**Please do NOT open a public GitHub issue for security vulnerabilities.**

Instead:

1. **Email**: *(coming soon — currently the project maintains a private
   pre-release repo and a placeholder for security contact)*
2. **GitHub Private Vulnerability Reporting**: when source code is published,
   we will enable GitHub's private vulnerability reporting feature.

## What to Include

When reporting a vulnerability, please include:

- **Affected component** (e.g., HeuristicOracle, CorrelationEngine, Knowledge Base)
- **Vulnerability type** (e.g., logic flaw, false positive at scale, bypass technique)
- **Steps to reproduce** (with as much detail as possible)
- **Potential impact** (what could an attacker achieve?)
- **Suggested fix** (if you have one)

## Response Timeline

- **Acknowledgement**: within 5 business days
- **Initial assessment**: within 14 business days
- **Fix and disclosure**: depends on severity and complexity

## Scope

### In scope
- Detection bypass techniques
- False positive patterns that could be weaponized
- Logic flaws in tier-aware filtering
- Architectural weaknesses in the Cascading Oracle design

### Out of scope (during pre-release)
- Issues in third-party dependencies (report directly to those projects)
- Theoretical attacks without practical impact
- Issues already documented in the "Known Limitations" section of the README

## Hall of Fame

When source is released and the project receives security reports,
we will maintain a public Hall of Fame for responsible disclosures.

---

*Last updated: April 2026*
