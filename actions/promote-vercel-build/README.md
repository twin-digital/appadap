# promote-vercel-build

This is an opinionated action for promoting Vercel deployments to production. It finds a staged production Vercel deployment by commit SHA and promotes it to the live production domain. The action resolves git refs to commits, validates CI status, finds the exact deployment using Vercel's API, and promotes it using the official Vercel promotion system.

**Note:** This action is designed for a deployment workflow where production builds are staged first, tested, and then explicitly promoted.

## Usage

### Basic usage

```yaml
      - uses: twin-digital/appadap/actions/promote-vercel-build@v1
        with:
          vercel-project-id: ${{ vars.VERCEL_PROJECT_ID }}
          vercel-team-slug: ${{ vars.VERCEL_TEAM_SLUG }}
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
```

### Promote specific commit

```yaml
      - uses: twin-digital/appadap/actions/promote-vercel-build@v1
        with:
          ref: 'abc1234567890abcdef1234567890abcdef123456'
          vercel-project-id: ${{ vars.VERCEL_PROJECT_ID }}
          vercel-team-slug: ${{ vars.VERCEL_TEAM_SLUG }}
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
```

### Promote tagged release

```yaml
      - uses: twin-digital/appadap/actions/promote-vercel-build@v1
        with:
          ref: 'v1.2.3'
          vercel-project-id: ${{ vars.VERCEL_PROJECT_ID }}
          vercel-team-slug: ${{ vars.VERCEL_TEAM_SLUG }}
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
```

## Prerequisites

This action expects a specific deployment workflow:

- Vercel deployments must be tagged with `--meta commitSha=<SHA>` during deployment
- The target commit must have a successful CI workflow run (named `ci.yaml` by default)
- Only one production deployment should exist per commit SHA
- GitHub CLI (`gh`) must be available for API access
- Vercel CLI (`vercel`) must be installed
- `jq` must be installed for JSON parsing

## Inputs

### `ci-workflow-file`

**Required:** No  
**Default:** `ci.yaml`

Workflow file used to verify CI success before promoting.

### `debug`

**Required:** No  
**Default:** `false`

Enable verbose debug logging.

### `ref`

**Required:** No  
**Default:** `main`

Git ref of the build to promote (branch, tag, or commit SHA). The action will resolve this to a commit SHA before finding the deployment.

### `vercel-project-id`

**Required:** Yes

Vercel project ID.

### `vercel-scope`

**Required:** Yes

Vercel scope (team or personal) passed to the CLI as --scope.

### `vercel-token`

**Required:** Yes

Vercel API token for authenticating with Vercel's API. This should be team-scoped and least-privileged (only promotion 
is needed). To ensure the value is masked in any logs, it is recommended to pass this from a secret value.

## Outputs

### `commit-sha`

Commit SHA of the promoted build.

### `preview-url`

Original deployment URL of the promoted build.

### `ref`

Git ref of the promoted build.
