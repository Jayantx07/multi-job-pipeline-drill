# Pipeline Audit

## lint
- **Intended Purpose**: Run code linting to ensure code style and quality.
- **Current Issue**: Runs in parallel with everything, no sequence enforced. Also lacks a timeout.
- **Fix**: Needs `timeout-minutes: 10`.

## unit-tests
- **Intended Purpose**: Run unit tests for the application.
- **Current Issue**: Runs at the start with everything else, lacks a timeout.
- **Fix**: Add `needs: lint` so it waits for linting to pass, and add `timeout-minutes: 15`.

## build
- **Intended Purpose**: Build the application and generate the `dist/` directory.
- **Current Issue**: Missing `upload-artifact` step to save the build output for subsequent jobs. Also lacks a timeout.
- **Fix**: Add `needs: lint` to sequence it properly, `timeout-minutes: 20`, and an `actions/upload-artifact@v4` step to upload `dist/` with name `app-build`.

## integration-tests
- **Intended Purpose**: Run integration tests using the built application.
- **Current Issue**: Tries to run without waiting for the build to finish, and lacks the built artifact (`dist/`). Missing a timeout.
- **Fix**: Add `needs: build`, `timeout-minutes: 30`, and an `actions/download-artifact@v4` step to download `app-build`.

## deploy-staging
- **Intended Purpose**: Deploy the application to the staging environment after tests pass on the `main` branch.
- **Current Issue**: Runs on every push regardless of branch, and without waiting for tests. No timeout.
- **Fix**: Add `needs: [unit-tests, integration-tests]`, condition `if: github.ref == 'refs/heads/main'`, and `timeout-minutes: 15`.

## deploy-production
- **Intended Purpose**: Deploy the application to the production environment after a successful deployment to staging on the `main` branch.
- **Current Issue**: Runs on every branch and without waiting for staging. No timeout.
- **Fix**: Add `needs: deploy-staging`, condition `if: github.ref == 'refs/heads/main'`, and `timeout-minutes: 15`.

## notify
- **Intended Purpose**: Send a notification about the pipeline run status.
- **Current Issue**: Skipped if upstream jobs fail. No timeout.
- **Fix**: Add `needs: deploy-production`, condition `if: always()`, and `timeout-minutes: 15`.
