# MVP Crawler System - Workflow Diagrams

**Created**: 2026-03-09
**Last Updated**: 2026-03-09
**Related**: [MVP PRD](../prds/mvp-hbb-directory.md) | [ADR-002](../decisions/ADR-002-mvp-tech-architecture.md)

This document contains workflow diagrams for the MVP crawler system. The MVP crawler is a locally-run Puppeteer/Playwright script that scrapes Instagram, classifies profiles via Claude API, and stores results in Supabase.

---

## 1. High-Level System Architecture

```mermaid
graph TB
    subgraph "Local Machine (Developer)"
        CRAWLER[Crawler Script<br/>Puppeteer/Playwright<br/>Non-headless browser]
        AI_CLIENT[Claude API Client]
    end

    subgraph "External"
        IG[Instagram<br/>Public profiles]
        CLAUDE[Claude API<br/>Haiku model]
    end

    subgraph "Supabase"
        DB[(PostgreSQL<br/>listings, crawl_sources,<br/>categories, planning_areas)]
        AUTH[Auth<br/>Admin login]
        STORAGE[Storage<br/>Listing images]
    end

    subgraph "Railway / Render"
        WEB[TanStack Start App<br/>Consumer pages + Admin routes]
    end

    CRAWLER -->|Scrape public profiles| IG
    CRAWLER -->|Send profile data| AI_CLIENT
    AI_CLIENT -->|Classify & extract| CLAUDE
    CRAWLER -->|Store raw + processed data| DB
    CRAWLER -->|Upload images| STORAGE

    WEB -->|Query listings| DB
    WEB -->|Admin auth| AUTH
    WEB -->|Serve images| STORAGE
```

---

## 2. Crawler Pipeline Flow

The end-to-end flow from discovery to storage.

```mermaid
flowchart TD
    START([Manual Trigger<br/>Developer runs script]) --> CONFIG[Load Config<br/>Hashtag list + seed accounts]

    CONFIG --> MODE{Discovery<br/>Mode?}

    MODE -->|Hashtag| HASHTAG[Browse hashtag pages<br/>#sghomebaker, #sgbakes, etc.]
    MODE -->|Seed List| SEED[Load seed account list<br/>from config file]

    HASHTAG --> COLLECT[Collect Instagram<br/>profile URLs]
    SEED --> COLLECT

    COLLECT --> DEDUP_CHECK{Already in<br/>Supabase?<br/>Check by IG handle}

    DEDUP_CHECK -->|Yes, skip| NEXT_PROFILE{More<br/>profiles?}
    DEDUP_CHECK -->|No, new| SCRAPE[Scrape Profile Page]

    SCRAPE --> EXTRACT[Extract Raw Data<br/>- Display name<br/>- Bio text<br/>- Profile photo URL<br/>- Website link<br/>- Post count, followers<br/>- Recent post images]

    EXTRACT --> AI_PROCESS[Send to Claude API<br/>for classification]

    AI_PROCESS --> HBB_CHECK{Is HBB?<br/>Confidence ≥ 0.5?}

    HBB_CHECK -->|No| STORE_REJECTED[Store as rejected<br/>in crawl_sources]
    HBB_CHECK -->|Yes| ENRICH[Enrich with AI results<br/>- Category & tags<br/>- Contacts, hours, area<br/>- Clean description]

    ENRICH --> DOWNLOAD_IMG[Download & upload<br/>images to Supabase Storage]

    DOWNLOAD_IMG --> STORE_PENDING[Store in crawl_sources<br/>status: pending_review]

    STORE_REJECTED --> NEXT_PROFILE
    STORE_PENDING --> NEXT_PROFILE

    NEXT_PROFILE -->|Yes| DEDUP_CHECK
    NEXT_PROFILE -->|No| SUMMARY([Print Summary<br/>Discovered: N<br/>New HBBs: N<br/>Rejected: N])
```

---

## 3. AI Classification Workflow

How Claude API processes each scraped profile.

