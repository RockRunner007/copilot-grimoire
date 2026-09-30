# Repository Skills

Skills discoverable by GitHub Copilot/agents must live at `.github/skills/<name>/SKILL.md`
with YAML frontmatter containing `name` and a `description` written as "Use when...".
Files named `skill.md` (lowercase) or placed outside this convention are **not** discovered.

## Available Skills

- [`appsec-remediation`](appsec-remediation/SKILL.md): Triage or remediate a specific GHAS
	finding surfaced by the user.
- [`ghas-reporting`](ghas-reporting/SKILL.md): Collect or export scoped GHAS counts for
	specified repositories or applications.

## Template

Copy this when adding a new skill:

```markdown
---
	name: my-skill-name
description: Use when the user asks to <trigger phrase>, <trigger phrase>, or <trigger phrase>.
---

# My Skill Name

## What it does
...

## Instructions for Copilot
1. ...
```

Use lowercase letters, numbers, and hyphens for new skill names. The `name` must match the skill
directory name; see [the Copilot customization baseline](../../specs/skills/readme.md) for the
company-wide conventions and compatibility note on existing skill IDs.
