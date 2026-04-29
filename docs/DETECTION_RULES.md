# NorthNarrow — Detection Rules Catalog

Complete list of the 100 detection rules currently in NorthNarrow v0.3.0.
Each rule is mapped to MITRE ATT&CK and classified by commercial tier.

**Note**: rule IDs and names are public. The actual matching conditions
(regex, command_line patterns, IoC values) are part of the closed-source
Knowledge Base and are not disclosed here.

---

## Tier Distribution

| Tier | Rules | Purpose |
|------|------:|---------|
| 🆓 Free | 54 | Malware/ransomware/scripting common patterns |
| 🥉 Pro | 21 | Supply chain, advanced detection, evasion |
| 🥈 Business | 16 | Server, container, cloud, post-exploit |
| 🥇 Enterprise | 9 | APT-tier, kernel-level, advanced rootkits |
| **Total** | **100** | |

---

## Rule Categories

### Windows Core (3 rules) — Free tier
| ID | Name | MITRE |
|----|------|-------|
| NEX-W-0001 | PowerShell with encoded command | T1059.001 |
| NEX-W-0002 | PowerShell hidden window | T1059.001 |
| NEX-W-0003 | Office macro from Downloads | T1204.002 |

### Linux Core (10 rules) — Free tier
| ID | Name | MITRE |
|----|------|-------|
| NEX-L-0001 | Bash TCP reverse shell | T1059.004 |
| NEX-L-0002 | Download from raw IP (DNS evasion) | T1105 |
| NEX-L-0003 | chmod +x on temporary directory | T1222.002 |
| NEX-L-0004 | Known cryptominer process | T1496 |
| NEX-L-0005 | SUID binary enumeration | T1083 |
| NEX-L-0006 | Sudo privilege enumeration | T1033 |
| NEX-L-0007 | Base64 piped to shell | T1140 |
| NEX-L-0008 | Netcat listening | T1090 |
| NEX-L-0009 | Python reverse shell oneliner | T1059.006 |
| NEX-L-0010 | Shell redirect to /dev/tcp | T1059.004 |

### CanisterWorm Suite (5 rules) — Pro tier
Threat intelligence: TeamPCP supply-chain campaign, March 2026
References: JFrog, Socket, Aikido security advisories

| ID | Name | MITRE |
|----|------|-------|
| NEX-L-0011 | npm postinstall spawns Python | T1195.002 |
| NEX-L-0012 | systemd --user persistent service | T1543.002 |
| NEX-L-0013 | ICP canister C2 traffic | T1102 |
| NEX-L-0014 | Credential file harvest (.npmrc/.ssh/.aws) | T1552.001 |
| NEX-L-0015 | Background npm publish (worm) | T1195.002 |

### Initial Access (15 rules)
| ID | Name | MITRE | Tier |
|----|------|-------|------|
| NEX-L-IA-001 | Web server spawning shell (LFI/RCE) | T1190 | Business |
| NEX-L-IA-002 | Database process spawning shell | T1190 | Business |
| NEX-L-IA-003 | Mail server spawning shell | T1190 | Business |
| NEX-L-IA-004 | Container escape: shell on host PID ns | T1611 | Business |
| NEX-L-IA-005 | Cron job triggering interactive shell | T1053.003 | Free |
| NEX-L-IA-006 | Download with base64 decode pipe | T1140 | Free |
| NEX-L-IA-007 | curl/wget piped directly to shell | T1059.004 | Free |
| NEX-L-IA-008 | OpenSSL used as download tool | T1105 | Pro |
| NEX-L-IA-009 | dd writing to system path | T1565 | Free |
| NEX-L-IA-010 | xxd reverse hex to executable | T1140 | Pro |
| NEX-L-IA-011 | awk/sed used as command executor | T1059.004 | Free |
| NEX-L-IA-012 | Perl one-liner with socket | T1059.006 | Free |
| NEX-L-IA-013 | Ruby one-liner with TCPSocket | T1059.006 | Free |
| NEX-L-IA-014 | PHP -r with system/exec/shell_exec | T1059.004 | Free |
| NEX-L-IA-015 | Node.js -e with child_process | T1059.007 | Free |