```mermaid
flowchart TD
    START([Profile Data<br/>from Scraper]) --> BUILD_PROMPT[Build Classification Prompt]

    BUILD_PROMPT --> INCLUDE[Include in prompt:<br/>- Display name<br/>- Bio text<br/>- Hashtags from recent posts<br/>- Follower/post count<br/>- Location if available]

    INCLUDE --> CALL_CLAUDE[Call Claude API<br/>Model: Haiku<br/>Single prompt, structured JSON output]

    CALL_CLAUDE --> PARSE{Valid JSON<br/>response?}

    PARSE -->|No| RETRY{Retry<br/>count < 2?}
    RETRY -->|Yes| CALL_CLAUDE
    RETRY -->|No| FALLBACK[Use rule-based fallback<br/>Keyword matching on bio]

    PARSE -->|Yes| EXTRACT_RESULTS[Extract Results]

    EXTRACT_RESULTS --> HBB_RESULT[HBB Detection<br/>isHBB: bool<br/>confidence: 0-1<br/>reasoning: string]

    EXTRACT_RESULTS --> CAT_RESULT[Category & Tags<br/>subCategory: string<br/>tags: string array]

    EXTRACT_RESULTS --> ENTITY_RESULT[Entity Extraction<br/>whatsapp: string<br/>phone: string<br/>email: string<br/>hours: string<br/>planningArea: string]

    EXTRACT_RESULTS --> DESC_RESULT[Description<br/>cleanDescription: string]

    FALLBACK --> HBB_RESULT

    HBB_RESULT --> RETURN([Return All Results<br/>to Crawler Pipeline])
    CAT_RESULT --> RETURN
    ENTITY_RESULT --> RETURN
    DESC_RESULT --> RETURN
```

**Single-prompt approach**: All four tasks (HBB detection, categorization, entity extraction, description generation) are handled in one Claude API call to minimize cost and latency. The prompt requests a single structured JSON response containing all fields.

**Example prompt structure**:
```
Analyze this Instagram profile and determine if it's a Singapore home-based food business.

Profile:
- Name: {name}
- Bio: {bio}
- Hashtags: {hashtags}
- Followers: {followers}
- Posts: {post_count}

Return JSON with:
1. isHBB (bool), confidence (0-1), reasoning
2. foodSubCategory (from: cakes, cookies, meals, snacks, desserts, beverages, dietary-specific)
3. tags (relevant keywords)
4. contacts (whatsapp, phone, email extracted from bio)
5. operatingHours (if mentioned)
6. planningArea (Singapore area if mentioned)
7. cleanDescription (2-3 sentence business description)
```

---

## 4. Admin Review & Approval Flow

How admins review crawler results and publish listings.

```mermaid
flowchart TD
    START([Admin logs in<br/>/admin via Supabase Auth]) --> DASHBOARD[Admin Dashboard<br/>View pending reviews]

    DASHBOARD --> FILTERS[Filter by:<br/>- Status: pending/approved/rejected<br/>- Confidence score<br/>- Category]

    FILTERS --> SELECT[Select a crawl_source<br/>entry to review]

    SELECT --> REVIEW[Review Screen<br/>- AI-extracted data<br/>- Original IG profile link<br/>- Scraped images<br/>- Confidence score<br/>- AI reasoning]

    REVIEW --> DECISION{Admin<br/>Decision?}

    DECISION -->|Reject| REJECT[Mark crawl_source<br/>as rejected]
    DECISION -->|Edit & Approve| EDIT[Edit listing fields<br/>- Fix name, description<br/>- Adjust category/tags<br/>- Correct contacts<br/>- Set planning area]
    DECISION -->|Approve as-is| APPROVE_DIRECT[Use AI-extracted<br/>data directly]

    EDIT --> CREATE_LISTING[Create listing record<br/>status: approved]
    APPROVE_DIRECT --> CREATE_LISTING

    CREATE_LISTING --> LINK[Link listing to<br/>crawl_source record]

    LINK --> PUBLISH[Listing visible<br/>on consumer site]

    REJECT --> NEXT{More to<br/>review?}
    PUBLISH --> NEXT

    NEXT -->|Yes| SELECT
    NEXT -->|No| DONE([Review session complete])
```

---

## 5. Duplicate Detection Flow

How the crawler avoids creating duplicate entries.

```mermaid
flowchart TD
    START([New Profile<br/>Scraped from IG]) --> NORMALIZE[Normalize Data<br/>- Lowercase IG handle<br/>- Strip @ prefix<br/>- Normalize phone to +65 format]

    NORMALIZE --> CHECK_HANDLE{IG handle exists<br/>in crawl_sources?}

    CHECK_HANDLE -->|Yes, exact match| EXISTING[Found existing record]

    EXISTING --> STATUS{Existing<br/>status?}

    STATUS -->|rejected| SKIP_REJECTED[Skip<br/>Previously rejected]
    STATUS -->|pending/approved| CHECK_STALE{Last crawled<br/>> 30 days ago?}

    CHECK_STALE -->|Yes| REFRESH[Update raw data<br/>Re-run AI classification]
    CHECK_STALE -->|No| SKIP_RECENT[Skip<br/>Recently crawled]

    CHECK_HANDLE -->|No match| CHECK_FUZZY[Fuzzy Check<br/>- Similar business name?<br/>- Same phone/whatsapp?<br/>- Same email?]

    CHECK_FUZZY --> FUZZY_RESULT{Potential<br/>duplicate?}

    FUZZY_RESULT -->|Same contact info| FLAG[Flag for manual review<br/>Store as pending_review<br/>with duplicate_of reference]

    FUZZY_RESULT -->|No match| NEW[Create new<br/>crawl_source entry]

    SKIP_REJECTED --> NEXT([Next Profile])
    SKIP_RECENT --> NEXT
    REFRESH --> NEXT
    FLAG --> NEXT
    NEW --> NEXT
```

