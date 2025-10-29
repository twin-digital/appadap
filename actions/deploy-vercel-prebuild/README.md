# Deploy Vercel Prebuild Action

This composite action handles the complete deployment workflow for Vercel applications with GitHub integration:

- Creates GitHub deployment records
- Downloads prebuild artifacts
- Configures environment variables
- Runs database migrations
- Deploys to Vercel (preview or production)
- Updates GitHub deployment status
- Adds PR comments for preview deployments

## Inputs

### `vercel-token` (required)

**Description:** Vercel API token  
**Type:** string

### `environment` (required)

**Description:** Vercel environment to deploy to  
**Type:** string  
**Allowed values:** `preview` or `production`

### `artifact-name` (optional)

**Description:** Name of the GitHub artifact containing the prebuild  
**Type:** string  
**Default:** `vercel-build-output`

### `description` (optional)

**Description:** Description for the GitHub deployment  
**Type:** string  
**Default:** `Automated deployment`

## Outputs

### `deployment-url`

**Description:** URL of the deployed Vercel application  
**Type:** string

### `deployment-id`

**Description:** GitHub deployment ID for status tracking  
**Type:** string

## What This Action Does

1. **Creates** a GitHub deployment record with the specified description
2. **Updates** deployment status to "in_progress"
3. **Validates** the environment input (must be "preview" or "production")
4. **Downloads** the specified GitHub artifact containing the Vercel prebuild
5. **Sets up** environment variables using the `pull-vercel-env` action
6. **Runs** database migrations (`pnpm run db:migrate`)
7. **Deploys** the prebuild to Vercel with appropriate flags for the target environment
8. **Updates** deployment status to "success" or "failure" based on result
9. **Adds/updates** PR comment with deployment link (preview only)
10. **Returns** the deployment URL and deployment ID for use in subsequent steps

## GitHub Integration Features

### Deployment Status

- Creates deployment records visible in GitHub's Deployments tab
- Shows deployment progress in PR checks
- Updates status throughout deployment lifecycle
- Provides direct links to deployed environments

### PR Comments (Preview Only)

For preview deployments on pull requests, automatically:

- Creates a formatted comment with deployment link
- Updates existing comments instead of creating duplicates
- Includes commit SHA for reference
- Shows professional deployment status with emoji

## Example Usage

### Deploy to preview environment (Pull Request)

```yaml
- name: Deploy to preview
  id: deploy
  uses: ./.github/actions/deploy-vercel-prebuild
  with:
    vercel-token: ${{ secrets.VERCEL_TOKEN }}
    environment: preview
    description: 'Preview deployment for PR #${{ github.event.number }}'

- name: Use outputs
  run: |
    echo "Deployed to ${{ steps.deploy.outputs.deployment-url }}"
    echo "GitHub deployment ID: ${{ steps.deploy.outputs.deployment-id }}"
```

### Deploy to production (Main Branch)

```yaml
- name: Deploy to production
  id: deploy
  uses: ./.github/actions/deploy-vercel-prebuild
  with:
    vercel-token: ${{ secrets.VERCEL_TOKEN }}
    environment: production
    description: 'Production deployment from main branch'

- name: Run tests against deployment
  run: pnpm run test:e2e:smoke
  env:
    PLAYWRIGHT_BASE_URL: ${{ steps.deploy.outputs.deployment-url }}
```

## Deployment Behavior

### Preview Environment

- Uses `npx vercel deploy --archive=tgz --prebuilt --yes`
- Creates a preview deployment with a unique URL
- Updates PR with deployment link and status
- Creates "preview" environment in GitHub

### Production Environment

- Uses `npx vercel deploy --archive=tgz --prebuilt --prod --yes`
- Creates a staged production build (not automatically aliased to the production domain)
- Creates "production" environment in GitHub
- Allows for testing before promoting to live

## Requirements

- The specified artifact must exist and contain a valid Vercel prebuild
- Node.js and pnpm must be available in the runner environment
- The Vercel CLI must be available (usually via `npx vercel`)
- A valid Vercel API token with deployment permissions
- Database migration scripts must be available via `pnpm run db:migrate`
- GitHub token with repo permissions (automatically provided in Actions)

## Dependencies

This action depends on:

- `actions/download-artifact@v4` - for downloading the prebuild artifact
- `actions/github-script@v7` - for GitHub API integration
- `./.github/actions/pull-vercel-env` - for setting up environment variables

## Error Handling

- Validates environment input before proceeding
- Updates GitHub deployment status to "failure" on errors
- Checks that the deployment URL was successfully captured
- Fails fast if any step encounters an error
- Provides clear error messages for debugging

## Post-Deployment Integration

The action outputs can be used with the companion `update-deployment-status` action to update deployment status after tests:

```yaml
- name: Update deployment after tests
  uses: ./.github/actions/update-deployment-status
  with:
    deployment-id: ${{ steps.deploy.outputs.deployment-id }}
    deployment-url: ${{ steps.deploy.outputs.deployment-url }}
    test-status: ${{ job.status }}
    environment: preview
```
