# Reusable Workflows

The `appadap` project provides several opinionated workflows which can be reused by other projects.

## CI

A comprehensive CI workflow that runs linting, building, unit tests, and optionally end-to-end (E2E) tests for your project.

### Inputs

| Input              | Type     | Default       | Description                                                                            |
| ------------------ | -------- | ------------- | -------------------------------------------------------------------------------------- |
| `validate_scripts` | `string` | `"lint,test"` | Comma-delimited list of npm scripts to run for validation (run in parallel via matrix) |
| `build_type`       | `string` | `"auto"`      | Build system to use: `'auto'` (default), `'vercel'`, `'nextjs'`, `'skip'`              |
| `e2e_script`       | `string` | `"test:e2e"`  | The npm script name to execute for E2E tests                                           |
| `run_e2e`          | `string` | `"auto"`      | Controls E2E test execution. Values: `'true'`, `'false'`, or `'auto'` (default)        |

### Build Type Auto-Detection

The workflow supports multiple build systems through the `build_type` input:

**Supported Build Types:**

- **`vercel`** - Builds using Vercel CLI, creates `.next` build artifacts
  - Production: `vercel build --prod`
  - Preview: `vercel build`
  - Artifacts: `.next/`, `.vercel/`
  - Secrets required: `VERCEL_TOKEN`

- **`nextjs`** - Builds using Next.js standalone (without Vercel)
  - Runs: `pnpm run build`
  - Artifacts: `.next/`
  - Secrets required: None

- **`skip`** - Skips the build step entirely (useful for backend-only projects or when no build is needed)

**Auto-Detection Logic:**

When `build_type` is set to `'auto'` (default), the workflow inspects your repository:

1. If `vercel.json` exists or `.vercel/` directory is present → uses `'vercel'` build type
2. Else if `next.config.js`, `next.config.ts`, or `next.config.mjs` exists → uses `'nextjs'` build type
3. Otherwise → uses `'skip'` build type

**Manual Override:**

You can explicitly set the build type to bypass auto-detection:

```yaml
with:
  build_type: 'vercel' # Force vercel build even without vercel.json
```

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
- **Build type requirements:**
  - **`vercel`** - Requires `VERCEL_TOKEN` secret to be configured
  - **`nextjs`** - No secrets required
  - **`skip`** - No build-specific requirements
- **E2E test infrastructure** - E2E tests are expected to spin up their own infrastructure and application instance (e.g., using Docker Compose). There is no separate deployment step before running E2E tests.
- **Build artifacts** - If a build is performed, the E2E job will automatically download and restore the build artifacts

### Usage Example

**Basic usage (with auto-detection):**

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
      build_type: 'auto' # optional, this is the default
      e2e_script: 'test:e2e' # optional, this is the default
      run_e2e: 'auto' # optional, this is the default
```

**Advanced example with explicit configuration:**

```yaml
jobs:
  ci:
    uses: twin-digital/appadap/.github/workflows/ci.yaml@v0
    with:
      validate_scripts: 'lint,test,typecheck,db:generate-client'
      build_type: 'vercel'
      e2e_script: 'test:e2e:ci'
      run_e2e: 'true'
```

**Backend-only project (no build needed):**

```yaml
jobs:
  ci:
    uses: twin-digital/appadap/.github/workflows/ci.yaml@v0
    with:
      validate_scripts: 'lint,test'
      build_type: 'skip'
      run_e2e: 'false'
```

### Jobs

The workflow consists of five jobs:

1. **`detect_build_type`** - Determines the build type based on `build_type` input and project files (when set to `'auto'`)
2. **`validate`** - Runs validation scripts in parallel using a matrix strategy. Each script specified in `validate_scripts` runs as a separate job instance.
3. **`build`** - Builds the application based on the detected build type. Runs in parallel with `validate` to minimize pipeline duration. Skipped entirely if `build_type` is `'skip'`.
4. **`evaluate`** - Determines whether E2E tests should run based on inputs and auto-detection
5. **`e2e`** - Conditionally runs E2E tests if enabled (depends on `validate`, `build`, `detect_build_type`, and `evaluate` completing). Automatically downloads and restores build artifacts if applicable. Installs Playwright and Docker, and uploads test results.

### Artifacts

Artifacts are created based on the build type:

- **`build-output`** - Build artifacts (created by `build` job when `build_type` is not `'skip'`)
  - **`vercel`**: Contains `.next/` build directory and `.vercel/` configuration
  - **`nextjs`**: Contains `.next/` build directory
- **`playwright-report`** - E2E test results and Playwright reports (created by `e2e` job, retained for 30 days)
