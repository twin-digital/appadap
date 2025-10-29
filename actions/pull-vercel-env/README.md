# pull-vercel-env

This action pulls environment variables from Vercel and saves them to a file using the Vercel CLI. It validates inputs, sets restrictive file permissions to protect secrets, and provides feedback on the operation.

**Security Note:** This action automatically skips execution on forked pull requests to prevent secret exposure. When it does run, it writes secrets to disk temporarily. Clean up the environment file when no longer needed.

## Usage

### Basic usage

```yaml
- name: Pull Vercel environment variables
  id: pull
  uses: twin-digital/appadap/actions/pull-vercel-env@v1
  with:
    vercel-token: ${{ secrets.VERCEL_TOKEN }}
    environment: production

# Use environment variables for your steps
- name: Run command with env
  run: pnpm run some-command
  env:
    # Environment variables are now available

# Clean up when done
- name: Remove environment file
  if: always()
  run: rm -f ${{ steps.pull.outputs.envfile-path }}
```

### Custom output file

```yaml
- name: Pull Vercel environment variables
  uses: twin-digital/appadap/actions/pull-vercel-env@v1
  with:
    vercel-token: ${{ secrets.VERCEL_TOKEN }}
    environment: preview
    file: .env.preview
```

## Prerequisites

- `.vercel/project.json` must exist in the repository root. This file is created when you run `vercel link` locally and contains the project ID and org ID needed to pull environment variables from the correct Vercel project. The file should be committed to your repository.
- Vercel CLI must be available (via `npx vercel`)
- **Important:** Do not commit `.env` or other environment files containing secrets to your repository. Add them to `.gitignore`.

## Inputs

### `environment`

**Required:** Yes

Vercel environment to pull variables from. Allowed values: `preview` or `production`.

### `file`

**Required:** No  
**Default:** `.env`

Output file path to store environment variables. The file will be created with restrictive permissions (600) to prevent unauthorized access.

### `vercel-token`

**Required:** Yes

Vercel API token for authenticating with Vercel's API.

## Outputs

### `envfile-path`

Path to the generated environment file. Use this to clean up the file when done: `rm -f ${{ steps.pull.outputs.envfile-path }}`

## Security Considerations

- **Forked PR protection**: The action automatically skips execution on pull requests from forked repositories, preventing secret exposure to untrusted code
- **Restrictive permissions**: Files are created with `umask 077` (permissions 600), ensuring only the owner can read/write
- **Temporary use**: Many workflows only need the environment file for a few steps. Always clean up when done using `rm -f ${{ steps.pull.outputs.envfile-path }}` in an `if: always()` step
- **Git ignore**: Ensure `.env` and similar files are in your `.gitignore` to prevent accidental commits of secrets
