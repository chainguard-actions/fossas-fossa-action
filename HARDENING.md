<!-- markdownlint-disable -->

# Hardening Report: fossas--fossa-action/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fossas--fossa-action/v1.9.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in both workflow files are pinned to mutable version tags rather than immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the upstream action tag is moved or compromised.

Failing references in .github/workflows/rebuild-dist.yml:
- `uses: actions/checkout@v6` (line 18)
- `uses: actions/setup-node@v6` (line 36)

Failing references in .github/workflows/test.yml:
- `uses: actions/checkout@v6` (lines 9, 28, 72)
- `uses: actions/setup-node@v6` (lines 12, 31)
- `uses: actions/upload-artifact@v7` (line 44)

All should be replaced with full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/rebuild-dist.yml:18`
- `.github/workflows/rebuild-dist.yml:36`
- `.github/workflows/test.yml:9`
- `.github/workflows/test.yml:12`
- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:31`
- `.github/workflows/test.yml:44`
- `.github/workflows/test.yml:72`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string. In the 'Demonstrate report output' step, the step output `steps.example-generate-report.outputs.report` is injected into the shell command before the shell ever sees it, allowing an attacker who can influence the report content to inject arbitrary shell commands.

Offending line:
```
echo '${{ steps.example-generate-report.outputs.report }}' | jq
```

Fix: pass the value via an `env:` variable and reference it as a quoted shell variable:
```yaml
env:
  REPORT: ${{ steps.example-generate-report.outputs.report }}
run: echo "$REPORT" | jq
```

Locations:

- `.github/workflows/test.yml:80`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/test.yml` has no top-level `permissions:` key and neither of its jobs (`lint`, `fossa-scan`) defines a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. `write` on `contents`). A minimal explicit `permissions:` block (e.g. `permissions: read-all` or specific scopes) should be added at the top level or on each job.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings:
1. unpinned-uses: Pinned all `uses:` references to full commit SHAs in both .github/workflows/rebuild-dist.yml and .github/workflows/test.yml. actions/checkout@v6 → SHA d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-node@v6 → SHA 249970729cb0ef3589644e2896645e5dc5ba9c38, actions/upload-artifact@v7 → SHA 043fb46d1a93c77aae656e7c1c64a875d1fc6a0a.
2. script-injection: In test.yml 'Demonstrate report output' step, moved `${{ steps.example-generate-report.outputs.report }}` from the run: shell string into an env: block as REPORT, and referenced it as "$REPORT" in the shell command.
3. missing-permissions: Added top-level `permissions: contents: read` block to test.yml to restrict the GITHUB_TOKEN to the minimum needed scope.

