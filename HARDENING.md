<!-- markdownlint-disable -->

# Hardening Report: fossas--fossa-action/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fossas--fossa-action/v1.8.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in .github/workflows/test.yml use mutable version tags instead of pinned 40-character commit SHAs. Failing references: `actions/checkout@v5` (lines 10, 28, 88), `actions/setup-node@v6` (lines 13, 31), `actions/upload-artifact@v4` (line 46). These can be silently updated to include malicious code.

Locations:

- `.github/workflows/test.yml:10`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:31`
- `.github/workflows/test.yml:46`
- `.github/workflows/test.yml:88`

### script-injection (severity: high)

Sub-rule (a): A `run:` block directly interpolates a GitHub Actions expression inside a shell command string. In the 'Demonstrate report output' step, the expression `${{ steps.example-generate-report.outputs.report }}` is interpolated directly into `echo '${{ steps.example-generate-report.outputs.report }}' | jq`. The report output is attacker-influenced data (it comes from FOSSA analysis of potentially attacker-controlled code) and is substituted into the shell command before the shell ever sees it, enabling command injection.

Locations:

- `.github/workflows/test.yml:85`

### missing-permissions (severity: medium)

The workflow file .github/workflows/test.yml has no top-level `permissions:` key and neither of its jobs (`lint`, `fossa-scan`) defines a job-level `permissions:` block. Without explicit permissions, the workflow runs with the default token permissions, which may be overly broad (e.g., write access to contents and pull-requests depending on repository settings).

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/test.yml: (1) Pinned all six unpinned action references to full 40-char commit SHAs: actions/checkout@v5 → fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09, actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38, actions/upload-artifact@v4 → ea165f8d65b6e75b540449e92b4886f43607fa02, with original tags preserved as comments. (2) Fixed script injection in 'Demonstrate report output' step by moving ${{ steps.example-generate-report.outputs.report }} into the step's env block as REPORT and referencing it as "$REPORT" in the shell script. (3) Added top-level `permissions: {}` to enforce least-privilege token access.

