# appadap

> Dap your app!

Foundational library for Twin Digital applications. Accelerates development and helps _dapped apps_ (that is, apps which
use `appadap`) adhere to our preferred development patterns and practices.

## Frameworks & Technologies

Dapped apps use the following frameworks and technologies:

### Development

- **pnpm** for package management & **nvm** for node version management
- **Next.js 15** with React 19 and TypeScript
- **Prisma ORM** with PostgreSQL for database management
- **TailwindCSS** for styling

### Testing

- **Vitest** for unit and integration testing
- **Playwright** for end-to-end testing

### CI/CD & Infrastructure

- **GitHub Actions** for building, testing, etc.
- **Vercel** for hosting applications
- **Prisma Data Platform** for PostgreSQL hosting

## Feature Overview

- GitHub actions to facilitate our CI/CD process
- Complete GitHub Actions workflows defining our pipelines

## Using `appadap`

TBD

## Developing `appadap`

```bash
# Install dependencies
pnpm install

# Start development server
pnpm dev

# Run unit tests
pnpm test

# Run e2e tests
pnpm test:e2e

# Build for production
pnpm build
```

For detailed development information, see [docs/DEVELOPER.md](./docs/DEVELOPER.md).
