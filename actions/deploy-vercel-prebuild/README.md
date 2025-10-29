# deploy-vercel-prebuild

This is an opinionated, batteries-included deployment action for Vercel applications. It handles the complete deployment workflow including downloading prebuilt artifacts, configuring environment variables, running database migrations, deploying to Vercel, and managing GitHub deployment status with automatic PR comments for preview deployments.

**Note:** This action is designed for projects following a standardized deployment pipeline with pnpm, Prisma migrations, and specific artifact conventions.

## Usage

### Deploy to preview environment

```yaml
      - uses: twin-digital/appadap/actions/deploy-vercel-prebuild@v1
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-scope: ${{ vars.VERCEL_SCOPE }}
          environment: preview
```

### Deploy to staging environment

```yaml
      - uses: twin-digital/appadap/actions/deploy-vercel-prebuild@v1
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-scope: ${{ vars.VERCEL_SCOPE }}
          environment: staging
          description: 'Staging deployment for PR #${{ github.event.number }}'
```

### Deploy to production environment

```yaml
      - uses: twin-digital/appadap/actions/deploy-vercel-prebuild@v1
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-scope: ${{ vars.VERCEL_SCOPE }}
          environment: production
          description: 'Production deployment from main branch'
```

## Prerequisites

This action expects a standardized project setup:

- `.vercel/project.json` must exist in the repository root (created by running `vercel link` locally and should be committed)
- The specified artifact must exist and contain a valid Vercel prebuild (`.out/next-build.tgz` with `.next/` directory)
- Node.js and pnpm must be available in the runner environment
- Vercel CLI must be available (via `npx vercel`)
- Project must have a `db:migrate` script that runs database migrations (typically Prisma)
- GitHub token with repo permissions (automatically provided in Actions)

## Inputs

### `artifact-name`

**Required:** No  
**Default:** `vercel-build-output`

Name of the GitHub artifact containing the prebuild.

### `description`

**Required:** No  
**Default:** `Automated deployment`

Description for the GitHub deployment record.

### `environment`

**Required:** Yes

Vercel environment to deploy to. Allowed values: `preview`, `staging`, or `production`.

- **preview**: Creates a preview deployment with unique URL and PR comment
- **staging**: Creates a staged production build (uses `--prod` flag but not automatically aliased to production domain)
- **production**: Creates a production build (uses `--prod` flag)

### `vercel-scope`

**Required:** Yes

Vercel scope (team or personal) passed to the CLI as `--scope`.

### `vercel-token`

**Required:** Yes

Vercel API token for authenticating with Vercel's API.

## Outputs

### `deployment-id`

GitHub deployment ID for status tracking.

### `deployment-url`

URL of the deployed Vercel application.
