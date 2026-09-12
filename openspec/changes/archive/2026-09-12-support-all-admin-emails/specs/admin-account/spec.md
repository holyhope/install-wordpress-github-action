## Purpose

Defines how the action determines and exposes the WordPress admin account's email address during installation, so that every GitHub actor identity - including GitHub App/bot logins with characters WordPress rejects - can install WordPress successfully.

## ADDED Requirements

### Requirement: Admin email override input
The action SHALL accept an optional `admin_email` input. When set to a non-empty value, the action SHALL pass that value verbatim as the WordPress admin email to `wp core install`, without deriving or sanitizing an email from the GitHub actor login.

#### Scenario: Caller supplies admin_email
- **WHEN** the action is invoked with `admin_email: "team@example.com"`
- **THEN** WordPress is installed with `team@example.com` as the admin email, and no email is derived from `github.actor`

#### Scenario: Caller supplies admin_email for a bot actor
- **WHEN** the action runs as actor `datadog-official[bot]` and is invoked with `admin_email: "team@example.com"`
- **THEN** WordPress is installed with `team@example.com` as the admin email, and installation does not fail because of the actor login's characters

### Requirement: Default admin email is always a valid email address
When the `admin_email` input is not set (or empty), the action SHALL derive the admin email from `github.actor`, and SHALL sanitize that derived value so its local part contains only characters WordPress's email validation (`is_email()`) accepts as an unquoted local part, regardless of which characters appear in the actor login.

#### Scenario: Actor login contains characters WordPress rejects
- **WHEN** `admin_email` is not set and `github.actor` is `datadog-official[bot]`
- **THEN** the action derives an admin email whose local part has the rejected characters (such as `[` and `]`) removed or replaced, and `wp core install` succeeds instead of failing with `The '...' email address is invalid.`

#### Scenario: Actor login contains no characters WordPress rejects
- **WHEN** `admin_email` is not set and `github.actor` is `octocat`
- **THEN** the action derives the admin email `octocat@github.com`, unchanged from current behavior

### Requirement: Admin email output
The action SHALL expose an `admin_email` output containing the exact email address that was passed to `wp core install`, whether it came from the `admin_email` input or was derived from `github.actor`.

#### Scenario: Output reflects a supplied admin_email
- **WHEN** the action is invoked with `admin_email: "team@example.com"`
- **THEN** the action's `admin_email` output is `team@example.com`

#### Scenario: Output reflects a derived and sanitized admin email
- **WHEN** `admin_email` is not set and `github.actor` is `datadog-official[bot]`
- **THEN** the action's `admin_email` output is the sanitized email that was actually used to install WordPress, not the raw unsanitized `datadog-official[bot]@github.com`
