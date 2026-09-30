---
description: "Triage or remediate GitHub Advanced Security findings (secret, dependency, SAST, or code quality)."
agent: "agent"
argument-hint: "--triage | --secret | --dependency | --sast | --quality, then paste one or more findings"
---
Help me triage or fix the following Advanced Security finding(s): ${input:finding:Paste alert text, an export, rule ID/CWE/CVE, file and line, or link}

Mode/category flag: ${input:category:--triage, --secret, --dependency, --sast, or --quality}

Follow the `appsec-remediation` skill:
1. If `--triage` is present, classify and prioritize every supplied finding, identify duplicates
   and evidence gaps, and recommend a disposition. Do not edit files, dismiss alerts, rotate
   credentials, or change dependencies in triage mode.
2. If the category flag is missing or unclear, ask which of `--triage` (prioritization without
   changes), `--secret` (secret scanning),
   `--dependency` (Dependabot/SCA), `--sast` (code scanning), or `--quality` (code
   quality/lint) applies before continuing.
3. Read the actual affected file/line in this workspace if one is referenced — don't
   recommend a fix based on the alert text alone.
4. Check whether it could be a false positive first, using the per-category checks in the
   skill (reachability for dependencies, taint tracing for SAST, placeholder values for
   secrets) — state your reasoning either way.
5. Look up current best-practice guidance for this specific finding (the CVE/GHSA advisory,
   the CodeQL query help page, the relevant OWASP/CWE entry, or the linter's rule docs) using
   `fetch_webpage`, and summarize what you found. Don't fabricate an identifier you can't
   confirm.
6. Give a concrete fix (the actual code/config change), not just a generic explanation, and
   call out any step that needs my approval (credential rotation, history rewrite, dependency
   major-version bump).
7. If I ask about dismissing/suppressing instead of fixing, use the skill's dismissal-reason
   guidance (false positive, used in tests, won't fix, no bandwidth) rather than defaulting to
   "won't fix".
8. Only edit files in this workspace if I ask you to apply the fix; otherwise just show the
   recommended diff, using the skill's "Reporting the recommendation" structure (category,
   what the finding is, sources consulted, the fix, residual follow-up).
