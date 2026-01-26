# HBB Crawler System PRD

**Status**: Draft
**Owner**: TBD
**Created**: 2026-01-26
**Last Updated**: 2026-01-26

## 1. Overview

The HBB Crawler System is an automated data discovery and collection system that identifies, extracts, and profiles home-based businesses (HBBs) from social media platforms and online marketplaces in Singapore. This system solves the critical cold-start problem by pre-populating the HBB directory with thousands of business profiles before launch, eliminating the need for manual business owner onboarding.

### The Problem

Creating a directory website is straightforward (basic CRUD operations), but the real challenge is user acquisition. Without existing listings, there's no value for customers; without customer traffic, there's no incentive for businesses to join. This creates a chicken-and-egg problem that prevents most directory platforms from gaining traction.

### The Solution

An AI-powered crawler that:
1. Automatically discovers HBBs across social media platforms
2. Extracts essential business information (name, category, contact, images)
3. Creates pre-populated profiles in the directory
4. Allows business owners to claim and enhance their profiles later

This approach provides immediate value to customers (directory full of listings from day one) while reducing friction for businesses (claim existing profile vs. manual signup).

## 2. Goals & Success Metrics

### Primary Goals

1. **Cold-Start Elimination**: Launch directory with 5,000+ verified HBB profiles
2. **Data Quality**: Achieve >70% average profile completeness score
3. **Coverage**: Represent all major HBB categories (food, services, handmade, etc.)
4. **Accuracy**: Maintain <10% false positive rate (non-HBBs incorrectly classified)

### Success Metrics

**Crawler Performance**:
- Profiles discovered per day: >500
- Data extraction success rate: >85%
- Duplicate detection accuracy: >95%
- Platform coverage: Instagram, Facebook, Carousell (minimum)

**Data Quality**:
- Average profile completeness score: >70/100
- Profiles with 2+ contact methods: >80%
- Profiles with valid images: >90%
- Data freshness (updated within 90 days): >90%

**Business Impact**:
- Directory launch readiness: 5,000+ active profiles
- Claimed profile rate within 3 months: >10%
- Customer search success rate: >60%

## 3. User Stories

### As a Potential Customer
- I want to discover HBBs in my area so that I can support local businesses
- I want to see verified business information so that I can contact them easily
- I want to browse by category (food, services, crafts) so that I can find what I need

### As an HBB Owner
- I want my business to be automatically discovered so that I gain visibility without manual effort
- I want to claim my profile so that I can add more details and own my listing
- I want accurate information displayed so that customers can contact me correctly

### As a Platform Owner
- I want automated profile creation so that the directory has value from day one
- I want high-quality data so that customers trust the platform
- I want to minimize manual data entry so that we can scale efficiently

## 4. Requirements

### 4.1 Data Sources & Target Platforms

#### Primary Sources (High Priority)
- **Instagram**
  - Business/Creator accounts
  - Hashtags: `#sghomebusiness`, `#sghbb`, `#sgbakery`, `#sghandmade`, `#sgfoodie`, `#sghomebasedbusiness`
  - Location tags: Singapore HDB estates
  - Bio indicators: "Home-based", "HBB", "DM to order", "Self-pickup"

- **Facebook**
  - Facebook Pages (not personal profiles)
  - Facebook Groups: "Singapore Home Based Business", "SG HBB"
  - Community group posts
  - Location: Singapore

- **Carousell**
  - Seller profiles with multiple listings
  - Categories: Food, Handmade, Services
  - Singapore-based sellers
  - Indicators: "home-based", "self-collection"

#### Secondary Sources (Medium Priority)
- TikTok (business accounts with Singapore location)
- Google My Business (home-based listings)
- Telegram (public channels/groups)

#### Tertiary Sources (Future Consideration)
- Reddit (r/singapore mentions)
- LinkedIn (sole proprietors, freelancers)
- Personal websites/blogs

### 4.2 Data Extraction Requirements

#### Core Required Fields (Must Extract)

