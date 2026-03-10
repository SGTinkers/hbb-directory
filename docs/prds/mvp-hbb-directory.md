# MVP: Home-Based Business Directory (Food & Bakes)

**Status**: Draft
**Owner**: TBD
**Created**: 2026-03-09
**Last Updated**: 2026-03-10

---

## 1. Overview

A web-based directory for discovering home-based food and baking businesses in Singapore. The MVP focuses on the Food & Bakes vertical to build listing density in a high-demand category before expanding to other verticals.

The platform serves two sides:
- **Consumers** who want to discover and contact home-based food businesses in their area
- **Supply pipeline** powered by an Instagram crawler and admin curation to populate listings

### Problem Statement

Home-based businesses (HBBs) in Singapore lack visibility. They cannot put up physical signage and rely heavily on Instagram, Carousell, and word-of-mouth. Consumers have no centralized, searchable way to discover HBBs by category or location. Existing platforms (Instagram, Facebook groups) are fragmented and not optimized for discovery.

### Product Vision

Become the go-to directory for discovering home-based businesses in Singapore, starting with food and baked goods.

---

## 2. Goals & Success Metrics

### Primary Goal

Validate the concept: prove that a curated directory of home-based food businesses provides value to consumers and can attract organic traffic.

### Success Metrics (MVP)

| Metric | Target | Rationale |
|--------|--------|-----------|
| Listings live | 50–100 food HBBs | Minimum density to be useful |
| Organic traffic | First indexed pages on Google | SEO validation |
| Click-to-contact rate | Track baseline | Measure consumer intent |
| Qualitative feedback | 10+ user interviews | Validate value proposition |

---

## 3. User Stories

### Consumer

- **As a consumer**, I want to browse home-based food businesses by category (e.g., cakes, cookies, nasi lemak) so I can discover options near me.
- **As a consumer**, I want to search for a specific type of food or business name so I can quickly find what I'm looking for.
- **As a consumer**, I want to filter businesses by my planning area (e.g., Tampines, Bishan) so I can find HBBs nearby.
- **As a consumer**, I want to see a business's photos, description, contact info, and social links so I can decide whether to order from them.
- **As a consumer**, I want to click through to a business's WhatsApp or Instagram so I can place an order or enquire.

### Admin

- **As an admin**, I want to review crawler-discovered listings and approve/reject them so only legitimate businesses are shown.
- **As an admin**, I want to manually create and edit listings so I can add businesses that the crawler missed.
- **As an admin**, I want to manage listing categories and planning areas so the directory stays organized.

### Crawler System

