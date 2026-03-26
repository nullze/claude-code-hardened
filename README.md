# 🔐 claude-code-hardened

> **Maximum-restriction security configuration for Claude Code — built for enterprise, high-security, and regulated environments on macOS, Windows, and Linux.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Security Policy](https://img.shields.io/badge/Security-Hardened-red)](SECURITY.md)
[![Claude Code](https://img.shields.io/badge/Claude-Code-orange)](https://claude.ai/code)
[![Platform](https://img.shields.io/badge/Platform-macOS%20%7C%20Windows%20%7C%20Linux-blue)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## What Is This?

`claude-code-hardened` is a production-ready, open-source security configuration for [Claude Code](https://claude.ai/code) — Anthropic's agentic AI coding assistant. It provides a hardened `settings.json` and an MDM-deployable `managed_settings.json` that lock Claude Code to a **read-and-advise-only posture** with mandatory human approval gates before any file write, shell command, or system interaction.

Built by security engineers through direct penetration testing of Claude Code, Claude Cowork, and Claude Chat in enterprise environments. Designed for deployment on **macOS, Windows, and Linux** across Claude Teams and Claude for Work licenses.

### Who This Is For

- Security and platform engineering teams deploying Claude Code across developer fleets
- Organizations subject to **SOC 2, ISO 27001, HIPAA, FedRAMP, PCI-DSS, or GDPR** requirements
- Developers working with production credentials, cloud sessions, or sensitive codebases
- Teams using **Claude Cowork**, MCP connectors, or autonomous Claude Code sessions
- Any company that wants AI assistance without giving an AI agent unconstrained OS access

---

## The Problem

Claude Code runs as a **local agentic process that inherits your OS user's full identity** — every credential, cloud session, SSH key, secret, and permission on the machine. Without hardening, in a single session it can:

| What Claude Code Can Do (Default) | Real-World Impact |
|---|---|
| Read `~/.aws/credentials`, `~/.ssh/id_rsa`, `.env` | Full cloud account or infrastructure compromise |
| Execute `curl` or `wget` to external endpoints | Silent exfiltration of source code and credentials |
| Run `pip install` or `npm install` | Malicious package execution at install time |
| Write to `~/.zshrc`, LaunchAgents, or Startup folders | Persistent backdoor surviving session termination |
| Execute `aws`, `az`, `kubectl` with your active sessions | Complete cloud tenant compromise |
| Be prompt-injected via a README or code comment | Attacker-controlled execution under your identity |
| On Windows: invoke `powershell`, `certutil`, `bitsadmin` | LOLBin-based exfiltration bypassing AV/EDR |
| On Linux: read `/proc/*/environ`, write to systemd | Credential harvesting and persistent system-level execution |

**One malicious comment in a dependency's README is enough.** This configuration is the blast-radius reduction layer between Claude Code and your infrastructure.

---

## Repository Structure

```
claude-code-hardened/
├── README.md                        # This file
├── settings.json                    # User/project-level hardened config
├── managed_settings.json            # Enterprise MDM policy (non-overridable)
├── CONTRIBUTING.md                  # Contribution guidelines
├── SECURITY.md                      # Responsible disclosure policy
├── CHANGELOG.md                     # Version history
└── examples/
    ├── python-project.json          # Minimal allow-list for Python projects
    ├── node-project.json            # Minimal allow-list for Node.js/TS projects
    └── readonly-audit.json          # Zero-write audit/security review mode
```

---

## Quick Start

### Option 1 — User-Global (all Claude Code sessions for your OS user)

**macOS / Linux:**
```bash
mkdir -p ~/.claude
cp ~/.claude/settings.json ~/.claude/settings.json.bak 2>/dev/null || true
curl -o ~/.claude/settings.json \
  https://raw.githubusercontent.com/YOUR_ORG/claude-code-hardened/main/settings.json
```

**Windows (PowerShell):**
```powershell
New-Item -ItemType Directory -Force -Path "$env:APPDATA\Claude"
Copy-Item "$env:APPDATA\Claude\settings.json" `
  "$env:APPDATA\Claude\settings.json.bak" -ErrorAction SilentlyContinue
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/nullze/claude-code-hardened/main/settings.json" `
  -OutFile "$env:APPDATA\Claude\settings.json"
```

### Option 2 — Per-Project (applies only within a specific repository)

```bash
mkdir -p .claude
curl -o .claude/settings.json \
  https://raw.githubusercontent.com/nullze/claude-code-hardened/main/settings.json
```

> **Commit `.claude/settings.json` to your repository.** This ensures every developer using Claude Code in the project inherits the same security posture automatically, without individual configuration.

### Option 3 — Enterprise MDM Deployment (non-overridable)

#### macOS (Jamf / Mosyle / Intune)
Deploy `managed_settings.json` to:
```
/Library/Application Support/Claude/managed_settings.json
```
Lock after deployment:
```bash
sudo chown root:wheel "/Library/Application Support/Claude/managed_settings.json"
sudo chmod 444 "/Library/Application Support/Claude/managed_settings.json"
```

#### Windows (Intune / GPO)
Deploy `managed_settings.json` to:
```
C:\ProgramData\Claude\managed_settings.json
```
Lock after deployment (run as Administrator):
```powershell
$path = "C:\ProgramData\Claude\managed_settings.json"
icacls $path /inheritance:r /grant:r "SYSTEM:F" /grant:r "Administrators:R"
```

#### Linux (Ansible / Chef / Puppet)
Deploy `managed_settings.json` to:
```
/etc/claude/managed_settings.json
```
Lock after deployment:
```bash
sudo chown root:root /etc/claude/managed_settings.json
sudo chmod 444 /etc/claude/managed_settings.json
```

> Settings in `managed_settings.json` **cannot be overridden** by user-level or project-level `settings.json`. This is the recommended deployment method for regulated and enterprise environments.

---

## What This Configuration Blocks

### Credential & Secret Paths (all platforms)

| Platform | Paths Protected |
|---|---|
| **macOS** | `~/.ssh`, `~/.aws`, `~/.azure`, `~/.config/gcloud`, `~/.kube`, `~/.gnupg`, `~/Library/Keychains`, 1Password, Bitwarden |
| **Windows** | `AppData/Roaming/Microsoft/Credentials`, `AppData/Local/Microsoft/Credentials`, `Windows/System32/config` (SAM hive), GitHub CLI, 1Password, Bitwarden, PowerShell profiles |
| **Linux** | `/home/*/.ssh`, `/home/*/.aws`, `/home/*/.kube`, `/home/*/.gnupg`, `/home/*/.local/share/keyrings`, `/root/*`, `/proc/*/environ`, `/proc/*/cmdline` |
| **Cross-platform** | All `.env`, `.env.*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `*.jks`, `id_rsa*`, `id_ed25519*`, `terraform.tfstate`, `credentials.json`, `service-account*.json`, `*.rdp`, `*.ppk` |

### Dangerous Commands (all platforms)

| Category | Blocked Commands |
|---|---|
| **Network** | `curl`, `wget`, `nc`, `socat`, `ssh`, `scp`, `rsync`, `ftp`, `telnet`, `nmap`, `tcpdump` |
| **Cloud CLIs** | `aws`, `az`, `gcloud`, `kubectl`, `helm`, `eksctl`, `terraform`, `vault`, `consul`, `nomad` |
| **Containers** | `docker`, `podman`, `minikube`, `kind`, `k3d`, `containerd` |
| **Package Managers** | `pip install`, `npm install`, `yarn add`, `pnpm`, `bun`, `brew`, `gem`, `cargo`, `go get`, `composer`, `apt`, `yum`, `dnf`, `snap`, `flatpak`, `winget`, `choco`, `scoop` |
| **Privilege Escalation** | `sudo`, `su`, `runas`, `doas`, `pkexec`, `chmod`, `chown`, `icacls`, `takeown` |
| **Persistence** | `crontab`, `launchctl`, `systemctl`, `schtasks`, `sc.exe`, `update-rc.d`, `at`, `systemd-run` |
| **Windows LOLBins** | `powershell`, `cmd.exe`, `wscript`, `cscript`, `mshta`, `rundll32`, `regsvr32`, `msiexec`, `certutil`, `bitsadmin`, `wmic`, `reg`, `regedit` |
| **Linux Capabilities** | `nsenter`, `setcap`, `capsh`, `chattr`, `setfacl`, `strace`, `ltrace`, `auditctl` |
| **macOS System** | `security`, `osascript`, `defaults`, `screencapture`, `diskutil`, `codesign`, `hdiutil` |
| **Environment Exfil** | `env`, `printenv`, `export`, `set`, `history` |
| **Password Managers** | `op` (1Password CLI), `bw` (Bitwarden), `lpass` (LastPass), `pass` |
| **Registry** | `reg.exe`, `regedit`, `wmic` — zero-touch on all registry operations |
| **Debugging** | `gdb`, `lldb`, `dtrace`, `dtruss`, `strace`, `ltrace`, `perf` |
| **Inline Execution** | `python -c`, `python3 -c`, `node -e`, `ruby -e`, `perl -e`, `php -r` |

### Behavioral Rules (enforced in every session)

| Rule | Effect |
|---|---|
| **Mandatory approval gate** | Every write, modify, delete, or Bash command requires explicit human confirmation |
| **Prompt injection immunity** | Claude must detect and report in-file instructions attempting to override its behavior |
| **Credential zero-touch** | Claude must stop and report if it encounters credential material anywhere |
| **Network isolation** | No network connections of any kind, including localhost |
| **Scope lock** | Claude operates only within the explicitly stated project directory |
| **Windows registry freeze** | Zero reads or writes to registry keys via any mechanism |
| **Dependency freeze** | No package installation, upgrade, or removal on any platform |
| **Audit trail** | Every response prefixed with `[ACTION] | [REASON] | [AWAITING]` |

---

## Example Configurations

The `examples/` directory contains minimal allow-list configurations for common project types. These extend the base deny-list with only the permissions required for each use case:

| File | Use Case |
|---|---|
| `examples/python-project.json` | Python projects with pytest, ruff, mypy, bandit |
| `examples/node-project.json` | Node.js/TypeScript projects with pre-installed npm scripts |
| `examples/readonly-audit.json` | Security audits — zero writes, zero execution, structured findings output |

Copy the relevant example to `.claude/settings.json` in your project directory.

---

## Threat Model

### In Scope — Threats This Configuration Reduces

**Prompt Injection via Codebase**
An attacker embeds malicious instructions in a README, docstring, log, test fixture, or CI config. Claude reads the file during a legitimate task and follows embedded instructions.
- *Mitigated by:* Network, credential, and persistence blocks stop the most dangerous outcomes. The `rules` block instructs Claude to detect and report embedded instructions.

**Credential Harvesting**
Claude reads `~/.aws/credentials`, `.env`, `~/.ssh/id_rsa`, Windows Credential Manager, or similar files.
- *Mitigated by:* All credential paths are explicitly denied at the permission level. Hard control, not behavioral.

**Supply Chain Tampering**
Claude installs a malicious package or modifies a CI/CD pipeline based on attacker-influenced content.
- *Mitigated by:* All package managers and CI/CD pipeline write paths are blocked.

**Cloud Session Hijacking**
Claude executes `aws`, `az`, or `kubectl` using the developer's active cloud sessions.
- *Mitigated by:* All cloud CLIs are denied at the permission level.

**Windows LOLBin Abuse**
Claude is prompted to use `certutil`, `bitsadmin`, `mshta`, or PowerShell as download or execution proxies.
- *Mitigated by:* All identified Windows Living Off the Land Binaries are denied.

**Persistence Installation**
Claude writes to shell profiles, LaunchAgents, Startup folders, systemd units, or scheduled tasks.
- *Mitigated by:* All persistence directories and scheduling commands are write-blocked.

**Linux Privilege / Capability Abuse**
Claude uses `nsenter`, `setcap`, or Linux namespace tools to escalate privileges or escape containers.
- *Mitigated by:* All Linux capability and namespace management commands are denied.

---

### Out of Scope — Residual Risk

| Risk | Recommendation |
|---|---|
| **Rules-layer bypass** | The permission deny-list is the hard control. Rules are defense-in-depth. Layer with endpoint DLP and terminal logging. |
| **Secrets inline in project files** | Deploy `gitleaks` or `trufflehog` as a pre-commit hook. Claude reads project source files. |
| **Cowork GUI automation** | Cowork operates at the GUI layer, outside `settings.json` scope entirely. Apply separate controls; restrict to non-credentialed machines. |
| **MCP connector exfiltration** | Connected MCP servers (Gmail, Atlassian, etc.) are trusted exfiltration channels. Disconnect MCP in Claude Code environments. |
| **New Claude Code features** | Review config against every Claude Code release. New tools are not automatically blocked. |
| **Glob pattern edge cases** | Validate all rules in a staging environment before fleet MDM rollout. |
| **WSL2 on Windows** | Windows Subsystem for Linux runs a separate Linux environment; apply Linux-specific rules inside WSL2 separately. |

---

## Compliance Alignment

| Framework | Relevant Controls |
|---|---|
| **SOC 2 Type II** | CC6.1 (Logical access), CC6.6 (Restrict production access), CC7.2 (System monitoring) |
| **ISO 27001:2022** | A.8.2 (Privileged access), A.8.15 (Logging), A.8.25 (Secure development lifecycle) |
| **NIST AI RMF** | GOVERN 1.1, MANAGE 2.2 (Risk treatment for AI systems) |
| **NIST SP 800-53** | AC-6 (Least privilege), AU-2 (Audit events), SI-3 (Malicious code protection) |
| **CIS Controls v8** | Control 3 (Data protection), Control 6 (Access control management), Control 16 (Application security) |
| **PCI-DSS v4.0** | Req. 7 (Restrict access to system components), Req. 10 (Log and monitor) |
| **HIPAA** | §164.312(a)(1) (Access control), §164.312(b) (Audit controls) |

---

## Recommended Companion Controls

This configuration is most effective as one layer in a defense-in-depth stack:

- **Pre-commit secret scanning:** [`gitleaks`](https://github.com/gitleaks/gitleaks), [`trufflehog`](https://github.com/trufflesecurity/trufflehog)
- **Endpoint DLP:** CrowdStrike, Zscaler, Microsoft Purview — alert on `curl`/`wget` to unapproved domains
- **Terminal session logging:** macOS Unified Logging, Windows Event Forwarding, auditd on Linux
- **SIEM integration:** Ingest Claude Code session logs for anomaly detection on unusual command proposals
- **Git commit signing:** GPG-signed commits ensure AI-generated changes are attributable
- **Dev containers:** Run Claude Code inside a Dev Container or Lima VM with no credential directory mounts
- **MCP OAuth scoping:** If MCP connectors are required, scope tokens to minimum necessary permissions (e.g., Gmail compose-only, no read)

---

## Versioning & Maintenance

This repository tracks Claude Code releases. When Anthropic ships new features or tools, the configuration is reviewed and updated. **Watch this repository** for release notifications.

| Config Version | Claude Code Era | Platforms | Last Reviewed |
|---|---|---|---|
| v1.0.0 | Claude 4.x | macOS, Windows, Linux | 2026-03 |

> Pin your MDM deployment to a tagged release (`v1.0.0`) rather than `main` so your fleet doesn't receive untested configuration changes automatically.

---

## Contributing

Contributions are welcome. Priority areas:

- Missing tools, CLIs, or credential paths for any platform
- MDM deployment guides (Jamf, Intune, Mosyle, Ansible, GPO)
- Additional example configs for Java, Go, Rust, and other ecosystems
- CI/CD GitHub Action to validate the config on PRs
- WSL2-specific hardening guidance

Please read [CONTRIBUTING.md](CONTRIBUTING.md) before submitting. All changes are reviewed for security implications before merging. Report bypass vulnerabilities via [SECURITY.md](SECURITY.md) — not as public issues.

---

## License

MIT — see [LICENSE](LICENSE). Free for commercial and non-commercial use.

---

## Acknowledgments

Built by security engineers who identified these risks through direct security testing of Claude Code, Claude Cowork, and Claude Chat in enterprise environments with Claude Teams and Claude for Work licenses.

---

*Searching for: claude code security hardening | claude code enterprise policy | claude code settings.json restrict | harden claude ai | claude code prompt injection protection | claude code windows security | claude code linux hardening | claude code macos lockdown | claude code least privilege | anthropic claude code devsecops | claude code managed policy MDM | claude code teams license security configuration | ai agent security enterprise | llm agent hardening | claude code credential protection*
