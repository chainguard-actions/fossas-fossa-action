<!-- markdownlint-disable -->

# Hardening Report: fossas--fossa-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fossas--fossa-action/v2.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag refs instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised.

In `.github/workflows/rebuild-dist.yml`:
- `uses: actions/checkout@v6` (line 19)
- `uses: actions/setup-node@v6` (line 37)

In `.github/workflows/test.yml`:
- `uses: actions/checkout@v6` (lines 10, 27, 68)
- `uses: actions/setup-node@v6` (lines 13, 30)
- `uses: actions/upload-artifact@v7` (line 47)

Locations:

- `.github/workflows/rebuild-dist.yml:19`
- `.github/workflows/rebuild-dist.yml:37`
- `.github/workflows/test.yml:10`
- `.github/workflows/test.yml:13`
- `.github/workflows/test.yml:27`
- `.github/workflows/test.yml:30`
- `.github/workflows/test.yml:47`
- `.github/workflows/test.yml:68`

### missing-permissions (severity: medium)

`.github/workflows/test.yml` has no top-level `permissions:` key and neither of its jobs (`lint`, `fossa-scan`) defines a `permissions:` block. Without explicit permissions, the workflow inherits the default repository token permissions, which may be overly broad (e.g. write access to contents). A minimal permissions block such as `permissions: read-all` or specific scopes should be added.

Locations:

- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string in the "Demonstrate report output" step of `.github/workflows/test.yml`. The expression `${{ steps.example-generate-report.outputs.report }}` is substituted into the shell command before the shell parses it, allowing an attacker who can influence the report output to inject arbitrary shell commands.

Offending line:
```
echo '${{ steps.example-generate-report.outputs.report }}' | jq
```

Fix: pass the value via an `env:` variable and reference it as a quoted shell variable, e.g.:
```yaml
env:
  REPORT: ${{ steps.example-generate-report.outputs.report }}
run: echo "$REPORT" | jq
```

Locations:

- `.github/workflows/test.yml:98`

### github-env-injection (severity: high)

In the "Determine diff revision" step of `.github/workflows/test.yml`, the values of `BEFORE_SHA` (sourced from `github.event.before`) and `DEFAULT_BRANCH` (sourced from `github.event.repository.default_branch`) are written to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who can control these values (e.g. via a crafted push event) could inject newlines to poison the `$GITHUB_OUTPUT` file and set arbitrary output variables.

Offending lines:
```
echo "rev=$(git rev-parse "origin/$DEFAULT_BRANCH")" >> "$GITHUB_OUTPUT"
echo "rev=$BEFORE_SHA" >> "$GITHUB_OUTPUT"
```

Fix: sanitize before writing, e.g.:
```bash
safe_rev=$(printf '%s' "$BEFORE_SHA" | tr -d '\n\r')
echo "rev=$safe_rev" >> "$GITHUB_OUTPUT"
```

Locations:

- `.github/workflows/test.yml:80`
- `.github/workflows/test.yml:82`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across .github/workflows/test.yml and .github/workflows/rebuild-dist.yml:
1. Pinned all actions/checkout@v6, actions/setup-node@v6, and actions/upload-artifact@v7 references to their full 40-char SHA commits with tag comments.
2. Added top-level `permissions: contents: read` to test.yml.
3. Moved `${{ steps.example-generate-report.outputs.report }}` out of the run: shell string into an env: block (REPORT variable) to prevent script injection.
4. Sanitized BEFORE_SHA and git rev-parse output with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT to prevent newline injection.

