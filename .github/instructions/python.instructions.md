---
name: python-conventions
description: "Use when writing or editing Python scripts in this repo (GHAS export, metrics, ticketing). Covers secret handling, HTTP retry patterns, and test mocking conventions."
applyTo: "**/*.py"
---

# Python Conventions for This Repo

- Never hardcode or print tokens, passwords, or other secrets. Read credentials from the
  environment or an approved secret store; do not expose values in logs, errors, or test output.
- For HTTP calls, use existing client and error-handling patterns when available. Handle
  pagination and transient failures deliberately, and raise an appropriate error rather than
  silently returning incomplete data.
- Keep network calls out of tests by mocking the HTTP client or service boundary.
- Inspect the current checkout for tests and declared dependencies before choosing commands or
  suggesting installations. Do not assume removed scripts or test modules exist.
