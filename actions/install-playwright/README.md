# install-playwright

This action installs Playwright browsers and system dependencies for e2e testing. Caches the playwright browser
downloads to speed up subsequent builds.

## Usage

```yaml
- uses: twin-digital/appadap/actions/install-playwright@v1
```

## Prerequisites

- `pnpm` is installed
- `package.json` has a valid version of `@playwright/test` specified as a devDependency
