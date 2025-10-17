## Changelog

### New Features

-   **feat: make the action more portable**
    -   The action is now portable and works on Linux, Windows, and macOS runners with x64 and ARM64 architectures.
-   **feat: support both OIDC and access key authentication**
    -   The action now supports two authentication methods: OpenID Connect (OIDC) and AWS Access Keys.

### Bug Fixes

-   **fix: update actions/checkout version in linter workflow**
    -   The `actions/checkout` version in the `linter.yml` workflow has been updated to `v4` to fix a compatibility issue with the runner.

### Other Changes

-   **refactor: improve readability of deploy-to-s3 step**
    -   The `deploy-to-s3` step has been refactored to make it more readable.
-   **ci: add actionlint workflow**
    -   A new workflow has been added to lint the action.
-   **docs: update Hugo version to 0.151.1 in README**
    -   The Hugo version in the `README.md` file has been updated to the latest version.
-   **feat: add error handling**
    -   Error handling has been added to the action.
