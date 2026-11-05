## Changelog

### v0.1.0 (2026-11-05)

#### Bug Fixes
- **fix(auth): preserve OIDC credentials during deployment**: Prevent empty `aws-access-key-id` and `aws-secret-access-key` inputs from overwriting temporary `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` variables exported by `aws-actions/configure-aws-credentials`.
- **fix(install): fix macOS, Windows, and multi-version Hugo binary downloads**:
  - Support macOS universal `.tar.gz` and `.pkg` releases (`darwin-universal`).
  - Use `curl -fsSL` instead of `wget` and handle `hugo.exe` properly on Windows runners.
  - Automatically fall back across `hugo_extended_withdeploy`, `hugo_extended`, and modern/legacy naming conventions.

#### Security & Modernization
- **security: prevent script injection in composite steps**: Pass all user inputs via step `env:` variables instead of inline `${{ inputs.* }}` interpolation inside `run:` scripts.
- **chore(deps): upgrade inner actions to v6**: Update `actions/cache` from `v3` to `v6` and `aws-actions/configure-aws-credentials` from `v1` to `v6`.

#### New Features
- **feat: support `hugo-version: latest` (and default to `latest`)**: Automatically resolve the latest Hugo version via the GitHub Releases API when `hugo-version` is omitted or set to `latest` (also strips leading `v` if provided).
- **feat: add `working-directory` input**: Support building and deploying Hugo sites located in subdirectories.
- **feat: improve theme and module fetching**: Use `git submodule update --init --recursive` and automatically run `hugo mod get` when a `go.mod` file is present.
- **feat: add action outputs and GitHub Step Summary**: Expose `hugo-version-installed` and `build-path` outputs and generate a markdown deployment summary in `$GITHUB_STEP_SUMMARY`.
- **ci: add cross-platform end-to-end build matrix**: Test the action across Ubuntu, macOS, and Windows with multiple Hugo versions in `.github/workflows/linter.yml`.

---

### Previous Changes

#### New Features
- **feat: make the action more portable**: The action works on Linux, Windows, and macOS runners with x64 and ARM64 architectures.
- **feat: support both OIDC and access key authentication**: Support OpenID Connect (OIDC) and AWS Access Keys.

#### Bug Fixes
- **fix: update actions/checkout version in linter workflow**: Updated `actions/checkout` in `linter.yml` to `v4`.
- **fix: make aws-region not required**: `aws-region` input is not required when not using OIDC.

#### Other Changes
- **refactor: improve readability of deploy-to-s3 step**
- **ci: add actionlint workflow**
- **docs: update Hugo version to 0.151.2 in README**
- **feat: add error handling**
