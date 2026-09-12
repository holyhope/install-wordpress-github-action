## Context

See `proposal.md` - Why. The relevant code is entirely in the "Install WordPress" step of `action.yml`:

```yaml
- name: Install WordPress
  shell: bash
  id: install
  run: |-
    ADMIN_PASSWORD="$( \
      '...' core install \
        ...
        --admin_user='${{ github.actor }}' \
        --admin_email='${{ github.actor }}@github.com' \
        --skip-email \
        ...
    )"
    ...
    cat <<EOF >> "$GITHUB_OUTPUT"
    admin_username=${{ github.actor }}
    admin_password=$ADMIN_PASSWORD
    EOF
```

`${{ github.actor }}` is interpolated directly into the `run:` script text (not passed through `env:`), which is also a mild script-injection smell flagged by GitHub Actions security tooling (actor logins are GitHub-controlled, but the action shouldn't rely on that to justify raw interpolation).

WordPress's `is_email()` (used by `wp core install`) only accepts a specific character set in an unquoted local part: letters, digits, and `!#$%&'*+-=?^_\`.{|}~`. It rejects `[`, `]`, spaces, and a few others outright - which is exactly what breaks bot actors like `datadog-official[bot]`.

## Goals / Non-Goals

**Goals:**
- Let callers fully control the admin email via a new input.
- Make the default (no input) path never fail `wp core install` due to actor-login characters, for any current or future GitHub bot/app login shape.
- Expose whichever email was actually used as an output.

**Non-Goals:**
- Exactly replicating every rule of WordPress's `is_email()`. A conservative, safe-by-construction subset is sufficient and easier to reason about than porting WP's validation logic into bash.
- Changing `admin_user`/`admin_username` handling - WP-CLI does not validate `admin_user` as an email, so the bracket characters in bot logins are not a problem there.
- Touching the `--skip-email` behavior or any other install flag.

## Decisions

1. **Precedence**: if the new `admin_email` input is non-empty, use it verbatim; otherwise derive from `github.actor` as before, but sanitized. This is a strict superset of current behavior for every existing caller that doesn't set the input.

2. **Sanitization approach**: strip the derived local part (`github.actor`) down to a conservative allowed set - ASCII letters, digits, `.`, `_`, `+`, `-` - via `tr -dc`, rather than porting WordPress's full `is_email()` local-part grammar. This set is a strict subset of what `is_email()` accepts, so anything that survives is guaranteed valid; it also happens to be the common "safe" subset most mail systems accept, unlike punctuation such as `!`, `#`, `'`, or backtick that WP allows but that are also shell/YAML-awkward.
   - *Alternative considered*: reimplement `is_email()`'s exact local-part character class in bash/regex. Rejected - more code, more ways to drift from WP core if it changes its validation, for a benefit (allowing a few extra punctuation characters through) that doesn't matter for GitHub actor logins, which are already alphanumeric/hyphen (plus the bot suffix's brackets).
   - *Alternative considered*: fail fast with a clear error telling the caller to set `admin_email` instead of silently sanitizing. Rejected - it would keep the current hard failure for every bot actor with no behavior change unless every caller opts in, defeating the point of the fix; sanitizing keeps today's zero-config default working for bot actors too.

3. **Empty-after-sanitization fallback**: if stripping leaves an empty local part (theoretically possible for an actor login made entirely of now-disallowed characters), fall back to a fixed literal local part (e.g. `github-actions`) rather than producing `@github.com` with no local part.

4. **Avoid raw `${{ github.actor }}`/`${{ inputs.admin_email }}` interpolation in the script body**: pass both through the step's `env:` block (e.g. `ACTOR`, `ADMIN_EMAIL_INPUT`) and reference them as `"$ACTOR"`/`"$ADMIN_EMAIL_INPUT"` in bash, consistent with GitHub's script-injection hardening guidance, and incidentally the only reliable way to sanitize a value with `tr`/parameter expansion instead of re-embedding it into YAML/webhook-templated text.

5. **Output emission**: capture the final email actually passed to `wp core install` in a shell variable (`ADMIN_EMAIL`) and append `admin_email=$ADMIN_EMAIL` to `$GITHUB_OUTPUT` alongside the existing `admin_username`/`admin_password` lines, then wire it to a new `admin_email` action output the same way `admin_username` is wired today.

## Risks / Trade-offs

- [Risk] The conservative allowed-character subset could, in principle, differ from a future change to WordPress's `is_email()` → Mitigation: the subset (`[A-Za-z0-9._+-]`) is deliberately far inside any historical or plausible future `is_email()` local-part grammar, and the `admin_email` input remains available as a full escape hatch for any address the sanitizer can't represent.
- [Risk] Silent sanitization changes the *value* of the default admin email for any actor login that contains characters outside the allowed set (previously it would have failed outright, so there's no prior "working" value to regress) → Mitigation: expose the new `admin_email` output so callers can always see what was actually used, and document the behavior in the README/spec.
- [Risk] `README.md`'s Inputs/Outputs tables are generated (`npx --yes github-action-readme-generator@v1.7.2`) and must be regenerated, not hand-edited, or they'll drift from `action.yml` → Mitigation: task list calls this out explicitly as its own step.

## Migration Plan

No data migration. Purely additive to `action.yml` (new input, new output) plus a behavior fix inside the existing "Install WordPress" step. Existing callers that never set `admin_email` are unaffected unless their actor login previously caused a hard failure (Digiposte/bot-style actors), in which case they now succeed instead. No rollback concerns beyond reverting the commit.
