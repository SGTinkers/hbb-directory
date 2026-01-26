# ADR-001: Turbo Monorepo with pnpm

**Date**: 2026-01-26
**Status**: Accepted
**Deciders**: Team

## Context

We needed to set up a monorepo structure to manage multiple applications and shared packages efficiently. The system needs to support:
- Multiple apps (frontend, backend, etc.)
- Shared packages/libraries
- Fast builds and efficient caching
- Good developer experience

## Decision

We decided to use **Turborepo** with **pnpm** as our monorepo solution.

## Alternatives Considered

### 1. Nx
- **Pros**: Feature-rich, great tooling, strong community
- **Cons**: More complex setup, steeper learning curve, opinionated

### 2. Lerna
- **Pros**: Mature, well-established
- **Cons**: Slower than modern alternatives, less active development

### 3. Yarn Workspaces (alone)
- **Pros**: Simple, built into Yarn
- **Cons**: No build orchestration, no caching layer

## Consequences

### Positive
- **Fast builds**: Turborepo's caching significantly speeds up builds
- **Simple configuration**: Minimal setup required
- **Efficient package management**: pnpm saves disk space with content-addressable storage
- **Great DX**: Simple commands, clear output
- **Scalable**: Works well as the monorepo grows

### Negative
- **Learning curve**: Team needs to learn Turbo's task pipeline system
- **Less mature than some alternatives**: Turbo is newer (though backed by Vercel)

### Neutral
- **Vercel-optimized**: Works especially well with Vercel deployment (may be beneficial later)
- **Growing ecosystem**: Community and tooling still expanding

## References

- [Turborepo Documentation](https://turbo.build)
- [pnpm Documentation](https://pnpm.io)