---

## 6. Data Quality Scoring Flow

How each crawled profile gets a quality/completeness score.

```mermaid
flowchart TD
    START([Profile Data<br/>After AI Processing]) --> INIT[Initialize Score = 0]

    INIT --> CHECK_NAME{Has business<br/>name?}
    CHECK_NAME -->|Yes| ADD_NAME[Score + 10]
    CHECK_NAME -->|No| SKIP_NAME[Skip]

    ADD_NAME --> CHECK_DESC
    SKIP_NAME --> CHECK_DESC

    CHECK_DESC{Has clean<br/>description?}
    CHECK_DESC -->|Yes| ADD_DESC[Score + 15]
    CHECK_DESC -->|No| SKIP_DESC[Skip]

    ADD_DESC --> CHECK_PHOTO
    SKIP_DESC --> CHECK_PHOTO

    CHECK_PHOTO{Has profile<br/>photo?}
    CHECK_PHOTO -->|Yes| ADD_PHOTO[Score + 15]
    CHECK_PHOTO -->|No| SKIP_PHOTO[Skip]

    ADD_PHOTO --> CHECK_CONTACT
    SKIP_PHOTO --> CHECK_CONTACT

    CHECK_CONTACT{Has 1+<br/>contact method?}
    CHECK_CONTACT -->|Yes| ADD_CONTACT[Score + 15]
    CHECK_CONTACT -->|No| SKIP_CONTACT[Skip]

    ADD_CONTACT --> CHECK_MULTI
    SKIP_CONTACT --> CHECK_CATEGORY

    CHECK_MULTI{Has 2+<br/>contact methods?}
    CHECK_MULTI -->|Yes| ADD_MULTI[Score + 5]
    CHECK_MULTI -->|No| SKIP_MULTI[Skip]

    ADD_MULTI --> CHECK_CATEGORY
    SKIP_MULTI --> CHECK_CATEGORY

    CHECK_CATEGORY{Has food<br/>sub-category?}
    CHECK_CATEGORY -->|Yes| ADD_CAT[Score + 10]
    CHECK_CATEGORY -->|No| SKIP_CAT[Skip]

    ADD_CAT --> CHECK_AREA
    SKIP_CAT --> CHECK_AREA

    CHECK_AREA{Has planning<br/>area?}
    CHECK_AREA -->|Yes| ADD_AREA[Score + 10]
    CHECK_AREA -->|No| SKIP_AREA[Skip]

    ADD_AREA --> CHECK_IMAGES
    SKIP_AREA --> CHECK_IMAGES

    CHECK_IMAGES{Has 2+<br/>post images?}
    CHECK_IMAGES -->|Yes| ADD_IMG[Score + 10]
    CHECK_IMAGES -->|No| SKIP_IMG[Skip]

    ADD_IMG --> CHECK_HOURS
    SKIP_IMG --> CHECK_HOURS

    CHECK_HOURS{Has operating<br/>hours?}
    CHECK_HOURS -->|Yes| ADD_HOURS[Score + 10]
    CHECK_HOURS -->|No| SKIP_HOURS[Skip]

    ADD_HOURS --> FINAL
    SKIP_HOURS --> FINAL

    FINAL[Final Score: 0-100] --> CLASSIFY{Score<br/>range?}

    CLASSIFY -->|≥ 70| HIGH[High Quality<br/>Priority review]
    CLASSIFY -->|40-69| MEDIUM[Medium Quality<br/>Standard review]
    CLASSIFY -->|< 40| LOW[Low Quality<br/>May need manual enrichment]

    HIGH --> STORE[Store quality score<br/>with crawl_source]
    MEDIUM --> STORE
    LOW --> STORE
```

**Scoring breakdown (100 points total)**:

| Component | Points | Rationale |
|-----------|--------|-----------|
| Business name | 10 | Basic identifier |
| Clean description | 15 | Critical for listing page |
| Profile photo | 15 | Visual trust signal |
| 1+ contact method | 15 | Core CTA requirement |
| 2+ contact methods | 5 | Bonus for multiple channels |
| Food sub-category | 10 | Enables filtering |
| Planning area | 10 | Enables location filtering |
| 2+ post images | 10 | Visual product showcase |
| Operating hours | 10 | Useful but often missing |

