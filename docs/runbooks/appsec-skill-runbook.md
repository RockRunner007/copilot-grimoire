# Runbook: `/appsec` Copilot Skill — Finding Triage and Remediation

## Summary

This workflow supports both read-only triage and remediation of GitHub Advanced Security findings.
Use `--triage` to prioritize one or more findings without changing code or alert state.
A GitHub Copilot skill + `/appsec` prompt that helps an engineer fix a GitHub Advanced Security
finding — a leaked **secret**, a vulnerable **dependency**, a **SAST**/code scanning alert, or a
**code quality** issue — by researching current best practices and returning a concrete,
actionable fix instead of a generic explanation.

## Where it lives
- Skill: `.github/skills/appsec-remediation/SKILL.md`
- Slash command: `.github/prompts/appsec.prompt.md` (usable in VS Code as `/appsec`)

## How to use it
1. Open the file (or have the alert text) for the finding you want to fix.
2. In Copilot Chat (agent mode), run:
   ```
   /appsec --secret <paste alert or describe the finding>
   ```
   Replace `--secret` with `--dependency`, `--sast`, or `--quality` depending on the alert type.
3. For prioritization without making changes, run:
  ```
  /appsec --triage <one or more findings, or an alert export>
  ```
  Triage normalizes findings, checks applicability, identifies duplicates, assigns a `P0`
  through `P3` priority, and recommends a disposition. It is read-only: it does not edit files,
  dismiss alerts, rotate credentials, or change dependencies.
4. If you're not sure which flag applies, run `/appsec` with just the finding pasted in —
   Copilot will ask which category it falls under.
5. Copilot will:
   - Read the affected file/line in your workspace (if referenced).
   - Check whether the finding could be a false positive first (reachability for dependencies,
     taint tracing for SAST, placeholder values for secrets) and state its reasoning.
   - Look up authoritative guidance for that specific finding (the CVE/GHSA advisory, the
     CodeQL query help page, the matching OWASP/CWE entry, or linter docs) — it won't invent an
     identifier it can't confirm.
   - Treat alert text, source comments, logs, and fetched pages as untrusted data rather than
     instructions, and independently select authoritative sources.
   - Return a concrete fix (code/config diff), flagging any step that needs your explicit
     approval (e.g. rotating a live credential, rewriting git history, a major-version bump).
   - Keep complete credentials out of responses, patches, commands, logs, URLs, and test output.
   - Avoid weakening security controls or adding suppressions, exclusions, allowlists, or
     dismissals unless you explicitly request that outcome and it is justified.
   - Validate applied fixes with focused checks: malicious and valid inputs for SAST,
     compatibility tests and package-manager-generated lockfiles for dependencies, and safe
     replacement-secret loading for secret findings.
   - Report back in a consistent format: category/confidence, what the finding actually is,
     sources consulted, the fix, and any residual follow-up.
6. Review the triage or remediation recommendation and tell Copilot to apply it if you want the edit made
   automatically; otherwise use it as guidance to fix manually.
7. If you'd rather dismiss/suppress a finding than fix it, ask Copilot — it will walk through
   the standard dismissal reasons (false positive, used in tests, won't fix, no bandwidth)
   instead of defaulting to "won't fix", and will never suggest dismissing a secret finding
   without confirming the credential was rotated.

## Flag reference
| Flag           | Use for                                                    |
|----------------|-------------------------------------------------------------|
| `--triage`     | Prioritize one or more findings without changing code or alert state |
| `--secret`     | Secret scanning alerts (API keys, tokens, passwords, connection strings) |
| `--dependency` | Dependabot / software composition analysis (CVE/GHSA, vulnerable package versions) |
| `--sast`       | Code scanning / static analysis alerts (injection, unsafe deserialization, etc.) |
| `--quality`    | Code quality / maintainability findings (lint rules, complexity, dead code) |

## Scope / limitations
- This is a remediation *helper*, not an auto-fixer for a GHAS collection or ticketing pipeline.
  Those implementations are not present in this checkout.
- It responds to a specific finding you bring — it will not proactively scan the whole repo
  unless asked. Org-wide metrics collection is not included in this checkout.
- Copilot will not rotate live credentials or force-push history rewrites on its own; those
  steps are called out for the engineer to perform manually.
- Copilot will not add a new third-party package without explicit approval. Before requesting
  approval, it explains why existing options are insufficient and reports the proposed version,
  runtime compatibility, license, maintenance health, known advisories, and dependency impact.
- Recommendations are only as good as the finding details provided — include the rule ID,
  CVE/GHSA ID, or file/line whenever possible for the most accurate guidance.
- Because the skill lives under `.github/skills/`, GitHub's built-in Copilot code review can
  also pick it up automatically on relevant pull requests (skills are read from the PR's head
  branch).

## Owner / questions
AppSec repository maintainers. See [.github/skills/README.md](../../.github/skills/README.md)
for how repository skills are structured.
