# Release Checklist

This fork's release workflow is intentionally scoped to Windows ARM64 desktop artifacts only.
Upstream remains the source for macOS, Linux, Windows x64, and npm CLI releases.

## What the workflow does

- Workflow: `.github/workflows/release.yml`
- Triggers:
  - push tag matching `v*.*.*` for stable releases
  - manual `workflow_dispatch` for stable or nightly releases
- Runs release quality gates first: lint, typecheck, test.
- Builds one desktop artifact:
  - Windows `arm64` NSIS installer
- Publishes one GitHub Release with only Windows ARM64 files:
  - `*.exe`
  - `*.blockmap`
  - `*.yml`
- Does not build or publish:
  - macOS artifacts
  - Linux artifacts
  - Windows x64 artifacts
  - npm CLI packages
- Signing is optional and auto-detected from Azure Trusted Signing secrets.

## Normal CI

Pull requests and pushes to `main` still use `.github/workflows/ci.yml`.
That workflow intentionally keeps the broad validation suite so code breakage is caught before a fork release is cut.

## Nightly builds

- Workflow: `.github/workflows/release.yml`
- Trigger: manual `workflow_dispatch` with `channel=nightly`.
- Runs the same release quality gates as the tagged release flow.
- Builds only Windows ARM64 desktop artifacts.
- Publishes a GitHub prerelease only:
  - tag format: `nightly-vX.Y.Z-nightly.YYYYMMDD.<run_number>`
  - release name includes the short commit SHA
  - `make_latest` is always `false`
- Uses the current `apps/desktop/package.json` semver core (`X.Y.Z`) as the nightly base, then appends a nightly prerelease suffix.
- Publishes Electron auto-update metadata to the dedicated `nightly` updater channel.
- Does not publish the CLI package to npm.
- Does not commit version bumps back to `main`.

## Desktop auto-update notes

- Runtime updater: `electron-updater` in `apps/desktop/src/main.ts`.
- Update UX:
  - Background checks run on startup delay + interval.
  - No automatic download or install.
  - The desktop UI shows a rocket update button when an update is available; click once to download, click again after download to restart/install.
- Provider: GitHub Releases (`provider: github`) configured at build time.
- Repository slug source:
  - `T3CODE_DESKTOP_UPDATE_REPOSITORY` (format `owner/repo`), if set.
  - otherwise `GITHUB_REPOSITORY` from GitHub Actions.
- Temporary private-repo auth workaround:
  - set `T3CODE_DESKTOP_UPDATE_GITHUB_TOKEN` (or `GH_TOKEN`) in the desktop app runtime environment.
  - the app forwards it as an `Authorization: Bearer <token>` request header for updater HTTP calls.
- Required release assets for this fork's updater:
  - Windows ARM64 installer (`.exe`)
  - channel metadata: `latest*.yml` for stable releases, `nightly*.yml` for nightly releases
  - `*.blockmap` files (used for differential downloads)

## 1) Dry-run release without signing

Use this first to validate the fork release pipeline.

1. Confirm no signing secrets are required for this test.
2. Create a test tag:
   - `git tag v0.0.0-test.1`
   - `git push origin v0.0.0-test.1`
3. Wait for `.github/workflows/release.yml` to finish.
4. Verify the GitHub Release contains only Windows ARM64 assets.
5. Download the artifact and sanity-check installation on Windows ARM64.

## 2) Azure Trusted Signing setup (Windows)

Required secrets used by the workflow:

- `AZURE_TENANT_ID`
- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`
- `AZURE_TRUSTED_SIGNING_ENDPOINT`
- `AZURE_TRUSTED_SIGNING_ACCOUNT_NAME`
- `AZURE_TRUSTED_SIGNING_CERTIFICATE_PROFILE_NAME`
- `AZURE_TRUSTED_SIGNING_PUBLISHER_NAME`

Checklist:

1. Create Azure Trusted Signing account and certificate profile.
2. Record ATS values:
   - Endpoint
   - Account name
   - Certificate profile name
   - Publisher name
3. Create or choose an Entra app registration (service principal).
4. Grant service principal permissions required by Trusted Signing.
5. Create a client secret for the service principal.
6. Add Azure secrets listed above in GitHub Actions secrets.
7. Re-run a tag release and confirm the Windows ARM64 installer is signed.

## 3) Ongoing release checklist

1. Ensure `main` is green in CI.
2. Bump app version as needed.
3. Create release tag: `vX.Y.Z`.
4. Push tag.
5. Verify workflow steps:
   - preflight passes
   - Windows ARM64 build passes
   - release job uploads only Windows ARM64 files
6. Smoke test the downloaded artifact on Windows ARM64.

## 4) Troubleshooting

- Windows build unsigned when expected signed:
  - Check all Azure ATS and auth secrets are populated and non-empty.
- Build fails with signing error:
  - Retry with secrets removed to confirm unsigned path still works.
  - Re-check certificate/profile names and tenant/client credentials.
- Non-Windows-ARM64 artifacts appear in a release:
  - Check `.github/workflows/release.yml` for accidental matrix or upload glob expansion.
