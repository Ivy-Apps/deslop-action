# Deslop Action

A GitHub Action that runs [@ivy-apps/deslop](https://www.npmjs.com/package/@ivy-apps/deslop) from NPM. Use it in your workflows to **check** for issues (and fail the job if needed) or **fix** issues and open a pull request.

## Usage

Reference the action from this repository. Pin to a major or exact version for predictable runs:

```yaml
# Pin to a specific version (recommended for CI)
uses: ivy-apps/deslop-action@v1

# Or pin to an exact tag
uses: ivy-apps/deslop-action@v1.0.0
```

### Check mode (default)

Deslop runs in read-only mode, reports problems to stderr, and exits with code 1 if there are issues. Use this in CI to block merges when the repo doesn’t pass Deslop checks.

```yaml
jobs:
  deslop:
    runs-on: ubuntu-latest
    steps:
      - uses: ivy-apps/deslop-action@v1
        with:
          version: "1"          # NPM version of @ivy-apps/deslop
          mode: check
```

### Fix mode

Deslop applies fixes to the working tree. The action then commits the changes to a new branch and opens a pull request targeting the branch that ran the workflow.

**Required:** In **Settings → Actions → General**, set “Workflow permissions” to **Read and write**.  
For the job that runs Deslop in fix mode, grant:

- `contents: write`
- `pull-requests: write`

Example:

```yaml
jobs:
  deslop-fix:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: ivy-apps/deslop-action@v1
        with:
          version: "1"
          mode: fix
```

Optional: use the action outputs (e.g. PR URL) in later steps:

```yaml
      - id: deslop
        uses: ivy-apps/deslop-action@v1
        with:
          version: "1"
          mode: fix

      - name: Comment PR link
        if: steps.deslop.outputs.pull-request-url != ''
        run: echo "Opened ${{ steps.deslop.outputs.pull-request-url }}"
```

## Inputs

| Input    | Required | Default   | Description |
|----------|----------|-----------|-------------|
| `gemini-api-key` | No | `""` | Gemini API key for Deslop (e.g. next-intl translations). Omit or use a placeholder; Deslop works without a valid key but translations won’t run. Pass via a secret: `gemini-api-key: ${{ secrets.GEMINI_API_KEY }}`. |
| `version` | No     | `latest` | NPM version of `@ivy-apps/deslop` (e.g. `"1.2.3"`, `"1"`, or `"latest"`). Prefer a specific version for reproducible runs. |
| `mode`   | No       | `check`   | `check` — report only, may fail the job; `fix` — apply fixes and open a PR. |
| `args`   | No       | `""`      | Extra arguments passed to the Deslop CLI. Use this if your Deslop version uses different flags or subcommands (e.g. `check` / `fix`).

## Outputs (fix mode)

| Output                 | Description |
|------------------------|-------------|
| `pull-request-number`  | Number of the created or updated PR. |
| `pull-request-url`     | URL of the created or updated PR. |

## Versioning

- Pin the **action** version (e.g. `@v1` or `@v1.0.0`) so workflow updates are intentional.
- Pin the **Deslop** version via the `version` input (e.g. `version: "1.2.3"`) so Deslop behavior is consistent across runs.

## License

BSD 3-Clause. See [LICENSE](LICENSE).