- **As the system**, I want to discover HBB accounts via Instagram hashtags (e.g., #sghomebaker, #homebakesg) so we can grow the directory.
- **As the system**, I want to scrape profile data from a seed list of known HBB accounts so we can populate initial listings.
- **As the system**, I want to extract business name, description, photos, and contact info from Instagram profiles so admins can review and approve listings.

---

## 4. Requirements

### 4.1 Functional Requirements

#### Consumer-Facing Directory (Web App)

| ID | Requirement | Priority |
|----|------------|----------|
| F-01 | Homepage with featured/recent listings and category browsing | Must |
| F-02 | Listing detail page with full business info, photos, social links | Must |
| F-03 | Keyword search across business name, description, and tags | Must |
| F-04 | Filter by food sub-category (e.g., cakes, cookies, meals, snacks) | Must |
| F-05 | Filter by planning area (~55 Singapore planning areas) | Must |
| F-06 | Combined search + filter (keyword + category + area) | Must |
| F-07 | SEO-optimized pages (SSR, meta tags, structured data) | Must |
| F-08 | Mobile-responsive design | Must |
| F-09 | Click-to-contact actions (WhatsApp, Instagram, phone) | Must |
| F-10 | Category landing pages (e.g., /cakes, /cookies) for SEO | Should |

#### Listing Data Model

Each listing contains:

| Field | Required | Source |
|-------|----------|--------|
| Business name | Yes | Crawler / Admin |
| Slug (URL-friendly) | Yes | Auto-generated |
| Description | Yes | Crawler / Admin |
| Food sub-category | Yes | Admin-assigned |
| Planning area | Yes | Admin-assigned |
| Contact: WhatsApp number | No | Crawler / Admin |
| Contact: Phone | No | Crawler / Admin |
| Instagram handle | No | Crawler / Admin |
| Other social links | No | Admin |
| Photos (up to 5) | No | Crawler / Admin |
| Operating hours | No | Admin |
| Delivery/pickup availability | No | Admin |
| Tags/keywords | No | Admin |
| Status | Yes | System (draft/pending/approved/rejected/archived) |
| Source | Yes | System (crawler/manual) |
| Instagram profile URL | No | Crawler |
| Created at | Yes | System |
| Updated at | Yes | System |

#### Instagram Crawler System

| ID | Requirement | Priority |
|----|------------|----------|
| C-01 | Crawl Instagram hashtags to discover HBB accounts | Must |
| C-02 | Scrape Instagram profiles from a curated seed list | Must |
| C-03 | Extract: display name, bio, profile photo, website, contact info | Must |
| C-04 | Extract: recent post images (for listing photos) | Should |
| C-05 | Deduplicate against existing listings (by Instagram handle) | Must |
| C-06 | Store raw crawl data separately from curated listing data | Must |
| C-07 | Support incremental crawls (don't re-process known accounts) | Should |
| C-08 | Configurable hashtag list and seed account list | Must |
| C-09 | Rate limiting and respectful crawling practices | Must |
| C-10 | AI classification: detect if account is a genuine HBB | Must |
| C-11 | AI extraction: auto-assign food sub-category and tags | Must |
| C-12 | AI extraction: extract contacts, hours, planning area from bio text | Must |
| C-13 | AI generation: generate clean business description from bio | Should |

**AI Classification**: The crawler uses Claude API (Haiku for cost efficiency) to:
1. **HBB Detection** — Determine if an Instagram account is genuinely a home-based business (vs commercial/personal)
2. **Category & Tag Extraction** — Auto-assign food sub-category and relevant tags
3. **Entity Extraction** — Parse unstructured bio text into structured fields (WhatsApp, hours, area)
4. **Description Generation** — Create clean business descriptions from emoji-heavy Instagram bios

**Initial hashtag seed list** (configurable):
- #sghomebaker, #homebakesg, #sgbakes, #homebakingsg
- #sghomecook, #sgfoodie, #homecooksg
- #sgcakes, #sgcookies, #sgpastry

#### Admin Dashboard

| ID | Requirement | Priority |
|----|------------|----------|
| A-01 | Admin authentication (simple, e.g., email/password) | Must |
| A-02 | View all listings with status filters (draft/pending/approved/rejected/archived) | Must |
| A-03 | Review and approve/reject crawler-discovered listings | Must |
| A-04 | Edit any listing field before approving | Must |
| A-05 | Manually create new listings | Must |
| A-06 | Archive/remove listings | Must |
| A-07 | Manage food sub-categories | Should |
| A-08 | Manage planning area list | Should |
| A-09 | View crawler run history and stats | Nice to have |
| A-10 | Trigger crawler runs manually | Nice to have |

### 4.2 Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Page load time | < 2s for listing pages |
| Mobile performance | Lighthouse score > 80 |
| SEO | SSR with proper meta tags, Open Graph, JSON-LD structured data |
| Availability | 99% uptime (standard for MVP) |
| Data freshness | Crawler runs at least weekly |
| Security | Admin auth, no public write access, sanitized inputs |

---

## 5. Design & User Experience

### 5.1 Consumer Web App

#### Key Pages

1. **Homepage**
   - Hero section with search bar
   - Browse by food sub-category (visual grid)
   - Featured/newest listings
   - Brief intro to what the directory is

2. **Search Results / Browse Page**
   - Listing cards in grid layout
   - Sidebar or top-bar filters (category, area)
   - Search bar
   - Infinite scroll
   - Empty state with suggestions

3. **Listing Detail Page**
   - Business name, description
   - Photo gallery
   - Contact actions (WhatsApp, Instagram, phone) — prominent CTAs
   - Category and area tags
   - Operating hours, delivery/pickup info

4. **Category Landing Pages**
   - SEO-optimized pages per sub-category
   - Filtered listing grid

#### Design Principles

- Clean, modern, mobile-first
- Fast — minimal JS, optimized images
- Contact actions should be the most prominent CTAs on listing pages
- No account required for consumers

### 5.2 Admin Dashboard

- Functional over beautiful — prioritize speed of moderation workflow
- Table-based listing management
- Inline editing where possible
- Bulk actions for approve/reject

---

## 6. Technical Considerations

### Tech Stack

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| Monorepo | Turborepo + pnpm | Already set up (ADR-001) |
| Consumer web app + Admin | TanStack Start (SSR via Nitro) | SSR for SEO, React ecosystem, no vendor lock-in (ADR-002) |
| Styling | Tailwind CSS + shadcn/ui | Utility-first CSS with accessible components |
| Database + Auth + Storage | Supabase (free tier) | Bundled Postgres + Auth + Storage for $0 (ADR-002) |
| Crawler | Puppeteer/Playwright (local) | Non-headless, manually triggered, $0 cost (ADR-002) |
| Hosting | Railway / Render (free tier) | No Vercel lock-in, runs Node server (ADR-002) |
| Image storage | Supabase Storage | Re-hosted images, 1GB free, integrated with backend |

### Architecture (High-Level)

```
┌────────────────────────────────┐     ┌──────────────────────┐
│  Web App (TanStack Start)       │     │  Instagram Crawler    │
│  ├─ Consumer pages (SSR)        │     │  (Puppeteer, local)   │
│  └─ Admin routes (/admin/*)     │     │  Manually triggered   │
└──────────────┬─────────────────┘     └──────────┬───────────┘
               │                                   │
               └──────────┬────────────────────────┘
                          │
                    ┌─────▼──────────────────────┐
                    │  Supabase                   │
                    │  ├─ PostgreSQL (listings,    │
                    │  │   crawl data, categories) │
                    │  ├─ Auth (admin login)       │
                    │  └─ Storage (listing images) │
                    └────────────────────────────┘
```

### Instagram Crawler Considerations

- **Runs locally**: Non-headless browser on a developer machine, manually triggered. Avoids hosting costs and headless detection.
- **No proxies initially**: Start with direct requests; add proxy rotation only if Instagram blocks requests.
- **Rate limiting**: Must respect Instagram's rate limits to avoid IP blocks.
- **Legal**: Scraping public Instagram profiles for directory purposes — keep data minimal and link back to original profiles.
- **Raw data separation**: Crawler stores raw scraped data in Supabase; admin curates and approves into the listings table.

### Database Schema (Simplified)

Key tables:
- `listings` — approved/curated business listings
- `crawl_sources` — raw data from Instagram crawler
- `categories` — food sub-categories
- `planning_areas` — Singapore planning areas (seeded from data.gov.sg)
- `crawl_runs` — crawler execution history

Note: Admin users are managed via Supabase Auth (no separate admins table needed).

---

## 7. Scope Boundaries

### In Scope (MVP)

- Consumer directory web app (browse, search, filter, view listings)
- Instagram crawler system (hashtag + seed list discovery)
- Admin dashboard (moderation, manual entry, listing management)
- Food & Bakes category only
- Singapore planning areas for location

### Out of Scope (Post-MVP)

- HBB owner self-registration / claiming listings
- User accounts for consumers
- Reviews and ratings
- Additional verticals beyond food (beauty, tutoring, crafts, etc.)
- Booking or ordering functionality
- Payment processing
- Mobile app (native)
- Notifications (email, push)
- Analytics dashboard for HBB owners
- Monetization features (featured listings, ads)

---

## 8. Timeline & Milestones

**Target**: Ship MVP in 2–4 weeks

| Week | Milestone | Deliverables |
|------|-----------|-------------|
| 1 | Foundation | Database schema, API scaffolding, crawler prototype, Next.js app setup |
| 2 | Core features | Listing pages (SSR), search + filters, crawler pipeline working, admin CRUD |
| 3 | Integration & polish | Admin moderation flow, crawler → approval pipeline, SEO optimization, responsive design |
| 4 | Launch prep | Seed 50+ listings via crawler, QA, deploy to production, soft launch |

---

## 9. Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|-----------|------------|
| Instagram blocks scraping | No supply pipeline | Medium | Use proxy rotation, respect rate limits, have manual entry as fallback |
| Low listing density at launch | Poor user experience | Medium | Prioritize crawler + manual curation in week 1-2; focus on one category |
| Scraped data quality is poor | High admin overhead | Medium | Build good admin editing UX; extract only high-signal fields |
| SEO takes time to rank | Low organic traffic initially | High | Complement with social media sharing; proper technical SEO from day 1 |
| Scope creep | Delayed launch | Medium | Strict MVP boundaries; post-MVP backlog for everything else |

---

## 10. Open Questions

1. ~~**Instagram scraping approach**~~: **Resolved** — Custom Puppeteer/Playwright, run locally with non-headless browser, manually triggered. See ADR-002.
2. ~~**Image hosting**~~: **Resolved** — Re-host to Supabase Storage. See ADR-002.
3. **Domain name**: What domain will the directory live on? — TBD, to be decided before launch.
4. ~~**Legal review**~~: **Resolved** — Proceeding with scraping public Instagram data. Keep data minimal, link back to original profiles.
5. ~~**Planning area data source**~~: **Resolved** — Use data.gov.sg for planning area list.
