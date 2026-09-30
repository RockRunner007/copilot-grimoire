# Project Guidelines

## Repository Purpose

This checkout contains reusable Copilot guidance for triaging specific GitHub Advanced Security
(GHAS) findings and collecting scoped GHAS reports. It does not currently contain the former
metrics/ticketing pipeline, workflows, or their configuration and tests. See [README.md](../README.md)
for the retained capabilities.

## Validation

- Do not assume a test suite, dependency manifest, or pipeline implementation exists in this
  checkout. Inspect the current files before selecting commands or recommending code changes.
- Never ask the user to paste a GitHub token or other credential into chat.

## Third-Party Dependencies

- Do not add a new third-party package to a dependency manifest, lockfile, workflow, or
  installation command without the user's explicit approval. Installing dependencies already
  declared by the repository does not require this additional approval.
- Before requesting approval, explain why the standard library and existing dependencies are
  insufficient. Identify the proposed package and version, compatibility with supported runtime
  versions, license, maintenance and release health, known security advisories, and expected
  direct and transitive dependency impact.
- Prefer the smallest supported stable version change that satisfies the requirement. Do not use
  prerelease packages unless the user explicitly approves them, and do not delay a security fix
  merely to remain one release behind the latest version.

## Skills

- Reusable Copilot skills live in `.github/skills/<name>/SKILL.md` (see
  [.github/skills/README.md](./skills/README.md) for the required frontmatter format).
- Use kebab-case for skill directory and frontmatter names; they must match.
- The retained skills are `appsec-remediation` and `ghas-reporting`.

## Copilot Customizations

- Use repo-wide or path-specific instructions for standing rules, prompts for focused
  parameterized tasks, skills for reusable multi-step procedures, and custom agents when a role
  needs distinct tools or an isolated workflow.
- Define workspace agents in `.github/agents/*.agent.md`. Give each a specific description and
  explicitly allow only the minimum tools it needs; omit edit/execute tools for read-only roles.
- Do not add MCP servers or credentials without explicit approval. If an integration is approved,
  allowlist the required tools, prefer read-only access, and use approved secret settings.
- The workspace GitHub MCP is configured read-only in `.vscode/mcp.json`; do not broaden its
  toolsets or enable write access without explicit approval.
- Prompts, skills, and agent instructions are guidance, not security enforcement. Use platform
  permissions and repository controls for access restrictions and merge requirements.

## Specification Placement

- Store all new requirements, feature specifications, architecture decisions, integration designs, data contracts, and workflow plans under `specs/`.
- Use the closest existing area under `specs/`; create a clearly named area when none applies.
- Keep each area's overview in its `readme.md`; place detailed topics in descriptively named Markdown files beside it.
- Use `docs/` for operator runbooks, end-user instructions, and reference documentation rather than specifications.
- Do not relocate existing documents unless the task explicitly requires it.
