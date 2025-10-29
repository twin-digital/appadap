# Changesets

This project uses [Changesets](https://github.com/changesets/changesets) for version management and changelog generation.

## How to use Changesets

### 1. Create a changeset for your changes

When you make changes that should trigger a new release, run:

```bash
pnpm changeset
```

This will:

- Ask you which packages should be bumped (select common-lift)
- Ask for the type of change (patch, minor, major)
- Ask for a summary of the changes

### 2. Version and publish (maintainers only)

The release process is automated via GitHub Actions when changes are merged to the `main` branch:

1. **Version PR**: Changesets will automatically create a "Version Packages" PR that updates package.json and generates CHANGELOG.md
2. **Release**: When the Version PR is merged, the release workflow will automatically publish the new version

### Manual commands (if needed)

```bash
# Check the status of changesets
pnpm changeset:status

# Apply version bumps locally (normally done by GitHub Actions)
pnpm changeset:version

# Publish packages (normally done by GitHub Actions)
pnpm changeset:publish
```

## Types of changes

- **patch** (0.0.X): Bug fixes, small improvements
- **minor** (0.X.0): New features, backwards compatible changes
- **major** (X.0.0): Breaking changes

For more information, see the [Changesets documentation](https://github.com/changesets/changesets/blob/main/docs/intro-to-using-changesets.md).
