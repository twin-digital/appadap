# @twin-digital/appadap

## 0.4.2

### Patch Changes

- a477bdb: update e2e job to use fast runners

## 0.4.1

### Patch Changes

- 0fe1c3c: update action references to be absolute

## 0.4.0

### Minor Changes

- 98a37bd: BREAKING: update expected artifact name for deploy-vercel-prebuild to "build-output"

## 0.3.0

### Minor Changes

- 5092182: add CI workflow for validating and building projects

### Patch Changes

- 5092182: resolve CI deployment failures caused by circular check requirements
- 10084c6: switch changesets to github-api mode so commits are signed
- 5092182: introduce linting and fix pre-existing issues

## 0.2.0

### Minor Changes

- 309b032: standardize on lowercase environment names everywhere

## 0.1.4

### Patch Changes

- 0e58657: fix "missing version" errors when using the deploy-vercel-prebuild or promote-vercel-build actions

## 0.1.3

### Patch Changes

- 0ae8c3c: publish major & minor version alias tags

## 0.1.2

### Patch Changes

- b14471f: apply additional fix to tag publishing

## 0.1.1

### Patch Changes

- 64c577c: fix tag publishing failures

## 0.1.0

### Minor Changes

- 64d90ca: add shared actions
  - deploy-vercel-prebuild
  - install-dependencies
  - install-docker
  - install-playwright
  - manage-deployment-status
  - promote-vercel-build
  - pull-vercel-env
