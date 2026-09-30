---
description: "Generate, summarize, or export a scoped GHAS metrics report for one repository or application."
agent: "agent"
argument-hint: "--repo owner/repository | --app application-name | --export csv/json path"
---
Help me generate or export a scoped GHAS metrics report.

Scope: ${input:scope:Enter --repo owner/repository, or --app application name and its full repository names}
Output: ${input:output:Optional --export csv|json and a user-approved output path}

Follow the `ghas-reporting` skill:
1. For `--repo`, confirm the repository's full owner/repository name. For `--app`, ask for the
   full repository names that belong to the application if they are not provided.
2. If `--export` is present, confirm the CSV or JSON format and output path before writing. Do not
   export secrets, credentials, or unnecessary alert-level details.
3. Use the user's current authenticated GitHub CLI/session context and GitHub API for the
   requested repositories; do not require local scripts, dashboard artifacts, or SharePoint
   configuration. Do not use browser collection unless I explicitly authorize it as a last resort.
   If CLI/session access is unavailable and I have not authorized browser collection, stop and
   report the blocker. Summarize in chat unless I ask to save a JSON or CSV artifact.
4. For every repository, collect open Code Scanning, Dependabot, and Secret Scanning alert counts
   plus Code Scanning default-setup status. Record endpoint failures separately.
5. Summarize scope, fetch timestamp, open findings by alert type, Code Scanning severity counts,
   scan status, and any API errors. Treat API errors as incomplete data, not zero findings.
6. Do not ask me to paste tokens into chat or display token values. Tell me to authenticate with
   GitHub CLI or restore the current authenticated session when access is unavailable. Only ask
   for browser authorization as a separate last-resort step.
7. Do not commit generated output. Explain that alert-level remediation belongs in `/appsec`.
