<!-- markdownlint-disable -->

# Hardening Report: DeLaGuardo--setup-clojure/13.5.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **DeLaGuardo--setup-clojure/13.5.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings in update-major-version-tag.yml. The 'Get tag name' step uses `${{ github.event_name }}` and `${{ inputs.tag }}` directly in the shell script. The 'Update major version tag' step uses `${{ steps.get-tag.outputs.tag }}` and `${{ steps.get-tag.outputs.major_tag }}` directly in the shell script. These values flow through YAML template substitution before the shell sees them, enabling command injection.

Locations:

- `.github/workflows/update-major-version-tag.yml:28`
- `.github/workflows/update-major-version-tag.yml:29`
- `.github/workflows/update-major-version-tag.yml:38`
- `.github/workflows/update-major-version-tag.yml:39`

### github-env-injection (severity: high)

In update-major-version-tag.yml, the 'Get tag name' step interpolates `${{ inputs.tag }}` directly into the shell variable TAG, then writes `echo "tag=$TAG" >> $GITHUB_OUTPUT` and `echo "major_tag=$MAJOR_TAG" >> $GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled tag input containing newlines could inject arbitrary key-value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/update-major-version-tag.yml:33`
- `.github/workflows/update-major-version-tag.yml:36`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or branch names instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag or branch is updated maliciously. Failing references include: actions/checkout@main, actions/setup-java@v3 (no_auth.yml); actions/checkout@main, actions/setup-java@v3 (smoke-tests.yml, repeated across all jobs); actions/checkout@v4 (update-major-version-tag.yml); actions/checkout@main, actions/setup-node@master, actions/setup-java@v3, actions/cache@v3 (workflow.yml).

Locations:

- `.github/workflows/no_auth.yml:13`
- `.github/workflows/no_auth.yml:17`
- `.github/workflows/smoke-tests.yml:17`
- `.github/workflows/smoke-tests.yml:20`
- `.github/workflows/update-major-version-tag.yml:22`
- `.github/workflows/workflow.yml:14`
- `.github/workflows/workflow.yml:18`
- `.github/workflows/workflow.yml:23`
- `.github/workflows/workflow.yml:30`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Affected files: no_auth.yml, smoke-tests.yml, workflow.yml.

Locations:

- `.github/workflows/no_auth.yml:1`
- `.github/workflows/smoke-tests.yml:1`
- `.github/workflows/workflow.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings across four workflow files:

1. script-injection (update-major-version-tag.yml): Moved ${{ github.event_name }}, ${{ inputs.tag }}, ${{ steps.get-tag.outputs.tag }}, and ${{ steps.get-tag.outputs.major_tag }} out of run: shell strings into env: blocks, referencing them as plain env vars in the shell.

2. github-env-injection (update-major-version-tag.yml): Added printf '%s' ... | tr -d '\n\r' sanitization before writing tag and major_tag to $GITHUB_OUTPUT.

3. unpinned-uses: Pinned all mutable action references to full 40-char commit SHAs with tag comments: actions/checkout@main→3d3c42e5, actions/checkout@v4→34e11487, actions/setup-java@v3→17f84c36, actions/setup-node@master→60c11408, actions/cache@v3→6f8efc29.

4. missing-permissions: Added permissions: {} at the top level of no_auth.yml, smoke-tests.yml, and workflow.yml.

