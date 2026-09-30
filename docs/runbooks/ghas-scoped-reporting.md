# Runbook: Scoped GHAS Sample Reporting

## Summary

Use `/report` in Copilot Chat to generate or export an on-demand, read-only GHAS metrics sample for one
repository or for an application represented by one or more repositories. The skill is portable:
it uses GitHub CLI/API access and can be copied to another repository without its reporting
scripts, dashboard, shared allowlist, or SharePoint configuration.

## Components

- Skill: `.github/skills/ghas-reporting/SKILL.md`
- Slash command: `.github/prompts/report.prompt.md` (run as `/report`)
- Default collector: GitHub CLI `gh api`

## Prerequisites

- GitHub CLI installed and authenticated with `gh auth login` using an account with read access to
  the requested repositories and GHAS alert APIs.
- Full repository names for the requested scope, such as `owner/repository`. For an application,
  provide every repository that implements it.

Never paste a token into Copilot Chat or commit it to a repository.

## Copilot workflow

1. Run `/report --repo owner/repository` for a single repository.
2. Run `/report --app application-name` for an application. Provide all full repository names
   that implement the application when prompted. Application-to-repository ownership is not
   stored in the metrics schema, so Copilot must not infer it by naming convention.
3. Add `--export csv <path>` or `--export json <path>` when a saved artifact is needed. Confirm
  the destination before writing; use an ignored or external location.
4. Copilot queries each repository through the GitHub API and returns a summary in chat, or writes
  the requested sanitized JSON/CSV export.
5. Review the returned summary, especially `status` and `api_errors`. A `403` or other API error
   means the affected metric is incomplete; it does not establish that the count is zero.

## Last-resort browser fallback

Use browser collection only when CLI/current-session access has failed and the user explicitly
authorizes it as a last resort. If authorized, share one SSO-authenticated browser tab for the
repository. Copilot can collect the displayed open counts by sequentially navigating that one tab
through the Code Scanning, Dependabot, Secret Scanning, and Security analysis views. It must not
open additional tabs. This result must be labeled browser-collected: GitHub's settings UI may not
reveal the REST API's exact Code Scanning default-setup state. Return the tab to the repository
page after collection.

## Manual workflow

Use this PowerShell sequence from any directory. Replace the repository name with the requested
scope; repeat it for each repository belonging to an application.

```powershell
$repo = "owner/repository"
$codeScanning = gh api --paginate --slurp "/repos/$repo/code-scanning/alerts?state=open&per_page=100" | ConvertFrom-Json
$dependabot = gh api --paginate --slurp "/repos/$repo/dependabot/alerts?state=open&per_page=100" | ConvertFrom-Json
$secretScanning = gh api --paginate --slurp "/repos/$repo/secret-scanning/alerts?state=open&per_page=100" | ConvertFrom-Json
$defaultSetup = gh api "/repos/$repo/code-scanning/default-setup" | ConvertFrom-Json

[pscustomobject]@{
  Repository = $repo
  CodeScanningOpen = @($codeScanning).Count
  CodeScanningCritical = @($codeScanning | Where-Object { $_.rule.security_severity_level -eq "critical" }).Count
  CodeScanningHigh = @($codeScanning | Where-Object { $_.rule.security_severity_level -eq "high" }).Count
  CodeScanningMedium = @($codeScanning | Where-Object { $_.rule.security_severity_level -eq "medium" }).Count
  DependabotOpen = @($dependabot).Count
  SecretScanningOpen = @($secretScanning).Count
  DefaultSetupState = $defaultSetup.state
}
```

Run each `gh api` call independently when troubleshooting so a `403` or `404` can be reported as
incomplete data rather than a zero count. Export the final `PSCustomObject` with `Export-Csv` only
when a saved artifact is necessary, and save it outside a repository or in an ignored directory.

## Interpreting the report

- `total_code_scanning_open`, `total_dependabot_open`, and `total_secret_scanning_open` count
  currently open findings by product.
- `total_code_scanning_critical`, `total_code_scanning_high`, and
  `total_code_scanning_medium` are Code Scanning severity breakdowns only.
- `scan_status` describes Code Scanning default setup discovery: `configured`, `pending`,
  `not_supported`, `error`, or `unknown`.
- `status` records collector failures for a repository. `api_errors` records alert endpoints the
  token could not access, such as `secret_scanning:403`.

Use `/appsec` for a specific alert's remediation or dismissal workflow. This reporting flow
summarizes metrics and does not retrieve enough alert-level detail to recommend a fix.

## Export rules

Exports include scope, timestamp, product counts, Code Scanning severity counts, setup status, and
endpoint errors. They preserve unavailable values explicitly and never include tokens, secret
values, or unnecessary alert-level locations. The default output is chat-only; SharePoint and
dashboard publication require a separate explicit request.

## Troubleshooting

| Symptom | Likely cause and action |
|---------|--------------------------|
| GitHub CLI/current session is unavailable | Run `gh auth login` and confirm access with `gh auth status`; restore the current session before considering browser collection. Do not expose the token in chat. |
| No results for a repository | Verify its full name and that the authenticated account can access it. |
| `api_errors` includes `*:403` | Grant the token read access to the affected GHAS alert type, then rerun. Keep the report marked incomplete until access is available. |
| A `gh api` request returns `404` | The alert feature may be unavailable for the repository, or the account cannot see it. Report the metric as unavailable. |
