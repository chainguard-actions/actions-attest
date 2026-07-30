<!-- markdownlint-disable -->

# Hardening Report: actions--attest/v4.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--attest/v4.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings in multiple workflow steps.

1. ci.yml 'Calculate subject digest' step: SHA_256=$(gh api "${{ env.SUBJECT }}" | shasum ...) — env context value substituted directly into shell.

2. ci.yml 'Dump output' step: run: jq < ${{ steps.attest.outputs.bundle-path }} — steps output interpolated directly into shell command.

3. commit-dist.yml 'Check for artifact' step: incoming="${{ runner.temp }}/incoming" — runner context interpolated directly into shell.

4. commit-dist.yml 'Verify head is unchanged' step: if [ "$current" != "${{ steps.check.outputs.head-sha }}" ] and echo with ${{ steps.check.outputs.head-sha }} — steps output in shell.

5. commit-dist.yml 'Apply rebuilt dist/' step: mv "${{ runner.temp }}/incoming/dist" dist — runner context in shell.

6. commit-dist.yml 'Commit and push' step: git push origin HEAD:${{ steps.check.outputs.head-ref }} — steps output in shell.

7. rebuild-dist.yml 'Record PR context' step: echo "${{ github.event.pull_request.number }}", echo "${{ github.event.pull_request.head.ref }}", echo "${{ github.event.pull_request.head.sha }}" — attacker-controllable github context (head.ref) interpolated directly into shell commands.

Locations:

- `.github/workflows/ci.yml:67`
- `.github/workflows/ci.yml:81`
- `.github/workflows/commit-dist.yml:52`
- `.github/workflows/commit-dist.yml:82`
- `.github/workflows/commit-dist.yml:83`
- `.github/workflows/commit-dist.yml:91`
- `.github/workflows/commit-dist.yml:104`
- `.github/workflows/rebuild-dist.yml:61`
- `.github/workflows/rebuild-dist.yml:62`
- `.github/workflows/rebuild-dist.yml:63`

### unpinned-uses (severity: high)

prober.yml references uses: actions/attest@main, which is pinned to a mutable branch name rather than a full 40-character commit SHA. This means the action code can change at any time without notice, creating a supply-chain attack vector. All uses: references must be pinned to an immutable SHA digest (e.g. actions/attest@<40-hex-char-sha> # main).

Locations:

- `.github/workflows/prober.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed script-injection in ci.yml (2 steps), commit-dist.yml (4 steps), and rebuild-dist.yml (1 step) by moving all ${{ }} expressions out of run: shell strings into step env: blocks and referencing them as plain environment variables. Pinned actions/attest@main to full SHA 508db95dd578ae2727ebd6217d5ba78e4fbda05d in prober.yml.

