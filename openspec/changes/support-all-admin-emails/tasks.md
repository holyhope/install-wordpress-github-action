## 1. Action definition

- [x] 1.1 Add an optional `admin_email` input to `action.yml` (no default, empty string means "not set") and verify `action.yml` still parses as valid YAML
- [x] 1.2 Add an `admin_email` output to `action.yml`, wired to a new `steps.install.outputs.admin_email`, mirroring how `admin_username` is wired

## 2. Install step behavior

- [x] 2.1 Pass `github.actor` and `inputs.admin_email` into the "Install WordPress" step via `env:` (e.g. `ACTOR`, `ADMIN_EMAIL_INPUT`) instead of interpolating `${{ ... }}` directly into the script body, and verify the step still runs (manual `act`/workflow run or code review of the diff)
- [x] 2.2 In the step's bash script, compute the effective admin email: use `$ADMIN_EMAIL_INPUT` verbatim if non-empty, else sanitize `$ACTOR` by stripping every character outside `[A-Za-z0-9._+-]` and appending `@github.com`, falling back to a fixed local part (e.g. `github-actions`) if sanitization leaves it empty
- [x] 2.3 Pass the computed email to `wp core install --admin_email=...` in place of the current raw `${{ github.actor }}@github.com` expression
- [x] 2.4 Emit `admin_email=<computed value>` to `$GITHUB_OUTPUT` alongside the existing `admin_username`/`admin_password` lines

## 3. Verification

- [x] 3.1 Manually trace/test the sanitization against `datadog-official[bot]` (must produce a valid, non-empty local part with no `[`/`]`) and `octocat` (must be unchanged: `octocat@github.com`), confirming both match the spec's scenarios
- [x] 3.2 Run this action end-to-end (e.g. via a workflow dispatch or the repo's own CI) with no `admin_email` input set while acting as a bot-style actor if possible, or otherwise confirm via a local `wp core install --admin_email=<sanitized-bot-login>@github.com` dry run that WordPress accepts the sanitized address
- [x] 3.3 Run this action with `admin_email` explicitly set and confirm the value passes through unchanged to both the WordPress install and the `admin_email` output

## 4. Documentation

- [x] 4.1 Regenerate `README.md`'s Inputs/Outputs tables via `npx --yes github-action-readme-generator@v1.7.2` (per the file's own generation instructions) and verify the new `admin_email` input and output now appear
- [x] 4.2 Review the regenerated README diff for unrelated churn before committing
