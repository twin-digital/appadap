# install-docker

This action installs Docker CLI and docker-compose in user-space without requiring sudo. It intelligently detects
existing installations and only installs what's needed. Both tools are installed independently and cached to speed up
subsequent builds.

## Usage

### With defaults

```yaml
- uses: twin-digital/appadap/actions/install-docker@v1
```

### With custom versions

```yaml
- uses: twin-digital/appadap/actions/install-docker@v1
  with:
    docker-version: '27.0.0'
    docker-compose-version: '2.39.0'
```

## Inputs

### `docker-version`

**Required:** No  
**Default:** `28.5.0`

The version of Docker CLI to install. The action will check if Docker is already installed and skip installation if the
existing version is equal to or newer than the specified version.

### `docker-compose-version`

**Required:** No  
**Default:** `2.40.0`

The version of docker-compose to install. The action will check if docker-compose is already installed and skip
installation if the existing version is equal to or newer than the specified version.

### `architecture`

**Required:** No  
**Default:** `auto`

The target architecture for the binaries. When set to `auto`, the action will automatically detect the architecture.
Supported values: `auto`, `x86_64`, `aarch64`.
