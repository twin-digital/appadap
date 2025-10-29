# Reusable Workflows

The `appadap` project provides several opinionated workflows which can be reused by other projects.

## CI

A comprehensive CI workflow that runs linting, building, unit tests, and optionally end-to-end (E2E) tests for your project.

### Inputs

| Input              | Type     | Default       | Description                                                                            |
| ------------------ | -------- | ------------- | -------------------------------------------------------------------------------------- |
| `validate_scripts` | `string` | `"lint,test"` | Comma-delimited list of npm scripts to run for validation (run in parallel via matrix) |
| `e2e_script`       | `string` | `"test:e2e"`  | The npm script name to execute for E2E tests                                           |
| `run_e2e`          | `string` | `"auto"`      | Controls E2E test execution. Values: `'true'`, `'false'`, or `'auto'` (default)        |

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
- **Validation scripts** - Your project's `package.json` must contain the scripts specified in `validate_scripts` (defaults: `lint` and `test`)
- **E2E test infrastructure** - E2E tests are expected to spin up their own infrastructure and application instance (e.g., using Docker Compose). There is no separate deployment step before running E2E tests.
- **Build artifacts** - The E2E job expects a `vercel-build-output` artifact containing a `next-build.tgz` file (automatically created by the `build` job)

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
      validate_scripts: 'lint,test' # optional, this is the default
      e2e_script: 'test:e2e' # optional, this is the default
      run_e2e: 'auto' # optional, this is the default
```

**Advanced example with custom validation scripts:**

```yaml
jobs:
  ci:
    uses: twin-digital/appadap/.github/workflows/ci.yaml@v0
    with:
      validate_scripts: 'lint,test,typecheck,db:generate-client'
      e2e_script: 'test:e2e:ci'
      run_e2e: 'true'
```

### Jobs

The workflow consists of four jobs:

1. **`validate`** - Runs validation scripts in parallel using a matrix strategy. Each script specified in `validate_scripts` runs as a separate job instance.
2. **`build`** - Builds the Vercel application and creates build artifacts. Runs in parallel with `validate` to minimize pipeline duration.
3. **`evaluate`** - Determines whether E2E tests should run based on inputs and auto-detection
4. **`e2e`** - Conditionally runs E2E tests if enabled (depends on `validate`, `build`, and `evaluate` completing successfully). Installs Playwright and Docker, and uploads test results.

### Artifacts

- **`vercel-build-output`** - Next.js build output (created by `build` job)
- **`playwright-report`** - E2E test results and Playwright reports (created by `e2e` job, retained for 30 days)