```json
{
  "businessName": "string (3-100 chars)",
  "category": "string (from taxonomy)",
  "displayPicture": "url (valid image, min 200x200px)",
  "description": "string (bio/about section)",
  "contactMethods": {
    "instagram": "string (@username)",
    "facebook": "string (page URL)",
    "whatsapp": "string (+65xxxxxxxx)",
    "telegram": "string (@username or link)",
    "email": "string (valid email)",
    "phone": "string (+65xxxxxxxx)"
  },
  "sourceUrl": "string (original profile URL)",
  "scrapedAt": "timestamp (ISO 8601)"
}
```

**Validation Rules**:
- At least ONE contact method must be present
- Business name cannot be empty or generic
- Display picture must be accessible and valid image format
- Category must match defined taxonomy

#### Secondary Fields (Nice to Have)

```json
{
  "operatingHours": "string (extracted from bio/posts)",
  "address": {
    "area": "string (e.g., Tampines, Jurong, CBD)",
    "postalCode": "string (6-digit Singapore postal)",
    "pickupDetails": "string (e.g., Self-collection only)"
  },
  "priceRange": "enum ($ | $$ | $$$)",
  "productSamples": ["url", "url", "url"],
  "reviews": {
    "platform": "string",
    "rating": "number (1-5)",
    "count": "number"
  },
  "socialMetrics": {
    "followers": "number",
    "posts": "number",
    "engagement": "number (avg likes/comments)"
  },
  "languages": ["string"],
  "lastActive": "timestamp (last post/update)"
}
```

#### Derived/Computed Fields

```json
{
  "isVerified": "boolean (platform verification badge)",
  "confidenceScore": "float (0-1, how confident this is an HBB)",
  "qualityScore": "float (0-100, profile completeness)",
  "activityStatus": "enum (active | dormant | inactive)",
  "tags": ["string", "string"]
}
```

### 4.3 HBB Identification Criteria

The system must determine if an account is genuinely a home-based business using AI classification.

#### Strong Indicators (High Confidence)

Bio contains keywords:
- "Home-based" / "HBB" / "Home business"
- "Self-collection" / "Self-pickup"
- "DM to order" / "WhatsApp to order"
- "Small batch" / "Homemade"
- "Operating from home"
- Postal code or HDB block number

Location signals:
- Singapore HDB estates (Tampines, Bedok, Jurong, etc.)
- Residential area tags

Contact patterns:
- Personal phone numbers (not 1800/6xxx business lines)
- WhatsApp/Telegram only (no physical store address)

Business nature:
- Food: Homemade cakes, cookies, meal prep
- Crafts: Handmade jewelry, art, crochet
- Services: Tuition, freelance, consulting

#### Moderate Indicators (Medium Confidence)

- Uses hashtags: `#sghomebusiness`, `#sghandmade`, `#sgbaker`
- Posts show home environment in background
- Mentions "limited slots" / "pre-order only"
- Small-scale production mentions
- Personal story in bio (solopreneur, side hustle)

#### Weak Indicators (Low Confidence)

- Singapore-based account
- Small follower count (<10k)
- Irregular posting schedule
- Manual/personal tone in captions

#### Exclusion Criteria (Definitely NOT HBB)

- Has physical store address in bio
- Professional photography studio setup
- Corporate website domain
- Registered company name with "Pte Ltd"
- Multiple employees mentioned
- Commercial kitchen/facility shown
- Professional business hours (9am-6pm weekdays)
- Large-scale operations (wholesale, export)

#### AI Classification Logic

```
IF (strong_indicators >= 2 OR 
    (strong_indicators >= 1 AND moderate_indicators >= 2))
  THEN confidence = HIGH (0.8-1.0)
  
ELSE IF (strong_indicators >= 1 OR moderate_indicators >= 3)
  THEN confidence = MEDIUM (0.5-0.79)
  
ELSE IF (moderate_indicators >= 2 OR weak_indicators >= 3)
  THEN confidence = LOW (0.3-0.49)
  
ELSE
  THEN confidence = VERY_LOW (0-0.29) → EXCLUDE
```

**Minimum Confidence Threshold**: 0.5 (medium confidence) to create profile

### 4.4 Categorization Logic & Taxonomy

