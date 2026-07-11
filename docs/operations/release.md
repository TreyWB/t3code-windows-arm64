# Windows ARM64 Release Checklist

This fork publishes Windows ARM64 desktop artifacts only. Upstream remains the
source for macOS, Linux, Windows x64, hosted web, and npm CLI releases.

## Workflow scope

`.github/workflows/release.yml` runs for stable `v*.*.*` tags and manual stable
or nightly dispatches. It:

- runs the Vite+ quality checks, typecheck, and tests;
- builds the Linux x64 `node-pty` prebuild needed by the packaged WSL backend;
- builds one Windows ARM64 NSIS installer on `windows-11-arm`;
- publishes only `.exe`, `.blockmap`, and updater `.yml` assets;
- never publishes the CLI, hosted web app, macOS, Linux, or Windows x64 artifacts.

Nightly releases are manual. They use the current desktop version as the base,
append the nightly date and workflow run number, publish as prereleases, and do
not update `main`.

## Local unsigned test build

From a Windows ARM64 checkout:

```powershell
vp install
vp run dist:desktop:artifact --platform win --target nsis --arch arm64 --output-dir release-local
```

The NSIS installer and updater metadata are written to `release-local`.

## Optional Azure Trusted Signing

The workflow enables signing only when all of these secrets are present:

- `AZURE_TENANT_ID`
- `AZURE_CLIENT_ID`
- `AZURE_CLIENT_SECRET`
- `AZURE_TRUSTED_SIGNING_ENDPOINT`
- `AZURE_TRUSTED_SIGNING_ACCOUNT_NAME`
- `AZURE_TRUSTED_SIGNING_CERTIFICATE_PROFILE_NAME`
- `AZURE_TRUSTED_SIGNING_PUBLISHER_NAME`

Without them, the workflow produces an unsigned installer suitable for testing.

## Release procedure

1. Sync `main` with `origin/main` and merge `upstream/main`.
2. Run `vp fmt`, `vp lint`, `vp run typecheck`, and the relevant tests.
3. Build and install the unsigned ARM64 EXE locally.
4. After approval, push the merge commit.
5. Create and push a stable tag, or run the workflow manually.
6. Confirm the release contains only Windows ARM64 `.exe`, `.blockmap`, and `.yml` files.
7. Smoke-test the downloaded installer and auto-update metadata.
