# ADR-002: MVP Tech Architecture

**Date**: 2026-03-09
**Status**: Accepted
**Deciders**: Team

## Context

We need to finalize the tech architecture for the HBB Directory MVP. The project is non-profit, so minimizing cost and operational complexity is critical. The PRD initially assumed Next.js, Vercel, separate admin dashboard, and a deployed crawler service — these assumptions were revisited.

Key constraints:
- $0/mo target for infrastructure (excluding domain)
- Minimal ops burden — small team, no dedicated DevOps
- SEO is critical — directory needs to rank on Google
- Instagram crawler needed for supply pipeline
- Avoid Vercel vendor lock-in

## Decision

### Frontend: TanStack Start

**Chosen over**: Next.js, Astro, SvelteKit, Nuxt

TanStack Start provides SSR via Nitro, deploys anywhere (not tied to Vercel), and stays in the React ecosystem. While younger than Next.js, it avoids the complexity and vendor lock-in concerns.

- SSR for SEO-critical pages (listing detail, category pages, search results)
- React ecosystem for component libraries (shadcn/ui)
- Nitro adapter for flexible deployment targets

### Styling: Tailwind CSS + shadcn/ui

Utility-first CSS with copy-paste accessible components. shadcn/ui provides both consumer-facing and admin UI components without adding a runtime dependency.

### Backend: Supabase (Free Tier)

**Chosen over**: Neon + NextAuth + R2, Vercel Postgres + Auth.js + R2

Supabase bundles PostgreSQL, Auth, and Storage into one platform:
- **Database**: PostgreSQL, 500MB on free tier — sufficient for MVP
- **Auth**: Built-in auth for admin login, supports email/password
- **Storage**: 1GB free for re-hosted listing images
- **Row Level Security**: Secure API access without a separate backend

One platform replaces three separate services, reducing integration complexity.

### Web Hosting: Railway / Render

**Chosen over**: Vercel, Cloudflare Pages, Netlify

Consolidates hosting on one platform. TanStack Start outputs a Node server via Nitro, which Railway/Render can run directly. Free tier available.

### Crawler: Local Puppeteer/Playwright (Manually Triggered)

**Chosen over**: Apify, Brightdata, deployed service on Railway, GitHub Actions cron

The crawler runs locally on a developer machine, not as a deployed service:
- **Non-headless browser**: Avoids headless detection, can use visible browser
- **Manually triggered**: No cron scheduling needed
- **No proxy rotation initially**: Start without proxies, add if blocked
- **$0 cost**: No hosting, no proxy service fees
- **Simpler debugging**: Visual browser makes it easy to inspect issues

The crawler writes raw data to Supabase. Admin then reviews and approves listings through the web app.

### Admin Dashboard: Same App

**Chosen over**: Separate Next.js/React app

Admin routes (`/admin/*`) live in the same TanStack Start app, protected by Supabase Auth. This avoids duplicating deployment infrastructure and shares UI components.

### Image Storage: Supabase Storage (Re-hosted)

**Chosen over**: Cloudflare R2, Instagram hotlinking

Scraped images are downloaded and re-hosted to Supabase Storage. This ensures images don't break if Instagram posts are deleted or accounts go private. 1GB free tier is sufficient for MVP.

### Planning Area Data: data.gov.sg

Singapore's ~55 planning areas will be seeded from data.gov.sg open data. This provides an official, maintained data source.

### Legal Approach: Public Data

Scraping public Instagram profiles for directory purposes. Data kept minimal and links back to original profiles.

## Monorepo Structure

```
apps/
  web/              ← TanStack Start (consumer + admin routes)
  crawler/          ← Puppeteer/Playwright scripts, run locally
packages/
  db/               ← Supabase client, generated types, query helpers
  shared/           ← Shared constants (categories, planning areas), types
```

## Cost Analysis

| Component | Service | Monthly Cost |
|-----------|---------|-------------|
| Database + Auth + Storage | Supabase (free tier) | $0 |
| Web Hosting | Railway / Render (free tier) | $0 |
| Crawler | Local machine | $0 |
| Domain | TBD | ~$1/mo |
| **Total** | | **~$1/mo** |

## Consequences

### Positive
- Near-zero infrastructure cost
- Single platform for DB + Auth + Storage (Supabase) reduces complexity
- Local crawler avoids headless detection and hosting costs
- No Vercel vendor lock-in
- React ecosystem available via TanStack Start

### Negative
- TanStack Start is less mature than Next.js — fewer tutorials, smaller community
- Local crawler requires a developer to manually trigger runs
- Railway/Render free tiers have limitations (may need to upgrade as traffic grows)
- Supabase free tier has limits (500MB DB, 1GB storage, 50K monthly active users)

### Risks
- TanStack Start SSR may have rough edges for SEO (sitemaps, structured data need manual setup)
- Instagram may block scraping even with non-headless browser
- Free tier limits may be hit sooner than expected if the directory grows

## References

- [TanStack Start](https://tanstack.com/start)
- [Supabase](https://supabase.com)
- [Railway](https://railway.app)
- [shadcn/ui](https://ui.shadcn.com)
- [data.gov.sg Planning Areas](https://data.gov.sg)
