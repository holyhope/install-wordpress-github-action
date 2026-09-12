## Why

The action derives the WordPress admin email from `github.actor` (`--admin_email='${{ github.actor }}@github.com'`) with no way to override it. WordPress's `is_email()` (invoked by `wp core install`) rejects the local part of an email whenever it contains characters such as `[` or `]`, which appear in the actor login of every GitHub App / bot user (e.g. `datadog-official[bot]`, `dependabot[bot]`, `renovate[bot]`). Any workflow run by one of these bot identities currently fails outright during "Install WordPress" with `Error: The '<actor>[bot]@github.com' email address is invalid.`, even though the run has nothing to do with email delivery (`--skip-email` is already passed).

## What Changes

- Add an optional `admin_email` input so callers can supply a known-valid email directly, bypassing the `github.actor`-derived value entirely.
- When `admin_email` is not provided, sanitize the `github.actor`-derived local part so it always satisfies WordPress's `is_email()` rules (strip/replace characters WordPress's local-part validation rejects, such as `[` and `]`), instead of passing the raw actor login through unmodified.
- Add an `admin_email` output (mirroring the existing `admin_username`/`admin_password` outputs) so callers can read back whichever email was actually used, whether supplied or derived.
- Document the new input/output in the README's generated Inputs/Outputs tables.

## Capabilities

### New Capabilities

- `admin-account`: requirements governing how the action determines and provisions the WordPress admin account's username, email, and password during `wp core install`.

### Modified Capabilities

(none - this is the action's first capability spec)

## Impact

- `action.yml`: new `admin_email` input, new `admin_email` output, and the "Install WordPress" step's `run` script (sanitization/precedence logic, plus emitting the extra output).
- `README.md`: regenerated Inputs/Outputs tables (via `npx --yes github-action-readme-generator@v1.7.2`, per the file's own generation instructions) to reflect the new input/output.
- No breaking change: existing callers that don't set `admin_email` keep working, and get a fix for the bot-actor failure case for free.
