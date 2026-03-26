# Changelog

All notable changes to `claude-code-hardened` are documented here.

This project follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) and uses [Semantic Versioning](https://semver.org/):

- **MAJOR** version — breaking changes to the config schema or structure
- **MINOR** version — new deny rules, new platform coverage, new example configs
- **PATCH** version — glob pattern fixes, documentation corrections, typos

Security vulnerability fixes are tagged `[SECURITY]` and always released as the highest priority.

---

## [1.0.0] — 2026-03

### Added — Initial Release

**Core Configuration**
- `settings.json`: User/project-level hardened configuration with maximum-restriction deny-list and mandatory approval gate rules
- `managed_settings.json`: Enterprise MDM-deployable policy that cannot be overridden by user or project settings

**macOS Coverage**
- Credential path lockout: `~/.ssh`, `~/.aws`, `~/.azure`, `~/.config/gcloud`, `~/.kube`, `~/.gnupg`, `~/Library/Keychains`, 1Password, Bitwarden
- Persistence vector blocks: `~/Library/LaunchAgents`, `~/Library/LaunchDaemons`, shell profiles
- macOS-specific command blocks: `security`, `osascript`, `defaults`, `screencapture`, `diskutil`, `hdiutil`, `codesign`, `pbcopy`, `pbpaste`
- Package manager blocks: `brew install/upgrade/uninstall`

**Windows Coverage**
- Credential path lockout: `AppData/Roaming/Microsoft/Credentials`, `Windows/System32/config` (SAM), GitHub CLI, 1Password, Bitwarden, PowerShell profiles
- Registry protection: `reg.exe`, `regedit`, `wmic` blocked
- LOLBin blocks: `certutil`, `bitsadmin`, `mshta`, `wscript`, `cscript`, `rundll32`, `regsvr32`, `msiexec`
- Scripting host blocks: `powershell`, `pwsh`, `cmd.exe`
- Persistence vector blocks: Startup folders, `schtasks`, `sc.exe` (service creation), `vssadmin`
- Lateral movement blocks: `psexec`, `winrm`, `runas`, `net user/localgroup/share/use`, `netsh`
- Package manager blocks: `winget`, `choco`, `scoop`
- Privilege escalation blocks: `icacls`, `takeown`, `auditpol`, `secedit`, `gpupdate`

**Linux Coverage**
- Credential path lockout: `/home/*/.ssh`, `/home/*/.aws`, `/home/*/.kube`, `/home/*/.gnupg`, `/home/*/.local/share/keyrings`, `/root/*` equivalents
- Process memory protection: `/proc/*/environ`, `/proc/*/cmdline`
- Persistence vector blocks: `/etc/crontab`, `/etc/cron.d/*`, `/etc/systemd/system/*`, `/var/spool/cron/*`, `update-rc.d`, `chkconfig`, `systemd-run`, `at`
- Privilege/capability command blocks: `nsenter`, `unshare`, `capsh`, `setcap`, `getcap`, `chattr`, `setfacl`, `usermod`, `useradd`, `passwd`
- Audit/forensic tool blocks: `strace`, `ltrace`, `perf`, `auditctl`, `ausearch`, `journalctl`
- Package manager blocks: `snap`, `flatpak`, `dpkg`, `rpm`, `zypper`, `pacman`, `apt`, `apt-get`, `yum`, `dnf`

**Cross-Platform**
- All cloud CLIs blocked: `aws`, `az`, `gcloud`, `kubectl`, `helm`, `eksctl`, `terraform`, `vault`, `consul`, `nomad`, `packer`
- All container runtimes blocked: `docker`, `podman`, `containerd`, `minikube`, `kind`, `k3d`
- All language package managers blocked: `pip`, `npm`, `yarn`, `pnpm`, `bun`, `gem`, `cargo`, `go get`, `composer`
- Network tools blocked: `curl`, `wget`, `nc`, `socat`, `nmap`, `tcpdump`, `ssh`, `scp`, `rsync`, `ftp`, `telnet`
- Privilege escalation blocked: `sudo`, `su`, `doas`, `pkexec`, `chmod`, `chown`
- Secret file types blocked: `.env`, `.pem`, `.key`, `.p12`, `.pfx`, `.jks`, `id_rsa*`, `id_ed25519*`, `.ppk`, `.rdp`, `terraform.tfstate`
- CI/CD pipeline write protection: `.github/workflows`, `.circleci`, `.gitlab-ci`, `Jenkinsfile`
- Password manager CLI blocks: `op` (1Password), `bw` (Bitwarden), `lpass` (LastPass), `pass`
- `WebFetch(*)` blocked
- Inline code execution blocked: `python -c`, `python3 -c`, `node -e`, `ruby -e`, `perl -e`, `php -r`
- Environment variable exfiltration blocked: `env`, `printenv`, `export`, `set`, `history`
- Debugging/tracing tool blocks: `gdb`, `lldb`, `dtrace`, `dtruss`, `strace`

**Behavioral Rules (all platforms)**
- Mandatory approval gate before every write, modify, delete, or Bash command
- Prompt injection detection and reporting requirement
- Credential zero-touch policy
- Network isolation mandate
- Scope lock to current project directory
- Windows registry zero-touch policy
- CI/CD and dependency freeze
- Structured audit trail prefix on every response

**Repository Files**
- `README.md`: Full project documentation, threat model, compliance alignment, deployment instructions
- `SECURITY.md`: Responsible disclosure policy and severity classification
- `CONTRIBUTING.md`: Contribution guidelines and PR process
- `CHANGELOG.md`: This file
- `examples/python-project.json`: Minimal allow-list for Python projects
- `examples/node-project.json`: Minimal allow-list for Node.js projects
- `examples/readonly-audit.json`: Zero-write, read-only audit mode

---

## Upcoming / Roadmap

| Item | Target Version | Notes |
|---|---|---|
| Jamf deployment guide | 1.1.0 | Step-by-step MDM profile creation |
| Intune deployment guide | 1.1.0 | Windows ADMX/JSON policy deployment |
| Ansible playbook for Linux | 1.1.0 | Automated deployment for Linux fleets |
| CI/CD validation workflow | 1.1.0 | GitHub Action to lint config on PR |
| Java/Maven example config | 1.2.0 | `examples/java-maven.json` |
| Go project example config | 1.2.0 | `examples/go-project.json` |
| WSL2-specific hardening notes | 1.2.0 | Windows Subsystem for Linux edge cases |

---

*To report a security vulnerability, see [SECURITY.md](SECURITY.md). To suggest a feature or new rule, open a GitHub Issue.*