---

## 7. Error Handling Flow

How the crawler handles common failure scenarios.

```mermaid
flowchart TD
    START([Attempt to<br/>scrape profile]) --> TRY[Navigate to<br/>Instagram profile URL]

    TRY --> RESULT{Page<br/>status?}

    RESULT -->|Success| EXTRACT([Continue to<br/>extraction])

    RESULT -->|Rate limited<br/>429 / login wall| RATE[Rate Limit Hit]
    RESULT -->|Profile not found<br/>404| NOT_FOUND[Profile Deleted]
    RESULT -->|Private account| PRIVATE[Private Account]
    RESULT -->|Network error| NETWORK[Network Error]
    RESULT -->|Page changed<br/>unexpected structure| STRUCTURE[Structure Changed]

    RATE --> PAUSE[Pause crawling<br/>Wait 5-10 minutes]
    PAUSE --> RETRY_RATE{Retry<br/>count < 3?}
    RETRY_RATE -->|Yes| TRY
    RETRY_RATE -->|No| STOP_SESSION[Stop crawler session<br/>Log: rate limited<br/>Resume later]

    NOT_FOUND --> LOG_404[Log as not_found<br/>Skip profile]
    LOG_404 --> NEXT([Next profile])

    PRIVATE --> LOG_PRIVATE[Log as private<br/>Skip profile]
    LOG_PRIVATE --> NEXT

    NETWORK --> RETRY_NET{Retry<br/>count < 3?}
    RETRY_NET -->|Yes| WAIT[Wait 30 seconds]
    WAIT --> TRY
    RETRY_NET -->|No| LOG_NET[Log network error<br/>Skip profile]
    LOG_NET --> NEXT

    STRUCTURE --> LOG_STRUCT[Log warning:<br/>page structure changed<br/>May need crawler update]
    LOG_STRUCT --> NEXT

    STOP_SESSION --> SUMMARY([Print session summary<br/>Completed: N<br/>Skipped: N<br/>Errors: N])
```

**Key error handling principles**:
- **Graceful degradation**: Skip problematic profiles, don't crash the whole run
- **Rate limit respect**: Pause and retry, stop the session if repeatedly limited
- **Visibility**: Log all errors with context for debugging
- **No proxy complexity**: Since we run locally with a visible browser, rate limits are handled by pausing rather than rotating proxies

---

## 8. End-to-End Data Flow

Complete flow from Instagram to consumer-facing listing.

```mermaid
flowchart LR
    subgraph "Local Crawler"
        A[Browse IG Hashtags<br/>& Seed Accounts]
        B[Scrape Profiles]
        C[Claude AI<br/>Classify & Extract]
        D[Download Images]
    end

    subgraph "Supabase"
        E[(crawl_sources<br/>raw + AI data)]
        F[(listings<br/>approved only)]
        G[Storage<br/>images]
    end

    subgraph "Admin Review"
        H[Review pending<br/>crawl_sources]
        I[Edit & approve<br/>or reject]
    end

    subgraph "Consumer Site"
        J[Browse / Search<br/>/ Filter]
        K[Listing Detail<br/>Page]
        L[Click to Contact<br/>WhatsApp / IG]
    end

    A --> B
    B --> C
    C --> E
    B --> D
    D --> G

    E --> H
    H --> I
    I -->|Approve| F

    F --> J
    G --> K
    J --> K
    K --> L
```

---

## Notes

### Diagram Rendering
These diagrams use Mermaid syntax and render on GitHub, GitLab, VS Code (with Mermaid extension), and most documentation platforms.

### Key Differences from Archived Diagrams
| Archived (Full Vision) | MVP |
|---|---|
| Multi-platform (IG, FB, Carousell, TikTok) | Instagram only |
| Redis/RabbitMQ job queue | Simple sequential script |
| Elasticsearch search index | Supabase Postgres full-text search |
| Deployed crawler service | Local, manually triggered |
| Proxy rotation + CAPTCHA solving | No proxies, pause on rate limit |
| CDN for images | Supabase Storage |
| Profile claiming workflow | Not in MVP |
| Multiple AI calls per profile | Single Claude API call per profile |

### Updating These Diagrams
When the crawler design changes:
1. Update the relevant diagram(s)
2. Update "Last Updated" date at top
3. Ensure consistency with [MVP PRD](../prds/mvp-hbb-directory.md) and [ADR-002](../decisions/ADR-002-mvp-tech-architecture.md)