### Persistence (12 rules)
| ID | Name | MITRE | Tier |
|----|------|-------|------|
| NEX-L-PE-001 | systemd service unit in /etc/systemd | T1543.002 | Free |
| NEX-L-PE-002 | rc.local modification | T1037.004 | Free |
| NEX-L-PE-003 | init.d script creation | T1037.004 | Free |
| NEX-L-PE-004 | XDG autostart entry creation | T1547.013 | Free |
| NEX-L-PE-005 | crontab interactive edit | T1053.003 | Free |
| NEX-L-PE-006 | Write to /etc/cron.* directories | T1053.003 | Free |
| NEX-L-PE-007 | 'at' command for one-shot scheduling | T1053.001 | Free |
| NEX-L-PE-008 | User shell rc/profile modification | T1546.004 | Free |
| NEX-L-PE-009 | /etc/profile.d/ script creation | T1546.004 | Pro |
| NEX-L-PE-010 | LD_PRELOAD library injection | T1574.006 | Pro |
| NEX-L-PE-011 | authorized_keys modification | T1098.004 | Pro |
| NEX-L-PE-012 | sshd_config modification | T1098.004 | Pro |

### Privilege Escalation & Defense Evasion (10 rules)
| ID | Name | MITRE | Tier |
|----|------|-------|------|
| NEX-L-PR-001 | SUID/SGID binary creation | T1548.001 | Business |
| NEX-L-PR-002 | sudoers file modification | T1548.003 | Business |
| NEX-L-PR-003 | Kernel exploit binary from /tmp | T1068 | Enterprise |
| NEX-L-PR-004 | Capability abuse (setcap CAP_SYS_ADMIN) | T1548 | Business |
| NEX-L-PR-005 | Polkit pkexec spawning shell (PwnKit) | T1068 | Pro |
| NEX-L-PR-006 | Bash history clearing | T1070.003 | Pro |
| NEX-L-PR-007 | Debugger attach (gdb/strace) | T1622 | Free |
| NEX-L-PR-008 | DirtyPipe / OverlayFS exploit pattern | T1068 | Enterprise |
| NEX-L-PR-009 | Log tampering (auditd/journald) | T1070.002 | Enterprise |
| NEX-L-PR-010 | MAC disable (AppArmor/SELinux) | T1562.001 | Enterprise |

### Credential Access & Discovery (10 rules)
| ID | Name | MITRE | Tier |
|----|------|-------|------|
| NEX-L-CR-001 | /etc/shadow read | T1003.008 | Free |
| NEX-L-CR-002 | Browser credential database access | T1555.003 | Free |
| NEX-L-CR-003 | SSH private key access | T1552.004 | Free |
| NEX-L-CR-004 | AWS credentials harvesting | T1552.001 | Free |
| NEX-L-CR-005 | kubeconfig theft | T1552.007 | Free |
| NEX-L-CR-006 | /proc/<pid>/maps memory dumping | T1003.007 | Pro |
| NEX-L-CR-007 | Cloud creds (Azure/GCP) | T1552.001 | Business |
| NEX-L-CR-008 | GnuPG keyring access | T1552.004 | Free |
| NEX-L-CR-009 | Post-exploit toolkit chain | T1057 | Business |
| NEX-L-CR-010 | Network configuration enumeration | T1016 | Free |

### Lateral Movement, C2 & Exfiltration (13 rules)
| ID | Name | MITRE | Tier |
|----|------|-------|------|
| NEX-L-LM-001 | SSH brute force / credential spray | T1110.003 | Free |
| NEX-L-LM-002 | SSH agent forwarding abuse | T1563.001 | Free |
| NEX-L-LM-003 | rsync to remote SSH host | T1048.002 | Free |
| NEX-L-LM-004 | scp file copy to remote host | T1048.002 | Free |
| NEX-L-LM-005 | DNS tunneling indicators | T1071.004 | Free |
| NEX-L-LM-006 | Tor process or .onion address | T1090.003 | Pro |
| NEX-L-LM-007 | Reverse SSH tunnel (-R port forward) | T1572 | Pro |
| NEX-L-LM-008 | ngrok / cloudflare tunnel for C2 | T1572 | Free |
| NEX-L-LM-009 | Socat creating bidirectional channel | T1572 | Free |
| NEX-L-LM-010 | Cloud storage upload (rclone/s3/gsutil) | T1567.002 | Business |
| NEX-L-LM-011 | Mass file compression (sensitive paths) | T1560.001 | Free |
| NEX-L-LM-012 | Cryptocurrency wallet file access | T1005 | Free |
| NEX-L-LM-013 | Webhook exfil (Discord/Slack/Telegram) | T1567 | Enterprise |

