# install-dependencies

This action handles the common setup tasks needed across all jobs in our workflows:

- Sets up pnpm package manager
- Sets up Node.js using the version specified in `.nvmrc`
- Installs dependencies with pnpm

## Usage

```yaml
      - uses: twin-digital/appadap/actions/install-dependencies@v1
```

## Prerequisites

- Project should specify the required node version in an `.nvmrc` file
