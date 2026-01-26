# HBB Directory

A Turborepo monorepo managed with pnpm.

## Structure

```
hbb-directory/
├── apps/          # Application packages
├── packages/      # Shared packages
├── package.json
├── pnpm-workspace.yaml
└── turbo.json
```

## Getting Started

### Install dependencies

```bash
pnpm install
```

### Development

```bash
pnpm dev
```

### Build

```bash
pnpm build
```

### Lint

```bash
pnpm lint
```

### Test

```bash
pnpm test
```

## Adding Apps/Packages

Create new apps in `apps/` directory and shared packages in `packages/` directory.

Each app/package should have its own `package.json` with a unique name.
