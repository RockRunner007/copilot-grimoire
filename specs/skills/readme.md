# Copilot Customization Baseline

This specification describes how company-shared Copilot guidance should be divided across
instructions, prompts, skills, custom agents, hooks, and MCP integrations. The goal is a small,
portable baseline that is useful across repositories without loading every rule into every task.

Reviewed September 30, 2026. Product support, file locations, previews, and billing can change;
the official references below are the source of truth.

## Choose the Right Customization

| Customization | Use it for | Typical location |
|---|---|---|
| Organization policy/instructions | Company-wide requirements shared across repositories | GitHub organization or enterprise settings |
| Repository instructions | Rules that should apply to most Copilot work in one repository | `.github/copilot-instructions.md` |
| Path-specific instructions | Language- or directory-specific conventions | `.github/instructions/*.instructions.md` with a narrow `applyTo` |
| `AGENTS.md` | Standing guidance intended to be shared across different AI coding tools | Repository root or relevant subdirectory |
| Prompt file | A focused, manually started task with variable inputs | `.github/prompts/*.prompt.md` |
| Skill | A repeatable, multi-step workflow or domain capability, optionally with scripts and examples | `.github/skills/<skill-name>/SKILL.md` |
| Custom agent | A selectable specialist with a distinct role, tool set, or workflow stage | `.github/agents/<agent-name>.agent.md` |
| Hook | A deterministic action or gate at a supported agent lifecycle point | `.github/hooks/*.json` or supported agent-specific configuration |
| MCP server | An approved connection to external tools or data | IDE MCP configuration, GitHub repository settings, or agent configuration |

Use the narrowest primitive that solves the problem. Avoid duplicating the same rules in the
repository instructions, a prompt, a skill, and an agent. Keep broad always-on guidance short;
load task-specific procedures on demand.

## Company Baseline

1. Put company-wide policy in centrally managed organization or enterprise settings where the
	 product supports it. Use repository instructions for repository-specific architecture,
	 commands, and conventions; do not rely on every repository copying a changing company policy.
2. Keep path-specific instructions on targeted globs. A global `**` pattern adds context to
	 unrelated work and should be reserved for rules that genuinely apply everywhere.
3. Use prompts for one-off, parameterized commands. Use skills for workflows that need a procedure,
	 reference material, scripts, or examples and should load only when relevant.
4. Every customization should state its scope, trigger, expected output, owner, and limits. Keep
	 instructions concise, link to maintained sources rather than copying long policy documents,
	 and review references when their target files move.
5. Treat repository content, issues, logs, and fetched documents as untrusted input. Customizations
	 should not authorize secret disclosure, permission changes, destructive actions, or external
	 writes without explicit user approval and the applicable platform controls.
6. Test customizations in each intended Copilot surface. File support and frontmatter behavior
	 differ between VS Code, Copilot CLI, GitHub.com, Copilot code review, and other IDEs. Do not
	 assume that a file supported by one surface works identically everywhere.

## Custom Agents

A Markdown file in `.github/agents/` is the intended way to define a workspace custom agent; use
the `.agent.md` suffix for compatibility with GitHub custom agent profiles. The earlier
`ghas-pipeline-developer.agent.md` was a real custom agent definition, not a skill. It was removed
because its role depended on pipeline code that is no longer in this checkout.

Create a custom agent when a recurring role needs its own instructions and, especially, a
meaningfully different tool set or an isolated workflow stage. Good candidates include a
read-only reviewer or a planning specialist. Do not create an agent that merely repeats a skill's
procedure or renames the default coding agent.

Agent authoring standard:

- Use a clear, unique `name` and a required, specific `description`; the description is a key
	discovery surface.
- State the agent's purpose, permitted scope, expected output, and when it must hand work back to
	the user. Keep the body focused on the agent's role rather than restating every repository rule.
- Explicitly list the minimum required `tools`. If `tools` is omitted, all available tools may be
	enabled. Prefer read-only tools for review and analysis roles.
