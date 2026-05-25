---
name: asc-cli-usage
description: Guidance for using asc cli in this repo (flags, output formats, pagination, auth, and discovery). Use when asked to run or design asc commands or interact with App Store Connect via the CLI.
---

# asc cli usage

Use this skill when you need to run or design `asc` commands for App Store Connect.

## Command discovery
- Always use `--help` to discover commands and flags.
  - `asc --help`
  - `asc builds --help`
  - `asc builds list --help`

## Flag conventions
- Use explicit long flags (e.g., `--app`, `--output`).
- No interactive prompts; destructive operations require `--confirm`.
- Use `--paginate` when the user wants all pages.

## Output formats
- Default output is minified JSON.
- Use `--output table` or `--output markdown` only for human-readable output.
- `--pretty` is only valid with JSON output.

## Authentication and defaults
- Prefer keychain auth via `asc auth login`.
- Fallback env vars: `ASC_KEY_ID`, `ASC_ISSUER_ID`, `ASC_PRIVATE_KEY_PATH`, `ASC_PRIVATE_KEY`, `ASC_PRIVATE_KEY_B64`.
- `ASC_APP_ID` can provide a default app ID.

### Two separate auths — they expire independently <!-- added: 2026-05-24 -->
- **API-key auth** (`asc auth login` / the `ASC_*` key env vars) backs the official endpoints: `asc apps`, `asc iap`, `asc builds`, `asc pricing`, etc. This is what `asc apps list` uses.
- **Web-session auth** (`asc web auth login --apple-id <email>` + 2FA) backs the *experimental* `asc web *` family (app creation, sandbox, privacy declarations). A DIFFERENT credential with its own expiry; `asc web auth status` shows whether it's live.
- Symptom that wastes time: `asc web apps create` failing with **"Session expired"** (or a misleading 409) while `asc apps list` works fine — the API key is valid but the *web* session lapsed. Refresh only the web session; don't re-auth the API key chasing it.
- `ASC_BYPASS_KEYCHAIN=1` makes `asc` use a file-based credential cache instead of the macOS Keychain — useful when a Keychain prompt would silently hang a non-interactive run. **Login and the subsequent command must use the SAME bypass state**, or the second command looks in the wrong store and reports "no cached session." (Undocumented env var — escape hatch, not a default.)

### In-app purchases (`asc iap`) <!-- added: 2026-05-24 -->
- `asc iap setup` does create + first-localization + initial price schedule in one call. If a later step fails, **the IAP record is still created** — resume the failed steps on the existing iapId (`asc iap localizations create`, `asc iap pricing schedules create --price <amount> --base-territory USA`) rather than delete + recreate.
- **IAP localization `--description` max = 55 characters** (display `--name` max 30). Easy to overshoot → `create_localization` fails.
- A new IAP sits in `MISSING_METADATA` until it has a localization, a price, AND a **review screenshot**; the **first IAP for an app must be attached to a version in the ASC web UI** (no API/CLI path for that attachment).

## Timeouts
- `ASC_TIMEOUT` / `ASC_TIMEOUT_SECONDS` control request timeouts.
- `ASC_UPLOAD_TIMEOUT` / `ASC_UPLOAD_TIMEOUT_SECONDS` control upload timeouts.
