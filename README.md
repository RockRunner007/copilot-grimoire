# GHAS Copilot Guidance

This checkout contains reusable Copilot guidance for two workflows:

- [`appsec-remediation`](.github/skills/appsec-remediation/SKILL.md) helps triage and remediate
	specific GHAS findings.
- [`ghas-reporting`](.github/skills/ghas-reporting/SKILL.md) collects scoped alert counts for
	explicitly named repositories or applications.

The former metrics/ticketing pipeline, workflows, and associated tests are not included in this
checkout. See [.github/skills/README.md](.github/skills/README.md) for the skill format and index,
and [the Copilot customization baseline](specs/skills/readme.md) for company-wide guidance and
future custom-agent standards. The workspace's read-only GitHub MCP configuration is in
[.vscode/mcp.json](.vscode/mcp.json).