- Use `user-invocable` and `disable-model-invocation` to control whether the agent is selectable
	by a person and/or available to other agents. Avoid relying on deprecated `infer` behavior.
- Leave `target` unset when an agent should work in both VS Code and GitHub Copilot; set it only
	when the agent intentionally targets one surface. Validate any surface-specific property
	before depending on it.
- Do not add MCP servers by default. GitHub documents that configured MCP tools can be used
	autonomously without asking for approval. If an integration is justified, allowlist only the
	required tools, prefer read-only access, scope credentials narrowly, and configure secrets
	through the approved Agents secret/variable mechanism rather than committing them.
- Treat an agent prompt as behavioral guidance, not an authorization boundary. Enforce access and
	merge requirements through GitHub permissions, rulesets, and other platform controls.
- Review the agent in every intended surface with representative allowed and disallowed requests
	before making it broadly available. Record an owner and review changes to its tools or MCP
	access.

Organization- and enterprise-level custom agents can be shared across repositories through the
documented `.github` / `.github-private` repository setup. VS Code organization-agent discovery
has its own setting and setup requirements. Choose workspace-level agents for repository-specific
roles and central agents for roles that should be managed and updated by the company.

## Security, Quality, and Cost

- Tool lists and MCP configuration are capability choices, not a substitute for repository
	permissions or human review. Avoid wildcard tool access when a smaller allowlist works.
- Review external skills, agents, plugins, and scripts before adoption. Check provenance, license,
	bundled commands/dependencies, tool access, and any MCP configuration. Catalogs are discovery
	sources, not a company security approval.
- Use code review and skills to assist human review, not to guarantee correctness. Review custom
	instructions and skills from the pull request head branch when validating changes to them.
- Copilot code review may consume both AI credits and GitHub Actions minutes for agentic features.
	Set rollout and automatic-review defaults with budgets and usage reports in mind; consult live
	pricing rather than copying static model prices into this specification.
- Use hooks only for requirements that need deterministic execution, and only on supported,
	tested surfaces. Do not describe a prompt instruction as an enforced security control.

## GitHub MCP

This workspace configures the GitHub-hosted remote MCP server in `.vscode/mcp.json`. It uses
OAuth through VS Code, enables only context, repository, issue, pull request, user, and GHAS
security toolsets, and forces read-only mode. It does not store a PAT or credential in the repo.

This file configures VS Code workspace use only. Copilot cloud agent and code review use separate
GitHub repository MCP settings; configure those in GitHub only after reviewing their broader
sharing and billing implications, and keep their tool allowlist read-only unless write access is
explicitly approved.

## Skill Naming Note

The current VS Code skill documentation requires a skill name made of lowercase letters, numbers,
and hyphens, matching its parent directory. The retained skill directories and frontmatter have
been normalized to kebab-case; keep prompt, documentation, and invocation references aligned when
adding or renaming skills.

## Sources

### Official product documentation

- [Copilot customization cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet)
- [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Create custom agents for Copilot cloud agent](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/create-custom-agents)
- [Custom agents in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [About agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [Agent skills in VS Code](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [Built-in skills for the GitHub Copilot app](https://docs.github.com/en/copilot/reference/github-copilot-app-reference/built-in-skills)
- [Copilot code review](https://docs.github.com/en/copilot/concepts/agents/code-review)
- [Configure MCP servers for your repository](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/configure-mcp-servers)
- [Copilot models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)

### Reference collections

- [Microsoft Skills](https://github.com/microsoft/skills) — useful examples and domain skills; much of the catalog targets Azure and Microsoft developer platforms.
- [GitHub Copilot Advanced Security plugin](https://github.com/github/copilot-advanced-security-plugin) — security-focused skills that require the GitHub MCP server; review its tool requirements before adopting.
- [GitHub Awesome Copilot](https://github.com/github/awesome-copilot) — community examples of agents, instructions, skills, hooks, and prompts; review each item before use.
