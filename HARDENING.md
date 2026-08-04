<!-- markdownlint-disable -->

# Hardening Report: actions--attest/v4.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--attest/v4.2.2** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ ... }}` expressions are interpolated directly inside `run:` shell blocks. In ci.yml, `${{ env.SUBJECT }}` (which itself contains `${{ github.repository }}` and `${{ github.sha }}`) is interpolated into a shell command: `SHA_256=$(gh api "${{ env.SUBJECT }}" | shasum ...)`. Additionally, `${{ steps.attest.outputs.bundle-path }}` is interpolated unquoted in `run: jq < ${{ steps.attest.outputs.bundle-path }}`, allowing shell metacharacter injection from a step output.

Locations:

- `.github/workflows/ci.yml:55`
- `.github/workflows/ci.yml:69`

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell blocks in commit-dist.yml. Specifically: `incoming="${{ runner.temp }}/incoming"` (runner context in shell), `"${{ steps.check.outputs.head-sha }}"` used in a string comparison and echo (step output in shell), `mv "${{ runner.temp }}/incoming/dist"` (runner context in shell), and `git push origin HEAD:${{ steps.check.outputs.head-ref }}` (step output — derived from PR artifact — injected unquoted into a git push command). Any of these allow shell metacharacter injection.

Locations:

- `.github/workflows/commit-dist.yml:48`
- `.github/workflows/commit-dist.yml:77`
- `.github/workflows/commit-dist.yml:78`
- `.github/workflows/commit-dist.yml:84`
- `.github/workflows/commit-dist.yml:85`
- `.github/workflows/commit-dist.yml:96`

### script-injection (severity: high)

Sub-rule (a): Attacker-controlled pull request data is interpolated directly inside `run:` shell commands in rebuild-dist.yml. The 'Record PR context' step writes `${{ github.event.pull_request.number }}`, `${{ github.event.pull_request.head.ref }}`, and `${{ github.event.pull_request.head.sha }}` directly into shell echo commands. A malicious PR author can craft these values to inject shell metacharacters.

Locations:

- `.github/workflows/rebuild-dist.yml:57`
- `.github/workflows/rebuild-dist.yml:58`
- `.github/workflows/rebuild-dist.yml:59`

### unpinned-uses (severity: high)

The step 'Attest build provenance' in prober.yml uses `actions/attest@main`, which is a mutable branch reference rather than a pinned 40-character commit SHA. This means the action code can change at any time without notice, creating a supply-chain risk.

Locations:

- `.github/workflows/prober.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed script injection in ci.yml (lines 55, 69), commit-dist.yml (lines 48, 77-78, 84-85, 96), and rebuild-dist.yml (lines 57-59) by moving all ${{ ... }} expressions out of run: shell blocks into step env: blocks and referencing them as plain shell variables. Fixed unpinned-uses in prober.yml by pinning actions/attest@main to commit SHA 1e69f48acb82d1966a394da916b4c1698aa569d6.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In .github/workflows/commit-dist.yml, added immediate sanitization using `printf '%s' ... | tr -d '\n\r'` for both `head_ref` and `head_sha` immediately before writing them to $GITHUB_OUTPUT (lines 72-73). The sanitized values are stored in `safe_head_ref` and `safe_head_sha` variables, which are then used in the echo statements. This ensures the sanitization is immediately adjacent to the $GITHUB_OUTPUT writes, satisfying the security requirement even though earlier sanitization via `tr -d '\r\n'` was already applied at read time.

