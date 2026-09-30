---
name: workflow-conventions
description: "Use when adding or editing GitHub Actions workflows in .github/workflows/. Covers avoiding duplicate logic, naming, and secret hygiene."
applyTo: ".github/workflows/**"
---

# Workflow Conventions for This Repo

- Before adding a workflow, check whether the code and configuration it would invoke exist in
  this checkout. Reuse an existing implementation when available; avoid copying application
  logic into workflow steps.
- Name workflow files descriptively and without typos. Delete empty or abandoned workflow files
  rather than leaving them as placeholders.
- Never echo a secret's value to logs. Printing whether a secret/var is configured
  (`secrets.FOO != ''`) is fine; printing the value itself is not.
- If a workflow's `schedule:` trigger is commented out, call that out in the workflow's `name`
  or a comment so its actual cadence (manual-only) isn't assumed to be automatic elsewhere
  (e.g. in README.md).