### Free Tier Additions — Common Threats (6 rules, M9)
| ID | Name | MITRE |
|----|------|-------|
| NEX-L-FR-001 | Mass file rename pattern (ransomware) | T1486 |
| NEX-L-FR-002 | Aggressive shell history clearing | T1070.003 |
| NEX-L-FR-003 | Hidden executable from home dotfile | T1564.001 |
| NEX-L-FR-004 | Suspicious tmpfs mount | T1564 |
| NEX-L-FR-005 | Self-replication via mass copy | T1080 |
| NEX-L-FR-006 | Shell expansion abuse ($IFS) | T1027.001 |

### Pro Tier Additions — Advanced Evasion (6 rules, M9)
| ID | Name | MITRE |
|----|------|-------|
| NEX-L-PRO-001 | Backdoored OpenSSH binary | T1554 |
| NEX-L-PRO-002 | LD_LIBRARY_PATH override | T1574 |
| NEX-L-PRO-003 | Process masquerading as kernel thread | T1036.005 |
| NEX-L-PRO-004 | memfd_create fileless execution | T1620 |
| NEX-L-PRO-005 | Reverse shell via /bin/sh redirection | T1059.004 |
| NEX-L-PRO-006 | tee + /dev/null obfuscation | T1027 |

### Business Tier Additions — Container/Cloud (6 rules, M9)
| ID | Name | MITRE |
|----|------|-------|
| NEX-L-BIZ-001 | Kubernetes pod escape (host PID ns) | T1611 |
| NEX-L-BIZ-002 | Docker socket access for breakout | T1611 |
| NEX-L-BIZ-003 | Cloud metadata service access (IMDS) | T1552.005 |
| NEX-L-BIZ-004 | Cgroup release_agent escape | T1611 |
| NEX-L-BIZ-005 | Privileged Docker container launch | T1611 |
| NEX-L-BIZ-006 | K8s service account token access | T1552.001 |

### Enterprise Tier Additions — APT (4 rules, M9)
| ID | Name | MITRE |
|----|------|-------|
| NEX-L-ENT-001 | Timestomping via touch -t | T1070.006 |
| NEX-L-ENT-002 | Bind mount overlay rootkit | T1014 |
| NEX-L-ENT-003 | ptrace process injection | T1055.008 |
| NEX-L-ENT-004 | LOLBin chain (legitimate parent) | T1218 |

---

## MITRE ATT&CK Coverage

NorthNarrow v0.3.0 covers **60+ unique MITRE ATT&CK techniques** across the
following tactics:

- **TA0001** Initial Access (15 rules)
- **TA0002** Execution (broad coverage)
- **TA0003** Persistence (12 rules)
- **TA0004** Privilege Escalation (10 rules)
- **TA0005** Defense Evasion (multiple)
- **TA0006** Credential Access (10 rules)
- **TA0007** Discovery (multiple)
- **TA0008** Lateral Movement (8 rules)
- **TA0010** Exfiltration (5 rules)
- **TA0011** Command and Control (5 rules)
- **TA0040** Impact (ransomware patterns)

---

## Threat Intelligence Sources

Every rule is grounded in published research. Reference sources:

- **CVE database** for vulnerability-driven rules
- **MITRE ATT&CK** Enterprise Matrix v15
- **GTFOBins** for LOLBin detection
- **PayloadsAllTheThings** for payload patterns
- **CIS Docker Benchmark** for container rules
- **OWASP Container Security Top 10** for K8s rules
- Specific APT campaign reports (Mandiant, JFrog, Aqua, Trail of Bits)

---

*Last updated: April 2026 (Milestone 9 — v0.3.0)*
