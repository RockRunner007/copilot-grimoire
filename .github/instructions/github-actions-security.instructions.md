---
name: github-actions-security
description: "Use when reviewing, adding, or changing GitHub Actions workflows in this repo. Covers least privilege, secret safety, action pinning, and untrusted input handling."
applyTo: ".github/workflows/**"
---

# GitHub Actions Security for This Repo

- Set workflow or job `permissions` explicitly and grant only the scopes required by the steps. Treat
  `contents: write`, `id-token: write`, and `security-events: write` as sensitive permissions that
  require a documented reason.
- Pin third-party actions to immutable commit SHAs when the workflow has write access, handles
  secrets, or runs on a schedule. Keep the human-readable version in a comment beside the SHA.
- Do not place `${{ github.event.* }}` or other untrusted event data directly inside a `run:` block.
  Pass values through environment variables and validate them before using them in shell or Python.
- Keep credentialed work on trusted triggers such as `workflow_dispatch` or controlled schedules.
  Do not combine secrets with `pull_request_target` or execute fork-controlled code in a privileged job.
- Avoid inline scripts when an existing tested repository script can perform the same work. If an
  inline script is unavoidable, add focused tests or a documented validation path for its behavior.
- Use `GITHUB_TOKEN` and repository secrets through step `env`; never write token values to logs,
  artifacts, summaries, or generated reports.
- For workflows that commit generated files, constrain the changed paths, use a dedicated bot identity,
  and verify that the generated output cannot include secrets or untrusted executable content.
- Review `actions/checkout`, dependency installation, artifact upload, and Azure login steps together:
  each can expand the supply-chain or credential exposure surface even when the main script is safe.
