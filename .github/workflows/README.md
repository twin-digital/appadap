# Reusable Workflows

The `appadap` project provides several opinionated workflows which can be reused by other projects.

## CI

A comprehensive CI workflow that runs linting, building, unit tests, and optionally end-to-end (E2E) tests for your project.

### Inputs

| Input        | Type     | Default      | Description                                                                     |
| ------------ | -------- | ------------ | ------------------------------------------------------------------------------- |
| `e2e_script` | `string` | `"test:e2e"` | The npm script name to execute for E2E tests                                    |
| `run_e2e`    | `string` | `"auto"`     | Controls E2E test execution. Values: `'true'`, `'false'`, or `'auto'` (default) |

### E2E Auto-Detect Behavior

The workflow includes intelligent auto-detection for E2E test execution based on the `run_e2e` input:

**Decision Priority** (highest to lowest):

1. **Workflow input** - If `run_e2e` is explicitly set to `'true'` or `'false'`, that value is used
2. **Repository/Organization variable** - Falls back to the `RUN_E2E` GitHub variable if defined
3. **Auto-detection** - If `run_e2e` is `'auto'` (default), the workflow checks if the specified `e2e_script` exists in `package.json`
4. **Default** - If none of the above apply, defaults to `false`

**Auto-detection logic:**

- Inspects `package.json` for the presence of the script named by `e2e_script`
- If the script exists: E2E tests will run
- If the script doesn't exist or `package.json` is missing: E2E tests will be skipped

### Prerequisites

- **`.nvmrc` file** - Your project must include an `.nvmrc` file at the repository root to specify the Node.js version
- **E2E test infrastructure** - E2E tests are expected to spin up their own infrastructure and application instance (e.g., using Docker Compose). There is no separate deployment step before running E2E tests.
- **Build artifacts** - The E2E job expects a `vercel-build-output` artifact containing a `next-build.tgz` file (automatically created by the `lint_build_test` job)

### Usage Example

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: twin-digital/appadap/.github/workflows/ci.yaml@v0
    with:
      e2e_script: 'test:e2e' # optional, this is the default
      run_e2e: 'auto' # optional, this is the default
```

### Jobs

The workflow consists of three jobs:

1. **`evaluate`** - Determines whether E2E tests should run based on inputs and auto-detection
2. **`lint_build_test`** - Runs linting, building, and unit tests; creates build artifacts
3. **`e2e`** - Conditionally runs E2E tests if enabled, installs Playwright and Docker, and uploads test results

### Artifacts

- **`vercel-build-output`** - Next.js build output (created by `lint-build-test` job)
- **`playwright-report`** - E2E test results and Playwright reports (created by `e2e` job, retained for 30 days)