#### Primary Categories

**1. Food & Beverage**
- Baked Goods (cakes, cookies, brownies, pastries)
- Meal Prep (healthy meals, frozen food, catering)
- Asian Cuisine (kueh, dumplings, noodles)
- Western Cuisine (pasta, pizza, sandwiches)
- Desserts & Sweets (puddings, ice cream, chocolate)
- Beverages (coffee, tea, juices)
- Dietary Specific (vegan, keto, halal, gluten-free)

**2. Handmade & Crafts**
- Jewelry & Accessories
- Art & Prints
- Crochet & Knitting
- Candles & Soaps
- Stationery & Paper Goods
- Toys & Dolls
- Home Decor

**3. Services**
- Tuition & Education (academic, music, art)
- Beauty & Wellness (nail art, massage, therapy)
- Pet Services (grooming, sitting, training)
- Photography (portraits, events)
- Design & Creative (graphic design, web design)
- Consulting (business, financial, career)
- Alterations & Tailoring

**4. Fashion & Apparel**
- Clothing (women, men, children)
- Accessories (bags, shoes, hats)
- Custom/Tailored
- Vintage/Thrifted

**5. Personal Care & Beauty**
- Skincare
- Cosmetics
- Haircare
- Fragrances

**6. Digital Products**
- Printables & Templates
- Digital Art
- E-courses
- Presets & Filters

#### Auto-Categorization via AI

The system uses AI to analyze:
- Bio text and post captions
- Hashtags used
- Product images (image classification)
- Multi-label classification (can belong to multiple categories)
- Confidence score per category

**Example AI Classification**:

Input:
```
Name: "BakeHappySG"
Bio: "Homemade brownies & cookies 🍪 DM to order! Self-collection at Tampines"
Hashtags: #sgbakes #brownies #cookies #homebaker
Images: [photos of brownies, cookies]
```

Output:
```json
{
  "primary": "Food & Beverage > Baked Goods",
  "secondary": ["Food & Beverage > Desserts & Sweets"],
  "confidence": 0.95,
  "tags": ["brownies", "cookies", "halal-friendly", "custom-orders"]
}
```

### 4.5 Data Validation & Quality Requirements

#### Profile Completeness Score (0-100)

| Component | Points |
|-----------|--------|
| Has display picture | +20 |
| Has description | +15 |
| Has 2+ contact methods | +15 |
| Has area/location | +10 |
| Has operating hours | +10 |
| Has 3+ product images | +10 |
| Has price range | +10 |
| Has social metrics | +5 |
| Has reviews | +5 |

#### Freshness Check

- Last active within 6 months: **ACTIVE**
- Last active 6-12 months ago: **DORMANT**
- Last active >12 months ago: **INACTIVE** (flag for review)

#### Spam/Quality Filters

- Minimum 3 posts/content pieces
- Account age >1 month
- Not suspended/banned on platform
- No adult/illegal content
- Not obvious bot/fake account

#### Data Cleaning

- Remove emojis from business names (store separately)
- Normalize phone numbers to +65 format
- Extract email addresses from bio text
- Parse operating hours into structured format
- Detect and extract postal codes
- Clean up hashtags and mentions
- Standardize area names (e.g., "TPY" → "Toa Payoh")

#### Handling Incomplete Data

- Create profile if minimum required fields present
- Flag profile as "needs enrichment"
- Set lower quality score
- Prioritize for manual review or re-crawl

### 4.6 Duplicate Detection & Merging Strategy

**Challenge**: Same HBB may exist across multiple platforms (Instagram, Facebook, Carousell)

#### Detection Methods

**High Confidence Match** (Auto-merge):
- Same username across platforms (e.g., @bakehappysg on IG and FB)
- Identical business name + same contact number
- Same email address or website URL
- Cross-platform links (IG bio links to FB page)

**Medium Confidence Match** (Flag for review):
- Similar business name (Levenshtein distance <3)
- Same contact number, different business name
- Same address/area + same category
- Very similar bio text (>80% similarity via embeddings)

**Low Confidence Match** (Keep separate, suggest merge):
- Same area + same category + similar products
- Visual similarity in logos/branding (image embeddings)

