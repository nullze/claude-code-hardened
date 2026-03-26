# Contributing to claude-code-hardened

Thank you for helping make AI-assisted development safer for everyone. This project is maintained by security engineers and contributions are reviewed with a security-first lens.

---

## What We Welcome

| Contribution Type | Notes |
|---|---|
| **New deny rules** | Missing tools, CLIs, package managers, or credential paths |
| **Platform coverage** | Windows, Linux, or macOS gaps |
| **Example allow-list configs** | Language- or framework-specific minimal allow-lists in `examples/` |
| **MDM deployment guides** | Jamf, Intune, Mosyle, Ansible, or GPO deployment documentation |
| **Bypass disclosures** | Via SECURITY.md — not via PRs or public issues |
| **Documentation improvements** | Clarity, accuracy, new threat scenarios |
| **CI/CD validation tooling** | Scripts that test the config against known-bad patterns |

---

## What We Do Not Accept

- Rules that weaken or loosen the default deny posture without an exceptional documented justification
- Allow-list entries in `settings.json` or `managed_settings.json` — the base configs stay at maximum restriction; permissive examples belong in `examples/` only
- Contributions that add real credentials, internal hostnames, or PII to any file
- Changes to `managed_settings.json` that remove rules without a documented threat model rationale reviewed by a maintainer

---

## Contribution Process

### 1. Open an Issue First

Before writing a PR, open a GitHub Issue describing:

- What tool, path, or command you want to add or change
- The platform it applies to (macOS / Windows / Linux / cross-platform)
- The threat scenario it closes (what could an attacker do without this rule?)
- Any known side effects on legitimate developer workflows

This avoids wasted effort and lets maintainers flag concerns early.

### 2. Fork and Branch

```bash
git clone https://github.com/YOUR_ORG/claude-code-hardened.git
cd claude-code-hardened
git checkout -b feat/block-xyz-tool
```

Use a descriptive branch name:
- `feat/block-<toolname>` — new deny rule
- `fix/glob-<platform>` — pattern fix
- `docs/<section>` — documentation
- `examples/<language>` — new example config

### 3. Make Your Changes

**For deny rule additions:**
- Add the rule to both `settings.json` AND `managed_settings.json`
- Place it in the correct logical section (credentials, network, cloud CLIs, package managers, etc.)
- Add a comment in the PR description explaining which threat it closes

**For example configs (`examples/`):**
- Only use `allow` entries — never remove from the `deny` list
- Keep allow-lists as narrow as possible (specific commands, not wildcards)
- Include a header comment in the JSON explaining the use case and assumptions

**For documentation:**
- Update the threat model table in README.md if you are adding a new threat scenario
- Update the "Residual Risk" table if relevant
- Update the version table in README.md's versioning section

### 4. Test Your Changes

Before submitting, manually verify your changes using the following checklist:

```
[ ] Rule added to both settings.json and managed_settings.json
[ ] Glob pattern tested against the target path/command on the target OS
[ ] No existing allow-list entries broken by the new rule
[ ] JSON is valid (run: cat settings.json | python3 -m json.tool)
[ ] README threat model or residual risk table updated if applicable
[ ] No credentials, PII, or internal hostnames in any file
```

### 5. Submit a Pull Request

Use this PR template:

```markdown
## Summary
One paragraph describing what this PR adds or changes.

## Threat Scenario Closed
Describe what an attacker could do without this rule.

## Platform
[ ] macOS  [ ] Windows  [ ] Linux  [ ] Cross-platform

## Files Changed
- settings.json — added deny rule for X
- managed_settings.json — added deny rule for X
- README.md — updated threat model table

## Testing
How you validated the glob pattern or rule works as intended.

## Residual Risk
Any known gaps or edge cases this PR does NOT close.
```

---

## Code Style and Conventions

**JSON formatting:**
- 2-space indentation
- Rules grouped by category with a blank line and comment between groups
- Alphabetical ordering within each group where possible

**Rule naming:**
- Use exact command names as they appear on the target OS
- Include both the bare name and `.exe` suffix for Windows binaries: `"Bash(reg *)"` and `"Bash(reg.exe *)"`
- Use `*` for wildcard arguments, never omit the space before `*` for commands with arguments

**Commit messages:**
```
feat(deny): block <tool> on <platform>
fix(glob): correct path pattern for <credential> on Windows
docs: add MDM deployment guide for Jamf
examples: add minimal allow-list for Python projects
```

---

## Review Criteria

All PRs are reviewed against:

1. **Does this close a real threat?** Deny rules must map to a documented abuse technique.
2. **Is the glob pattern correct?** We test patterns against real paths on all three platforms before merging.
3. **Does it break legitimate workflows?** Rules that block common safe operations without justification won't merge.
4. **Is it consistent with the project's least-privilege philosophy?** When in doubt, we err toward more restriction.
5. **Is it documented?** New threat scenarios must be reflected in the README.

Maintainers aim to review PRs within **7 business days**. Security-relevant fixes are prioritized.

---

## Development Environment

No special tooling required — this is a JSON configuration project. Recommended local setup:

```bash
# Validate JSON syntax
cat settings.json | python3 -m json.tool > /dev/null && echo "✅ valid"
cat managed_settings.json | python3 -m json.tool > /dev/null && echo "✅ valid"

# Check for accidental secrets
gitleaks detect --source . --verbose
```

---

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). Security contributions are especially valued, and all reporters and contributors will be treated with respect regardless of experience level.
