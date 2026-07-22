<!-- markdownlint-disable -->

# Hardening Report: actions--attest/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--attest/v4.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow steps use action references pinned to mutable tags rather than immutable full 40-character SHA commit digests, making them vulnerable to supply-chain attacks if the tag is moved.

In .github/workflows/check-dist.yml:
- `uses: actions/checkout@v6.0.2` (line 30)
- `uses: actions/setup-node@v6.2.0` (line 35)
- `uses: actions/upload-artifact@v6` (line 57)

In .github/workflows/codeql-analysis.yml:
- `uses: actions/checkout@v6.0.2` (line 33)
- `uses: github/codeql-action/init@v4` (line 38)
- `uses: github/codeql-action/autobuild@v4` (line 44)
- `uses: github/codeql-action/analyze@v4` (line 49)

All of these should be replaced with their full SHA digest, e.g. `actions/checkout@<40-char-sha> # v6.0.2`.

Locations:

- `.github/workflows/check-dist.yml:30`
- `.github/workflows/check-dist.yml:35`
- `.github/workflows/check-dist.yml:57`
- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:44`
- `.github/workflows/codeql-analysis.yml:49`

### script-injection (severity: high)

Two `run:` steps in ci.yml directly interpolate GitHub Actions expressions inside shell command strings (sub-rule a), allowing an attacker to inject arbitrary shell commands.

1. Line 57 — the `Calculate subject digest` step interpolates `${{ env.SUBJECT }}` directly inside a shell command:
   ```
   SHA_256=$(gh api "${{ env.SUBJECT }}" | shasum -a 256 | cut -d " " -f 1)
   ```
   `env.SUBJECT` is set from `${{ github.repository }}` and `${{ github.sha }}`, both of which flow through YAML template substitution before the shell sees them. Fix: pass the value via an `env:` variable and reference it as `"$SUBJECT"` in the shell.

2. Line 72 — the `Dump output` step interpolates `${{ steps.attest.outputs.bundle-path }}` directly inside a shell command:
   ```
   run: jq < ${{ steps.attest.outputs.bundle-path }}
   ```
   A malicious or compromised prior step could set `bundle-path` to a value containing shell metacharacters. Fix: move the value to an `env:` variable and reference it as `"$BUNDLE_PATH"` in the shell.

Locations:

- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:72`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed 7 unpinned action references across check-dist.yml and codeql-analysis.yml by replacing mutable tags with full 40-character SHA digests (preserving tags as comments). Fixed 2 script injection vulnerabilities in ci.yml: (1) moved env.SUBJECT into the step's env: block for the 'Calculate subject digest' step, referencing it as $SUBJECT in the shell; (2) moved steps.attest.outputs.bundle-path into an env: block as BUNDLE_PATH for the 'Dump output' step, referencing it as "$BUNDLE_PATH" in the shell.

