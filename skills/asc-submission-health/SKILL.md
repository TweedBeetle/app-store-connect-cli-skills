---
name: asc-submission-health
description: Preflight App Store submissions, submit builds, and monitor review status with asc. Use when shipping or troubleshooting review submissions.
---

# asc submission health

Use this skill to validate submission readiness and submit builds.

## Preconditions
- Auth configured and app/version/build IDs resolved.
- Build is processed (not in processing state).

## Validate

Run both validation commands before submitting:

```bash
# Comprehensive readiness check (metadata, localizations, review details, categories,
# build, pricing, screenshots, subscriptions, age ratings)
asc validate --app "APP_ID" --version "X.Y.Z" --strict --output table

# Submission-specific preflight (checks requirements with fix commands)
asc submit preflight --app "APP_ID" --version "X.Y.Z"

# If app has IAP
asc validate iap --app "APP_ID"

# If app has subscriptions (--strict exits non-zero on warnings)
asc validate subscriptions --app "APP_ID" --strict

# If distributing via TestFlight
asc validate testflight --app "APP_ID" --build "BUILD_ID"
```

Fix any issues reported before proceeding.

### Encryption

`asc submit preflight` now checks encryption compliance (asc 0.44.2+). For HTTPS-only apps, use `asc encryption declarations exempt-declare --plist ./Info.plist` to auto-patch, or add `ITSAppUsesNonExemptEncryption = NO` manually to Info.plist and rebuild.

## Submit

### Using Review Submissions API (Recommended)
```bash
# Create submission
asc review submissions-create --app "APP_ID" --platform IOS

# Add version to submission
asc review items-add \
  --submission "SUBMISSION_ID" \
  --item-type appStoreVersions \
  --item-id "VERSION_ID"

# Submit for review
asc review submissions-submit --id "SUBMISSION_ID" --confirm
```

### Using Submit Command
```bash
asc submit create --app "APP_ID" --version "1.2.3" --build "BUILD_ID" --confirm
```
Use `--platform` when multiple platforms exist.

## Monitor
```bash
# Check submission status
asc submit status --id "SUBMISSION_ID"
asc submit status --version-id "VERSION_ID"

# List all submissions
asc review submissions-list --app "APP_ID" --paginate
```

## Cancel / Retry
```bash
# Cancel submission
asc submit cancel --id "SUBMISSION_ID" --confirm

# Or via review API
asc review submissions-cancel --id "SUBMISSION_ID" --confirm
```
Fix issues, then re-submit.

## Notes
- `asc submit create` uses the new reviewSubmissions API automatically.
- Use `--output table` when you want human-readable status.
- macOS submissions follow the same process but use `--platform MAC_OS`.
