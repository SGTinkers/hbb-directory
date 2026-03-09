# HBB Directory

A Turborepo monorepo managed with pnpm.

## Structure

```
hbb-directory/
├── apps/          # Application packages
├── packages/      # Shared packages
├── docs/          # Project documentation
│   ├── prds/          # Product Requirements Documents
│   ├── architecture/  # System architecture & diagrams
│   ├── decisions/     # Architecture Decision Records (ADRs)
│   ├── backlog/       # Product backlog & user stories
│   ├── research/      # Research & landscape analysis
│   └── archive/       # Archived/superseded docs
├── design.pen     # UI/UX design file (Pencil format)
├── package.json
├── pnpm-workspace.yaml
└── turbo.json
```

## Documentation

Project documentation lives in the [`docs/`](docs/) directory. See [`docs/README.md`](docs/README.md) for full details.

- **PRDs** (`docs/prds/`) — Feature specifications and product requirements
- **Architecture** (`docs/architecture/`) — System design and workflow diagrams
- **ADRs** (`docs/decisions/`) — Architecture Decision Records tracking key technical choices
- **Research** (`docs/research/`) — Market and landscape research

## Design

The UI/UX design is maintained in `design.pen` (Pencil format). It contains:

- **Mobile screens** (390px) — Homepage, Search Results, Listing Detail, Category Landing, All Categories, Empty Search State
- **Desktop screens** (1280px) — Desktop versions of all the above
- **Design system** — Navy & Coral Pink palette, Plus Jakarta Sans (headlines), Inter (body)

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
