# Security Policy

## Scope

This repository contains security hardening configurations for Claude Code. A **security vulnerability** in the context of this project means:

- A bypass technique that allows Claude Code to access a path, execute a command, or perform a network operation that the deny-list is intended to block
- A glob pattern gap or syntax edge case that leaves a credential path or dangerous command unprotected
- A `rules` block bypass where Claude can be prompted to rationalize ignoring a behavioral control
- A platform-specific vector (macOS, Windows, Linux) not covered by the current configuration
- A new Claude Code feature or tool that introduces capabilities not addressed by the current deny-list

**Out of scope:**
- Vulnerabilities in Claude Code itself (report those directly to Anthropic at https://www.anthropic.com/security)
- General questions about how Claude Code works
- Feature requests or configuration suggestions (open a standard GitHub Issue instead)

---

## Reporting a Vulnerability

**Please do not open a public GitHub Issue for security vulnerabilities.** Public disclosure before a fix is available increases risk for all organizations using this configuration.

### How to Report

1. **Email:** Send a detailed report to the maintainers via the email address listed in the repository's GitHub profile, with the subject line: `[claude-code-hardened] Security Vulnerability Report`

2. **GitHub Private Vulnerability Reporting:** Use GitHub's built-in private reporting feature:
   - Navigate to the **Security** tab of this repository
   - Click **Report a vulnerability**
   - Complete the advisory form

### What to Include

Please include as much of the following as possible:

- **Description:** Clear explanation of the bypass or gap
- **Platform:** macOS / Windows / Linux (or all)
- **Claude Code version:** The version you tested against
- **Reproduction steps:** Exact steps to reproduce the issue
- **Proof of concept:** A minimal example demonstrating the gap (sanitized — do not include real credentials or sensitive data)
- **Impact assessment:** What an attacker could achieve by exploiting this gap
- **Suggested fix:** If you have a proposed rule or pattern change

### Response Timeline

| Stage | Target Timeframe |
|---|---|
| Acknowledgment of report | 48 hours |
| Initial severity assessment | 5 business days |
| Fix developed and reviewed | 14 business days (critical), 30 days (medium/low) |
| Public disclosure | After fix is merged and released, coordinated with reporter |

We follow **coordinated disclosure**. We will work with you to agree on a disclosure timeline and will credit you in the release notes unless you request otherwise.

---

## Severity Classification

We classify vulnerabilities using the following criteria:

| Severity | Description | Example |
|---|---|---|
| **Critical** | Allows credential access or remote code execution, bypasses both permission layer and rules layer | Glob pattern that misses `~/.aws/credentials` on a specific OS |
| **High** | Allows access to sensitive system paths or execution of a blocked command class | A package manager variant not in the deny-list |
| **Medium** | Partial bypass with limited impact, or requires user interaction to exploit | A behavioral rules bypass that still requires a cooperative user |
| **Low** | Minor gap with negligible real-world impact | A rarely-used legacy tool missing from the deny-list |

---

## Maintained Versions

| Version | Supported |
|---|---|
| `main` (latest) | ✅ Active |
| Tagged releases | ✅ Critical fixes backported |
| Forks | ❌ Not maintained by this project |

---

## Acknowledgments

We maintain a Hall of Fame for researchers who responsibly disclose vulnerabilities. Contributors will be credited in `CHANGELOG.md` and the GitHub Security Advisories unless anonymity is requested.
