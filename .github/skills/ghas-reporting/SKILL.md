---
name: ghas-reporting
description: "Use when the user asks to generate, pull, preview, summarize, export, or troubleshoot a GHAS metrics report for one repository, a group of repositories, or an application."
---

# Scoped GHAS Reporting

## What this skill does

Generate or export a scoped GHAS metrics sample for one repository or an application (represented by its
repositories). The report includes open Code Scanning, Dependabot, and Secret Scanning counts,
Code Scanning severity counts, scan configuration status, and the collection timestamp. It works
in any GitHub repository with `gh` CLI access; it does not require this repository's scripts,
dashboard, SharePoint configuration, or output format.

This skill reports aggregate metrics. For remediation of a specific finding, use
`appsec-remediation` or `/appsec` instead.

## Export mode

Use `--export` when the user wants a saved report rather than a summary in chat. Export mode uses
the same collection and error rules as normal reporting, then writes only the requested fields to
a user-approved CSV or JSON path outside the repository or under an ignored directory. It must:

- Confirm the scope and output format/path before writing.
- Include report metadata, repository scope, collection timestamp, product counts, Code Scanning
   severity counts, setup status, and per-repository endpoint errors.
- Preserve unavailable values as `unavailable` or an explicit error object; never convert them to
   zero or silently omit them.
- Avoid alert-level secret values and avoid writing credentials, tokens, or unnecessary alert
   locations to the export.
- Never publish to SharePoint or modify dashboard artifacts unless the user explicitly requests
   that AppSec integration.

## Required inputs

- A repository's full name, such as `owner/repository`; or
- An application name and the full names of all repositories that implement it.

An application is not a first-class field in the metrics schema. Do not infer its repository
membership from naming alone; ask the user for the repository list if it is not already known.

## Instructions for Copilot

1. Confirm the requested scope. For a repository, require its full `owner/repository` name. For
   an application, require the application's name and each owned `owner/repository` name.
2. State that the report contains open-alert counts and scan status, not alert-level remediation
   details. Offer `appsec-remediation` only if the user wants to investigate a particular alert.
3. Verify that `gh auth status` has an authenticated account with read access to the requested
   repositories and GHAS alert APIs. Prefer the user's current authenticated CLI/session context;
   never ask the user to paste a token into chat or print its value. Do not use browser collection
   unless the user explicitly asks for it as a last resort. If CLI/session access is unavailable
   and browser collection was not explicitly requested, stop and report the access blocker rather
   than silently switching collection methods.
4. Use `gh api --paginate` against each scoped repository's Code Scanning,
   Dependabot, and Secret Scanning alerts endpoints, requesting only `state=open`. For Code
   Scanning, group alert `rule.security_severity_level` values into critical, high, and medium.
   Query `code-scanning/default-setup` separately to determine configuration status.
5. Summarize the result in chat by default. Only write JSON or CSV when the user asks for a saved
   artifact. In `--export` mode, confirm the user-approved output path and format, then write the
   report there; put it in an ignored or external location.
6. Report the scanned repository count, repositories with findings, totals by alert type, Code
   Scanning critical/high/medium counts, and any endpoint failures. Clearly distinguish zero
   findings from a `403`, `404`, or other incomplete-data result.
7. For an application, combine the results only after every listed repository was processed; name
   unavailable metrics by repository rather than treating them as zero.

## Last-resort browser fallback

Only use this path when the user explicitly authorizes browser collection after CLI/session access
has failed. Reuse the one authenticated GitHub browser tab for each repository. Navigate it
sequentially through these paths and record the displayed `Open` count; do not open additional
browser tabs:

- `/OWNER/REPOSITORY/security/code-scanning?query=is%3Aopen`
- `/OWNER/REPOSITORY/security/dependabot?query=is%3Aopen`
- `/OWNER/REPOSITORY/security/secret-scanning?query=is%3Aopen`
- `/OWNER/REPOSITORY/settings/security_analysis`

Record Code Scanning tool failures separately from alert totals. The settings page establishes
whether GHAS features are enabled, but may not expose the REST API's exact default-setup state;
report that field as unavailable rather than inferring it. Return the tab to the repository page
when collection finishes.

## Portable GitHub CLI example

```powershell
$repo = "owner/repository"
$codeScanning = gh api --paginate --slurp "/repos/$repo/code-scanning/alerts?state=open&per_page=100" | ConvertFrom-Json
$dependabot = gh api --paginate --slurp "/repos/$repo/dependabot/alerts?state=open&per_page=100" | ConvertFrom-Json
$secretScanning = gh api --paginate --slurp "/repos/$repo/secret-scanning/alerts?state=open&per_page=100" | ConvertFrom-Json
$defaultSetup = gh api "/repos/$repo/code-scanning/default-setup" | ConvertFrom-Json
```

For an application, repeat the scoped requests for every owned repository and aggregate only the
successful responses. A `gh api` `403` indicates insufficient permissions; `404` can mean that
the alert feature is unavailable for the repository.