#### Merge Strategy

When duplicates are detected:

1. **Business Name**: Keep most complete or longest version
2. **Categories**: Union of all categories from both profiles
3. **Display Picture**: Keep highest quality/resolution image
4. **Contact Methods**: Merge all contact methods (union)
5. **Source URLs**: Keep array of all source URLs
6. **Social Metrics**: Aggregate/sum metrics across platforms
7. **Confidence Score**: Take maximum confidence score

#### Duplicate Prevention

Before creating new profile:
- Check against existing by business name (fuzzy match)
- Check against existing by contact methods (exact match)
- Check against existing by source URL (exact match)
- Maintain hash index of normalized business names
- Run weekly batch deduplication job

### 4.7 Crawler Operational Requirements

#### Crawling Strategy

**Phase 1: Discovery (Initial Seed)**
- Start with popular hashtags (#sghomebusiness, etc.)
- Top posts from relevant FB groups
- Featured sellers on Carousell
- **Target**: 10,000+ seed profiles discovered

**Phase 2: Expansion (Network Crawl)**
- Follow connections: tagged accounts, mentioned accounts
- Suggested/similar accounts by platform algorithms
- Hashtag co-occurrence (accounts using similar hashtags)
- **Target**: Exponential growth via network effect

**Phase 3: Maintenance (Refresh Strategy)**
- High quality profiles (score >70): Re-crawl every 30 days
- Medium quality (score 40-70): Re-crawl every 60 days
- Low quality (score <40): Re-crawl every 90 days
- Inactive profiles: Re-crawl every 180 days

#### Rate Limiting & Ethics

- Respect platform rate limits (avoid IP bans)
- Use rotating proxies if needed
- Implement exponential backoff on errors
- Honor robots.txt and platform Terms of Service
- Proper user-agent identification
- Crawl during off-peak hours (1am-6am SGT)
- Avoid overwhelming small accounts with requests

#### Error Handling

- Account deleted/suspended: Mark as inactive, keep historical data
- Access denied (private account): Skip, don't create profile
- Incomplete data: Create if minimum threshold met, flag for enrichment
- Platform changes: Alert for manual crawler update
- CAPTCHA encountered: Queue for manual resolution or use solving service

#### Monitoring & Logging

Track and log:
- Crawl success/failure rates by platform
- Data quality metrics over time
- Platform-specific issues (rate limits, blocks)
- Duplicate detection statistics
- Processing time per profile
- AI classification accuracy (sample manual review)

### 4.8 AI/LLM Integration Points

#### Use Cases for AI

**1. HBB Classification** (High Priority)
- **Input**: Profile bio, posts, hashtags, images
- **Output**: isHBB (yes/no), confidence score (0-1), reasoning
- **Model**: GPT-4 or fine-tuned classifier
- **Prompt Example**:

```
You are an expert at identifying home-based businesses in Singapore.

Profile Data:
- Username: @homebakerlove
- Bio: "🍪 Homemade cookies & brownies | Self-collection @ Bedok | DM to order 📱"
- Location: Singapore
- Followers: 450
- Recent Posts: Photos of cookies, cakes in home kitchen setting

Is this a home-based business? Provide confidence score (0-1) and reasoning.

Expected Output Format:
{
  "isHBB": true,
  "confidence": 0.92,
  "reasoning": "Strong indicators: 'homemade' in bio, self-collection mentioned, Bedok location (HDB area), DM-based ordering, home kitchen visible in posts"
}
```

**2. Category Classification** (High Priority)
- **Input**: Business description, products, images
- **Output**: Category labels with confidence scores
- **Model**: Multi-label classifier or GPT-4
- **Prompt Example**:

```
Categorize this business into relevant categories from the taxonomy.

Business: "BakeHappySG"
Description: "Freshly baked brownies, cookies and custom cakes for all occasions. Halal-certified ingredients."
Products: [images of brownies, cookies, birthday cakes]

Available Categories:
- Food & Beverage > Baked Goods
- Food & Beverage > Desserts & Sweets
- Food & Beverage > Dietary Specific
[... full taxonomy ...]

Output Format:
{
  "primary": "Food & Beverage > Baked Goods",
  "secondary": ["Food & Beverage > Desserts & Sweets", "Food & Beverage > Dietary Specific"],
  "tags": ["halal", "custom-cakes", "brownies", "cookies"],
  "confidence": 0.95
}
```

**3. Entity Extraction** (Medium Priority)
- **Input**: Unstructured bio text and posts
- **Output**: Structured contact info, hours, location
- **Model**: GPT-4 or NER (Named Entity Recognition)
- **Prompt Example**:

```
Extract structured information from this bio:

"Homemade healthy meal prep 🥗 | Mon-Fri 10am-6pm | Self-pickup @ Tampines St 45 | WhatsApp: 91234567 | IG: @healthymealssg"

Output Format:
{
  "contactMethods": {
    "whatsapp": "+6591234567",
    "instagram": "@healthymealssg"
  },
  "operatingHours": "Monday-Friday, 10:00 AM - 6:00 PM",
  "address": {
    "area": "Tampines",
    "street": "Tampines Street 45",
    "pickupDetails": "Self-pickup"
  },
  "products": ["meal prep", "healthy meals"]
}
```

**4. Description Generation** (Low Priority)
- **Input**: Messy bio with emojis, hashtags, abbreviated text
- **Output**: Clean, readable business description
- **Model**: GPT-3.5 or GPT-4

**5. Image Classification** (Low Priority)
- **Input**: Product images
- **Output**: Product type/category
- **Model**: CLIP, ResNet, or GPT-4 Vision
- **Use**: Support category classification

**6. Duplicate Detection** (Medium Priority)
- **Input**: Two business profiles
- **Output**: Similarity score, merge recommendation
- **Model**: Sentence embeddings (sentence-transformers)
- **Use**: Semantic similarity of descriptions, visual similarity of logos

## 5. Design & User Experience

### For Customers (Directory Users)

**Discovery Flow**:
1. Customer searches for "home-based bakery in Tampines"
2. System shows profiles discovered by crawler
3. Profile displays: name, category, location, contact, images
4. Customer can contact business directly via WhatsApp/Instagram

**Profile Display**:
- Clear labeling: "Profile created automatically" badge
- Source attribution: "Found on Instagram"
- Last updated timestamp
- Claim button visible for business owners

### For Business Owners (Profile Claiming)

**Claim Flow**:
1. Business owner discovers their auto-generated profile
2. Clicks "Claim This Business"
3. Verifies ownership (via Instagram DM, phone OTP, email)
4. Gains access to edit and enhance profile
5. Can add: operating hours, price list, photo gallery, about section

**Benefits of Claiming**:
- Full control over profile information
- Add custom branding and messaging
- Respond to customer inquiries
- Access to analytics (views, clicks)
- Priority placement in search results
- Verified badge

### Admin Dashboard

**Crawler Monitoring**:
- Real-time crawl status
- Profiles discovered today/this week/all-time
- Quality score distribution
- Category distribution
- Platform breakdown (IG vs FB vs Carousell)

**Manual Review Queue**:
- Low confidence profiles for review
- Potential duplicates for merge decision
- Reported incorrect profiles
- Claimed profiles pending verification

## 6. Technical Considerations

### High-Level Architecture

```
┌─────────────────┐
│  Social Media   │
│   Platforms     │
│ (IG, FB, etc.)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Crawler Layer  │
│  (Agent-based   │
│   browsers)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Extraction &   │
│  Validation     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  AI Processing  │
│  (Classification│
│   Categorization│
│   Extraction)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Deduplication  │
│  & Merging      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Database       │
│  (PostgreSQL)   │
└─────────────────┘
```

### Technology Stack Considerations

**Crawler**:
- Playwright or Puppeteer (headless browsers with JS execution)
- Proxy rotation for IP management
- CAPTCHA solving services (2captcha, anti-captcha)

**Queue System**:
- Redis or RabbitMQ for job queue management
- Background workers for processing

**AI/ML**:
- OpenAI API (GPT-4) for classification and extraction
- OR fine-tuned open-source models (Llama, Mistral)
- Sentence transformers for duplicate detection

**Database**:
- PostgreSQL for structured data
- MongoDB for flexible schema (alternative)
- Elasticsearch for search indexing

**Storage**:
- S3 or Cloudinary for images
- CDN for fast image delivery

### Data Pipeline

```
Discovery → Crawl → Extract → Validate → Classify → 
Deduplicate → Enrich → Store → Index → Serve
```

### Scalability Considerations

- Distributed crawling (multiple workers in parallel)
- Batch processing for AI classification (reduce API costs)
- Caching for frequently accessed data
- Rate limiting per platform
- Horizontal scaling of crawler workers

### Privacy & Legal Considerations

- Only scrape publicly available information
- Respect platform Terms of Service
- Provide opt-out mechanism for business owners
- PDPA compliance (Singapore Personal Data Protection Act)
- Clear attribution of data source
- Allow profile deletion upon request

## 7. Timeline & Milestones

### Phase 1: MVP Crawler (Weeks 1-4)
- Instagram crawler for hashtag-based discovery
- Basic data extraction (name, bio, image, contact)
- Simple HBB classification (keyword-based)
- PostgreSQL storage
- **Target**: 1,000 profiles discovered

### Phase 2: AI Integration (Weeks 5-8)
- GPT-4 integration for classification
- Category auto-classification
- Entity extraction (contacts, hours, location)
- Duplicate detection logic
- **Target**: 3,000 profiles with 70% quality score

### Phase 3: Multi-Platform (Weeks 9-12)
- Facebook crawler
- Carousell crawler
- Cross-platform deduplication
- Profile merging
- **Target**: 5,000+ profiles across all platforms

### Phase 4: Optimization & Refresh (Weeks 13-16)
- Crawler performance optimization
- Refresh strategy implementation
- Quality improvement iteration
- Manual review tooling
- **Target**: 10,000+ profiles, >80% quality

## 8. Open Questions

1. **AI Model Choice**: Should we use GPT-4 API (expensive but accurate) or fine-tune open-source models (cheaper but requires training data)?

2. **Proxy Strategy**: What proxy service should we use for crawler (residential proxies, datacenter proxies, or rotating proxies)?

3. **CAPTCHA Handling**: How aggressively should we handle CAPTCHAs (automated solving vs. manual queue)?

4. **Data Retention**: How long should we keep inactive/deleted profiles in the database (for historical purposes)?

5. **Claiming Verification**: What verification methods are acceptable (Instagram DM verification, phone OTP, email verification)?

6. **Legal Review**: Do we need legal review of scraping practices in Singapore context?

7. **Rate Limits**: What are safe crawling rates for each platform to avoid bans?

8. **Image Storage**: Should we download and store all images locally or just store URLs (risk of broken links)?

9. **Refresh Priority**: How do we prioritize which profiles to refresh first (most popular, most complete, most recent)?

10. **Manual Review**: What percentage of auto-generated profiles should go through manual review before publishing?

---

## Appendix A: Example AI Prompts

### HBB Classification Prompt

```
You are an expert at identifying home-based businesses in Singapore.

Analyze this profile and determine if it's a home-based business:

Username: {username}
Bio: {bio}
Location: {location}
Followers: {followers}
Posts Count: {posts_count}
Recent Hashtags: {hashtags}
Recent Captions: {recent_captions}

Consider these indicators:
- Strong: "home-based", "HBB", "self-collection", "DM to order", postal codes, HDB areas
- Moderate: #sghomebusiness hashtags, mentions of "homemade", "small batch", personal phone
- Exclusions: physical store address, "Pte Ltd", professional studio, commercial hours

Respond in JSON format:
{
  "isHBB": boolean,
  "confidence": float (0-1),
  "reasoning": string,
  "indicators_found": [array of strings]
}
```

### Category Classification Prompt

```
Categorize this business into the appropriate categories:

Business Name: {business_name}
Description: {description}
Products/Services: {products}
Hashtags: {hashtags}

Available Categories (select 1 primary, 0-3 secondary):
1. Food & Beverage
   - Baked Goods
   - Meal Prep
   - Asian Cuisine
   - Western Cuisine
   - Desserts & Sweets
   - Beverages
   - Dietary Specific
2. Handmade & Crafts
   - Jewelry & Accessories
   - Art & Prints
   - Crochet & Knitting
   - Candles & Soaps
   - Stationery & Paper Goods
   - Toys & Dolls
   - Home Decor
3. Services
   - Tuition & Education
   - Beauty & Wellness
   - Pet Services
   - Photography
   - Design & Creative
   - Consulting
   - Alterations & Tailoring
4. Fashion & Apparel
5. Personal Care & Beauty
6. Digital Products

Respond in JSON format:
{
  "primary_category": "string",
  "secondary_categories": [array],
  "tags": [array of relevant tags],
  "confidence": float (0-1)
}
```

### Entity Extraction Prompt

```
Extract structured information from this business bio:

Bio: {bio_text}

Extract:
1. Contact methods (WhatsApp, Telegram, Email, Phone, Instagram handle)
2. Operating hours (if mentioned)
3. Location/Area (Singapore neighborhoods, postal codes)
4. Pickup/Delivery details
5. Products/Services offered
6. Price indicators

Respond in JSON format:
{
  "contactMethods": {
    "whatsapp": "string (+65 format)",
    "telegram": "string",
    "email": "string",
    "phone": "string (+65 format)",
    "instagram": "string (@username)"
  },
  "operatingHours": "string (natural language)",
  "address": {
    "area": "string (neighborhood name)",
    "postalCode": "string (6 digits)",
    "pickupDetails": "string"
  },
  "products": [array of strings],
  "priceRange": "enum ($ | $$ | $$$)"
}
```

---

## Appendix B: Category Taxonomy (Full)

```
Food & Beverage
├── Baked Goods
│   ├── Cakes
│   ├── Cookies & Brownies
│   ├── Pastries & Tarts
│   └── Bread & Buns
├── Meal Prep
│   ├── Healthy Meals
│   ├── Frozen Food
│   └── Catering
├── Asian Cuisine
│   ├── Kueh & Traditional Snacks
│   ├── Dumplings & Dim Sum
│   └── Noodles & Rice Dishes
├── Western Cuisine
│   ├── Pasta
│   ├── Pizza
│   └── Sandwiches & Burgers
├── Desserts & Sweets
│   ├── Puddings & Jellies
│   ├── Ice Cream
│   └── Chocolate & Candy
├── Beverages
│   ├── Coffee
│   ├── Tea
│   └── Fresh Juices
└── Dietary Specific
    ├── Vegan
    ├── Keto
    ├── Halal
    └── Gluten-Free

Handmade & Crafts
├── Jewelry & Accessories
├── Art & Prints
├── Crochet & Knitting
├── Candles & Soaps
├── Stationery & Paper Goods
├── Toys & Dolls
└── Home Decor

Services
├── Tuition & Education
│   ├── Academic Tuition
│   ├── Music Lessons
│   └── Art Classes
├── Beauty & Wellness
│   ├── Nail Art
│   ├── Massage & Therapy
│   └── Makeup Services
├── Pet Services
│   ├── Grooming
│   ├── Pet Sitting
│   └── Training
├── Photography
│   ├── Portraits
│   ├── Events
│   └── Product Photography
├── Design & Creative
│   ├── Graphic Design
│   ├── Web Design
│   └── Content Creation
├── Consulting
│   ├── Business Consulting
│   ├── Financial Consulting
│   └── Career Coaching
└── Alterations & Tailoring

Fashion & Apparel
├── Clothing
│   ├── Women's Fashion
│   ├── Men's Fashion
│   └── Children's Clothing
├── Accessories
│   ├── Bags
│   ├── Shoes
│   └── Hats & Caps
├── Custom/Tailored
└── Vintage/Thrifted

Personal Care & Beauty
├── Skincare
├── Cosmetics
├── Haircare
└── Fragrances

Digital Products
├── Printables & Templates
├── Digital Art
├── E-courses
└── Presets & Filters
```
