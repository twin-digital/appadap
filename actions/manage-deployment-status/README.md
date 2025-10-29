# manage-deployment-status

This action creates GitHub deployment records and manages their status lifecycle throughout the deployment process. It integrates with GitHub's Deployments API to provide deployment tracking in the GitHub UI.

**Security Note:** This action automatically skips execution on forked pull requests to prevent unauthorized deployment attempts.

## Usage

### Create deployment and set to in_progress

```yaml
      - name: Create deployment
        id: create
        uses: twin-digital/appadap/actions/manage-deployment-status@v1
        with:
          environment: preview
          description: 'Preview deployment for PR #${{ github.event.number }}'
          status: in_progress
```

### Update deployment to success

```yaml
      - name: Update deployment to success
        uses: twin-digital/appadap/actions/manage-deployment-status@v1
        with:
          deployment-id: ${{ steps.create.outputs.deployment-id }}
          environment: preview
          deployment-url: ${{ steps.deploy.outputs.url }}
          status: success
```

### Update deployment to failure

```yaml
      - name: Update deployment to failure
        if: failure()
        uses: twin-digital/appadap/actions/manage-deployment-status@v1
        with:
          deployment-id: ${{ steps.create.outputs.deployment-id }}
          environment: preview
          status: failure
```

### Complete deployment workflow

```yaml
      - name: Create deployment
        id: create
        uses: twin-digital/appadap/actions/manage-deployment-status@v1
        with:
          environment: production
          description: 'Production deployment'
          status: in_progress
      
      # ... perform actual deployment ...
      
      - name: Mark deployment successful
        if: success()
        uses: twin-digital/appadap/actions/manage-deployment-status@v1
        with:
          deployment-id: ${{ steps.create.outputs.deployment-id }}
          environment: production
          deployment-url: https://myapp.vercel.app
          status: success
      
      - name: Mark deployment failed
        if: failure()
        uses: twin-digital/appadap/actions/manage-deployment-status@v1
        with:
          deployment-id: ${{ steps.create.outputs.deployment-id }}
          environment: production
          status: failure
```

## Prerequisites

- `GITHUB_TOKEN` must have `deployments: write` permission
- Repository must have GitHub Actions enabled
- Environment names must match repository environment settings (if environment protection rules are configured)

## Inputs

### `deployment-id`

**Required:** No

Existing deployment ID for status updates. Omit this when creating a new deployment. Provide this when updating an existing deployment's status.

### `deployment-url`

**Required:** No

Deployment URL to display in the GitHub Deployments UI. This appears as a clickable link next to successful deployments.

**Security Warning:** If the URL contains secrets (query parameters, tokens, API keys), they will be visible in the GitHub UI to anyone with repository access. Only pass public URLs without sensitive information.

### `description`

**Required:** No  
**Default:** `Automated deployment`

Human-readable description of the deployment shown in the GitHub UI.

### `environment`

**Required:** Yes

Deployment environment name. Must be exactly one of: `Preview`, `Staging`, or `Production` (case-sensitive, capitalized).

**Important:** Environment names are case-sensitive and must match exactly.

### `github-token`

**Required:** No  
**Default:** `${{ github.token }}`

GitHub token for API access. The default `GITHUB_TOKEN` is usually sufficient, but you may need a custom token with specific permissions.

**Required permissions:**
- `deployments: write` - Create and update deployment records
- `statuses: write` - Update deployment statuses

### `status`

**Required:** Yes

Deployment status. Must be one of: `in_progress`, `success`, or `failure`.

- `in_progress`: Deployment is actively happening
- `success`: Deployment completed successfully
- `failure`: Deployment failed

## Outputs

### `deployment-id`

GitHub deployment ID for the created or updated deployment. Use this to update the deployment status in subsequent steps.

## Event Compatibility

### Works with:
- `push` events
- `pull_request` events from the same repository
- `workflow_dispatch` events
- Any event where `GITHUB_TOKEN` has deployment permissions

### Does NOT work with:
- **Pull requests from forked repositories** - The action automatically skips execution with a warning. Forks don't have access to secrets or deployment permissions for security reasons.

## GitHub UI Integration

Deployments created by this action appear in:
- Repository's **Environments** tab (if environment protection rules are configured)
- Repository's **Deployments** section in the right sidebar
- Pull request timeline (for PR-related deployments)
- Commit status checks

## Security Considerations

- **Forked PR protection**: Automatically skips on forked pull requests to prevent unauthorized deployments
- **URL secrets**: Never pass secrets in `deployment-url` - they will be publicly visible in the GitHub UI
- **Token permissions**: Ensure `GITHUB_TOKEN` has minimum required permissions (`deployments: write`, `statuses: write`)
- **Environment protection**: Use exact case-sensitive environment names to ensure protection rules apply correctly
