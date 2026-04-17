# CI quality gates

- `.github/workflows/ci.yml` runs `bun run lint`, `bun run typecheck`, and `bun run test` on pull requests and pushes to `main`.
- `.github/workflows/release.yml` builds only Windows `arm64` desktop artifacts from a `v*.*.*` tag or manual dispatch and publishes one GitHub release.
- The release workflow does not build macOS, Linux, or Windows x64 artifacts, and it does not publish the CLI package to npm.
- The release workflow auto-enables Windows signing only when Azure Trusted Signing secrets are present. Without secrets, it still releases unsigned artifacts.
- See `docs/release.md` for full release/signing setup checklist.
