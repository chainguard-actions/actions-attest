<!-- markdownlint-disable -->

# Hardening Report: actions--attest/v4.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--attest/v4.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow references `actions/attest@main` — a mutable branch name rather than a full 40-character commit SHA. This means the action code can change at any time without notice, enabling a supply-chain attack if the branch is compromised. It should be pinned to a specific commit SHA (e.g., `actions/attest@<40-char-sha> # main`).

Locations:

- `.github/workflows/prober.yml:30`

### script-injection (severity: high)

Rule (a): Direct expression interpolation inside a `run:` shell command. In the 'Calculate subject digest' step, `${{ env.SUBJECT }}` is interpolated directly into the shell command: `SHA_256=$(gh api "${{ env.SUBJECT }}" | shasum -a 256 | cut -d " " -f 1)`. The `env.SUBJECT` value is derived from `github.repository` and `github.sha`, which are workflow-controlled contexts. Before the shell ever sees the string, YAML template substitution replaces the expression, allowing an attacker-controlled repository name to inject shell metacharacters. The value should be passed via an `env:` variable and double-quoted: `gh api "$SUBJECT"`.

Locations:

- `.github/workflows/ci.yml:53`

### script-injection (severity: high)

Rule (a): Direct expression interpolation inside a `run:` shell command. In the 'Dump output' step, `${{ steps.attest.outputs.bundle-path }}` is interpolated directly into the shell command: `run: jq < ${{ steps.attest.outputs.bundle-path }}`. Step outputs are workflow-controllable contexts; injecting a value containing shell metacharacters (e.g., a path with spaces or special characters) could alter command execution. The value should be passed via an `env:` variable and double-quoted: `env: BUNDLE_PATH: ${{ steps.attest.outputs.bundle-path }}` then `run: jq < "$BUNDLE_PATH"`.

Locations:

- `.github/workflows/ci.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed 3 findings across 2 files: (1) In prober.yml line 30, pinned `actions/attest@main` to full SHA `f7c74d28b9d84cb8768d0b8ca14a4bac6ef463e6` with `# main` comment. (2) In ci.yml line 53, moved `${{ env.SUBJECT }}` from the `run:` shell string into the step's `env:` block and referenced it as `"$SUBJECT"` in the shell command. (3) In ci.yml line 63, moved `${{ steps.attest.outputs.bundle-path }}` from the `run:` shell string into the step's `env:` block as `BUNDLE_PATH` and referenced it as `"$BUNDLE_PATH"` in the shell command.

