# HBB Crawler System - Workflow Diagrams

**Created**: 2026-01-26
**Last Updated**: 2026-01-26
**Related**: [HBB Crawler System PRD](../prds/hbb-crawler-system.md)

This document contains visual workflow diagrams for the HBB Crawler System architecture and data flows.

---

## 1. High-Level System Architecture

```mermaid
graph TB
    subgraph "Data Sources"
        IG[Instagram]
        FB[Facebook]
        CAR[Carousell]
        TT[TikTok]
        GMB[Google My Business]
    end

    subgraph "Crawler Layer"
        DISCO[Discovery Engine]
        SCRAPER[Web Scraper<br/>Playwright/Puppeteer]
        PROXY[Proxy Manager]
        QUEUE[Job Queue<br/>Redis/RabbitMQ]
    end

    subgraph "Processing Layer"
        EXTRACT[Data Extractor]
        VALIDATE[Validator]
        AI[AI Classifier<br/>GPT-4/Local Model]
        DEDUP[Deduplication<br/>Engine]
    end

    subgraph "Storage Layer"
        DB[(PostgreSQL<br/>Database)]
        CACHE[(Redis<br/>Cache)]
        CDN[CDN<br/>Image Storage]
    end

    subgraph "Application Layer"
        API[REST API]
        SEARCH[Search Index<br/>Elasticsearch]
        WEB[Web Frontend]
    end

    IG --> DISCO
    FB --> DISCO
    CAR --> DISCO
    TT --> DISCO
    GMB --> DISCO

    DISCO --> QUEUE
    QUEUE --> SCRAPER
    PROXY --> SCRAPER
    
    SCRAPER --> EXTRACT
    EXTRACT --> VALIDATE
    VALIDATE --> AI
    AI --> DEDUP
    
    DEDUP --> DB
    DB --> CACHE
    SCRAPER --> CDN
    
    DB --> API
    CACHE --> API
    DB --> SEARCH
    API --> WEB
    SEARCH --> WEB
```

---

## 2. Crawler Discovery & Crawl Flow

```mermaid
flowchart TD
    START([Start Crawler]) --> PHASE{Which Phase?}
    
    PHASE -->|Phase 1| SEED[Seed Discovery]
    PHASE -->|Phase 2| NETWORK[Network Crawl]
    PHASE -->|Phase 3| REFRESH[Refresh Crawl]
    
    SEED --> HASHTAGS[Scrape Hashtags<br/>#sghomebusiness, etc.]
    SEED --> GROUPS[Scrape FB Groups]
    SEED --> FEATURED[Scrape Carousell<br/>Featured Sellers]
    
    HASHTAGS --> SEEDQUEUE[Add to Job Queue]
    GROUPS --> SEEDQUEUE
    FEATURED --> SEEDQUEUE
    
    NETWORK --> FOLLOWS[Get Followed/Tagged<br/>Accounts]
    NETWORK --> SIMILAR[Get Similar/Suggested<br/>Accounts]
    NETWORK --> COHASH[Get Co-Hashtag<br/>Users]
    
    FOLLOWS --> NETQUEUE[Add to Job Queue]
    SIMILAR --> NETQUEUE
    COHASH --> NETQUEUE
    
    REFRESH --> GETPROFILES[Get Existing<br/>Profiles from DB]
    GETPROFILES --> PRIORITY{Priority<br/>Level?}
    
    PRIORITY -->|High Quality >70| REFRESH30[Queue for<br/>30-day refresh]
    PRIORITY -->|Medium 40-70| REFRESH60[Queue for<br/>60-day refresh]
    PRIORITY -->|Low <40| REFRESH90[Queue for<br/>90-day refresh]
    PRIORITY -->|Inactive| REFRESH180[Queue for<br/>180-day refresh]
    
    REFRESH30 --> REFQUEUE[Add to Job Queue]
    REFRESH60 --> REFQUEUE
    REFRESH90 --> REFQUEUE
    REFRESH180 --> REFQUEUE
    
    SEEDQUEUE --> CRAWL[Crawl Process]
    NETQUEUE --> CRAWL
    REFQUEUE --> CRAWL
    
    CRAWL --> END([Continue to<br/>Extraction Flow])
```

---

## 3. Data Extraction & Processing Flow

```mermaid
flowchart TD
    START([Job from Queue]) --> FETCHURL[Fetch URL with<br/>Headless Browser]
    
    FETCHURL --> CHECK{Page<br/>Accessible?}
    CHECK -->|No - Deleted| MARKDEL[Mark as Deleted]
    CHECK -->|No - Private| SKIP[Skip Profile]
    CHECK -->|No - Error| RETRY{Retry<br/>Count < 3?}
    CHECK -->|Yes| EXTRACT[Extract Data]
    
    RETRY -->|Yes| BACKOFF[Exponential<br/>Backoff]
    RETRY -->|No| FAILED[Mark as Failed]
    BACKOFF --> FETCHURL
    
    EXTRACT --> FIELDS{All Required<br/>Fields Present?}
    FIELDS -->|No| INCOMPLETE[Flag as<br/>Incomplete]
    FIELDS -->|Yes| VALIDATE[Validate Data]
    
    INCOMPLETE --> LOWPRIORITY[Set Low<br/>Quality Score]
    LOWPRIORITY --> AICLASS
    
    VALIDATE --> CLEAN[Data Cleaning<br/>- Remove emojis<br/>- Normalize phones<br/>- Extract emails]
    
    CLEAN --> AICLASS[AI Classification]
    AICLASS --> HBBCHECK{Is HBB?<br/>Confidence?}
    
    HBBCHECK -->|< 0.5| REJECT[Reject<br/>Not HBB]
    HBBCHECK -->|≥ 0.5| CATEGORY[AI Categorization]
    
    CATEGORY --> ENTITY[Entity Extraction<br/>- Contacts<br/>- Hours<br/>- Location]
    
    ENTITY --> QUALITY[Calculate<br/>Quality Score]
    
    QUALITY --> DUPCHECK[Duplicate Check]
    DUPCHECK --> ISDUP{Duplicate<br/>Found?}
    
    ISDUP -->|No| NEWPROFILE[Create New<br/>Profile]
    ISDUP -->|High Confidence| MERGE[Merge with<br/>Existing]
    ISDUP -->|Medium| FLAGREVIEW[Flag for<br/>Manual Review]
    
    NEWPROFILE --> STORE[Store in Database]
    MERGE --> STORE
    FLAGREVIEW --> STORE
    
    STORE --> INDEX[Index in<br/>Elasticsearch]
    INDEX --> COMPLETE([Complete])
    
    MARKDEL --> COMPLETE
    SKIP --> COMPLETE
    FAILED --> COMPLETE
    REJECT --> COMPLETE
```

---

## 4. AI Classification Workflow

```mermaid
flowchart TD
    START([Profile Data]) --> PREP[Prepare AI Prompt]
    
    PREP --> BIOTEXT{Has Bio<br/>Text?}
    BIOTEXT -->|Yes| ADDBIO[Add Bio to Prompt]
    BIOTEXT -->|No| SKIPBIO[Skip Bio]
    
    ADDBIO --> HASHTAGS{Has<br/>Hashtags?}
    SKIPBIO --> HASHTAGS
    
    HASHTAGS -->|Yes| ADDHASH[Add Hashtags<br/>to Prompt]
    HASHTAGS -->|No| SKIPHASH[Skip Hashtags]
    
    ADDHASH --> LOCATION{Has<br/>Location?}
    SKIPHASH --> LOCATION
    
    LOCATION -->|Yes| ADDLOC[Add Location<br/>to Prompt]
    LOCATION -->|No| SKIPLOC[Skip Location]
    
    ADDLOC --> CALLAI[Call AI API<br/>GPT-4]
    SKIPLOC --> CALLAI
    
    CALLAI --> PARSE[Parse JSON<br/>Response]
    
    PARSE --> VALID{Valid<br/>Response?}
    VALID -->|No| RETRY{Retry<br/>< 3?}
    VALID -->|Yes| EXTRACT[Extract Results]
    
    RETRY -->|Yes| CALLAI
    RETRY -->|No| FALLBACK[Use Rule-Based<br/>Fallback]
    
    EXTRACT --> CONFIDENCE[Get Confidence<br/>Score]
    FALLBACK --> CONFIDENCE
    
    CONFIDENCE --> THRESHOLD{Score<br/>≥ 0.5?}
    
    THRESHOLD -->|No| REJECTHBB[Reject as<br/>Non-HBB]
    THRESHOLD -->|Yes| CATEGORIZE[Categorize<br/>Business]
    
    CATEGORIZE --> CALLCAT[Call AI for<br/>Category Classification]
    CALLCAT --> PARSCAT[Parse Category<br/>Response]
    
    PARSCAT --> ENTITYEXT[Entity Extraction<br/>for Contacts/Hours]
    ENTITYEXT --> CALLENT[Call AI for<br/>Entity Extraction]
    CALLENT --> PARSEENT[Parse Extracted<br/>Entities]
    
    PARSEENT --> STORE[Store AI Results<br/>with Profile]
    REJECTHBB --> STORE
    
    STORE --> END([Return to<br/>Main Flow])
```

---

## 5. Duplicate Detection & Merging Flow

```mermaid
flowchart TD
    START([New Profile]) --> NORMALIZE[Normalize Data<br/>- Lowercase name<br/>- Clean contacts<br/>- Standardize area]
    
    NORMALIZE --> HASH[Generate Hash<br/>from Business Name]
    
    HASH --> INDEXCHECK[Check Hash Index<br/>in Database]
    
    INDEXCHECK --> FOUND{Exact Hash<br/>Match?}
    
    FOUND -->|Yes| HIGHCONF[High Confidence<br/>Duplicate]
    FOUND -->|No| FUZZY[Fuzzy Name Match<br/>Levenshtein Distance]
    
    FUZZY --> FUZZYMATCH{Distance<br/>< 3?}
    FUZZYMATCH -->|Yes| CHECKCONTACT[Check Contact<br/>Methods]
    FUZZYMATCH -->|No| CHECKALT[Check Alternative<br/>Signals]
    
    CHECKCONTACT --> CONTACT{Same Phone<br/>or Email?}
    CONTACT -->|Yes| HIGHCONF
    CONTACT -->|No| MEDCONF[Medium Confidence<br/>Duplicate]
    
    CHECKALT --> SOURCECHECK{Same<br/>Source URL?}
    SOURCECHECK -->|Yes| EXACTDUP[Exact Duplicate]
    SOURCECHECK -->|No| AREACHECK{Same Area +<br/>Category?}
    
    AREACHECK -->|Yes| BIOCHECK[Compare Bios<br/>Embedding Similarity]
    AREACHECK -->|No| NODUP[No Duplicate]
    
    BIOCHECK --> SIMILAR{Similarity<br/>> 0.8?}
    SIMILAR -->|Yes| MEDCONF
    SIMILAR -->|No| LOWCONF[Low Confidence<br/>Possible Duplicate]
    
    EXACTDUP --> UPDATETIME[Update Existing<br/>Timestamp Only]
    HIGHCONF --> AUTOMERGE{Auto-Merge<br/>Enabled?}
    
    AUTOMERGE -->|Yes| MERGE[Merge Profiles]
    AUTOMERGE -->|No| FLAGHIGH[Flag for<br/>Manual Review]
    
    MEDCONF --> FLAGMED[Flag for<br/>Manual Review]
    LOWCONF --> SUGGEST[Suggest Merge<br/>to Admin]
    NODUP --> CREATE[Create New<br/>Profile]
    
    MERGE --> MERGELOGIC[Merge Logic:<br/>- Union contacts<br/>- Best quality image<br/>- Union categories<br/>- Keep all sources]
    
    MERGELOGIC --> UPDATEDB[Update Existing<br/>Profile in DB]
    CREATE --> INSERTDB[Insert New<br/>Profile in DB]
    
    UPDATEDB --> COMPLETE([Complete])
    INSERTDB --> COMPLETE
    UPDATETIME --> COMPLETE
    FLAGHIGH --> COMPLETE
    FLAGMED --> COMPLETE
    SUGGEST --> COMPLETE
```

---

## 6. Profile Claiming Workflow

```mermaid
flowchart TD
    START([User Finds<br/>Their Profile]) --> CLICKCLAIM[Click "Claim<br/>This Business"]
    
    CLICKCLAIM --> METHOD{Verification<br/>Method?}
    
    METHOD -->|Instagram| IGVERIFY[Instagram<br/>Verification]
    METHOD -->|Phone| PHONEVERIFY[Phone OTP<br/>Verification]
    METHOD -->|Email| EMAILVERIFY[Email<br/>Verification]
    
    IGVERIFY --> IGDM[Send DM to<br/>Instagram Account<br/>with Code]
    IGDM --> IGWAIT[Wait for User<br/>to Enter Code]
    IGWAIT --> IGCHECK{Code<br/>Correct?}
    IGCHECK -->|No| IGFAIL[Verification Failed]
    IGCHECK -->|Yes| VERIFIED[Profile Verified]
    
    PHONEVERIFY --> SENDOTP[Send OTP to<br/>Phone Number]
    SENDOTP --> PHONEWAIT[Wait for User<br/>to Enter OTP]
    PHONEWAIT --> PHONECHECK{OTP<br/>Correct?}
    PHONECHECK -->|No| PHONEFAIL[Verification Failed]
    PHONECHECK -->|Yes| VERIFIED
    
    EMAILVERIFY --> SENDEMAIL[Send Verification<br/>Link to Email]
    SENDEMAIL --> EMAILWAIT[Wait for User<br/>to Click Link]
    EMAILWAIT --> EMAILCHECK{Link<br/>Clicked?}
    EMAILCHECK -->|No| EMAILFAIL[Verification Failed]
    EMAILCHECK -->|Yes| VERIFIED
    
    VERIFIED --> CREATEACCT[Create User<br/>Account]
    CREATEACCT --> LINKPROFILE[Link Account<br/>to Profile]
    LINKPROFILE --> GRANTACCESS[Grant Edit<br/>Permissions]
    
    GRANTACCESS --> ADDBADGE[Add "Verified"<br/>Badge to Profile]
    ADDBADGE --> NOTIFY[Notify User<br/>of Success]
    
    NOTIFY --> EDITFORM[Show Enhanced<br/>Edit Form]
    EDITFORM --> USERUPDATE[User Updates<br/>Profile Info]
    USERUPDATE --> SAVEDB[Save to Database]
    SAVEDB --> REINDEX[Reindex in<br/>Search Engine]
    
    REINDEX --> COMPLETE([Profile Claimed<br/>& Updated])
    
    IGFAIL --> RETRY{Retry<br/>Allowed?}
    PHONEFAIL --> RETRY
    EMAILFAIL --> RETRY
    
    RETRY -->|Yes| METHOD
    RETRY -->|No| FAILED([Claiming Failed])
```

---

## 7. Refresh Strategy Flow

```mermaid
flowchart TD
    START([Daily Refresh Job]) --> GETALL[Get All Profiles<br/>from Database]
    
    GETALL --> FOREACH{For Each<br/>Profile}
    
    FOREACH --> CHECKSTATUS{Activity<br/>Status?}
    
    CHECKSTATUS -->|Active| CHECKSCORE{Quality<br/>Score?}
    CHECKSTATUS -->|Dormant| DUEDATE1{Last Update<br/>> 60 days?}
    CHECKSTATUS -->|Inactive| DUEDATE2{Last Update<br/>> 180 days?}
    
    CHECKSCORE -->|> 70| DUEDATE3{Last Update<br/>> 30 days?}
    CHECKSCORE -->|40-70| DUEDATE4{Last Update<br/>> 60 days?}
    CHECKSCORE -->|< 40| DUEDATE5{Last Update<br/>> 90 days?}
    
    DUEDATE1 -->|Yes| QUEUEREF[Queue for<br/>Refresh]
    DUEDATE1 -->|No| SKIP[Skip]
    
    DUEDATE2 -->|Yes| QUEUEREF
    DUEDATE2 -->|No| SKIP
    
    DUEDATE3 -->|Yes| QUEUEREF
    DUEDATE3 -->|No| SKIP
    
    DUEDATE4 -->|Yes| QUEUEREF
    DUEDATE4 -->|No| SKIP
    
    DUEDATE5 -->|Yes| QUEUEREF
    DUEDATE5 -->|No| SKIP
    
    QUEUEREF --> ADDQUEUE[Add to Redis<br/>Job Queue]
    SKIP --> NEXT{More<br/>Profiles?}
    ADDQUEUE --> NEXT
    
    NEXT -->|Yes| FOREACH
    NEXT -->|No| PROCESS[Workers Process<br/>Refresh Jobs]
    
    PROCESS --> CRAWLAGAIN[Crawl Profile<br/>Again]
    CRAWLAGAIN --> COMPARE[Compare with<br/>Existing Data]
    
    COMPARE --> CHANGES{Data<br/>Changed?}
    
    CHANGES -->|Yes| UPDATE[Update Profile]
    CHANGES -->|No| TIMESTAMP[Update Last<br/>Checked Time Only]
    
    UPDATE --> CHECKSTILL{Still<br/>Active?}
    
    CHECKSTILL -->|Yes| UPDATEACTIVE[Update Status:<br/>Active]
    CHECKSTILL -->|No| UPDATEINACTIVE[Update Status:<br/>Inactive/Dormant]
    
    UPDATEACTIVE --> SAVECHANGES[Save to Database]
    UPDATEINACTIVE --> SAVECHANGES
    TIMESTAMP --> SAVECHANGES
    
    SAVECHANGES --> COMPLETE([Refresh Complete])
```

---

## 8. Error Handling & Retry Flow

```mermaid
flowchart TD
    START([Crawl Job]) --> TRY[Try to Fetch URL]
    
    TRY --> ERROR{Error<br/>Type?}
    
    ERROR -->|None| SUCCESS[Extract Data]
    ERROR -->|Rate Limited| RATELIMIT[Rate Limit<br/>Detected]
    ERROR -->|CAPTCHA| CAPTCHADET[CAPTCHA<br/>Detected]
    ERROR -->|Network| NETWORK[Network Error]
    ERROR -->|Timeout| TIMEOUT[Timeout Error]
    ERROR -->|404| NOTFOUND[Page Not Found]
    ERROR -->|403| FORBIDDEN[Access Forbidden]
    
    RATELIMIT --> BACKOFF1[Exponential Backoff<br/>2^n seconds]
    BACKOFF1 --> RETRYCHECK1{Retry Count<br/>< 5?}
    RETRYCHECK1 -->|Yes| INCRETRY1[Increment Retry]
    RETRYCHECK1 -->|No| FAILED[Mark as Failed]
    INCRETRY1 --> WAIT1[Wait Backoff<br/>Duration]
    WAIT1 --> TRY
    
    CAPTCHADET --> CAPTCHATYPE{CAPTCHA<br/>Solving Enabled?}
    CAPTCHATYPE -->|Yes| SOLVECAPTCHA[Send to Solving<br/>Service]
    CAPTCHATYPE -->|No| QUEUEMANUAL[Queue for Manual<br/>Resolution]
    SOLVECAPTCHA --> SOLVED{Solved?}
    SOLVED -->|Yes| TRY
    SOLVED -->|No| QUEUEMANUAL
    
    NETWORK --> RETRYCHECK2{Retry Count<br/>< 3?}
    RETRYCHECK2 -->|Yes| INCRETRY2[Increment Retry]
    RETRYCHECK2 -->|No| FAILED
    INCRETRY2 --> WAIT2[Wait 30s]
    WAIT2 --> TRY
    
    TIMEOUT --> RETRYCHECK3{Retry Count<br/>< 3?}
    RETRYCHECK3 -->|Yes| INCRETRY3[Increment Retry]
    RETRYCHECK3 -->|No| FAILED
    INCRETRY3 --> WAIT3[Wait 60s]
    WAIT3 --> TRY
    
    NOTFOUND --> CHECKEXISTS{Profile Exists<br/>in DB?}
    CHECKEXISTS -->|Yes| MARKDELETED[Mark as Deleted/<br/>Inactive]
    CHECKEXISTS -->|No| SKIP[Skip Profile]
    
    FORBIDDEN --> CHECKPRIVATE{Private<br/>Account?}
    CHECKPRIVATE -->|Yes| MARKPRIVATE[Mark as Private]
    CHECKPRIVATE -->|No| CHECKBAN{IP Banned?}
    CHECKBAN -->|Yes| ROTATEPROXY[Rotate Proxy]
    CHECKBAN -->|No| FAILED
    ROTATEPROXY --> TRY
    
    SUCCESS --> CONTINUE([Continue to<br/>Processing])
    FAILED --> ALERT[Alert Admin]
    SKIP --> COMPLETE([Complete])
    MARKDELETED --> COMPLETE
    MARKPRIVATE --> COMPLETE
    QUEUEMANUAL --> COMPLETE
    ALERT --> COMPLETE
```

---

## 9. Data Quality Scoring Flow

```mermaid
flowchart TD
    START([Profile Data]) --> INITSCORE[Initialize Score = 0]
    
    INITSCORE --> CHECKDP{Has Display<br/>Picture?}
    CHECKDP -->|Yes| ADDDP[Score + 20]
    CHECKDP -->|No| SKIPDP[Skip]
    
    ADDDP --> CHECKDESC{Has<br/>Description?}
    SKIPDP --> CHECKDESC
    
    CHECKDESC -->|Yes| ADDDESC[Score + 15]
    CHECKDESC -->|No| SKIPDESC[Skip]
    
    ADDDESC --> CHECKCONTACT{Has 2+<br/>Contacts?}
    SKIPDESC --> CHECKCONTACT
    
    CHECKCONTACT -->|Yes| ADDCONTACT[Score + 15]
    CHECKCONTACT -->|No| SKIPCONTACT[Skip]
    
    ADDCONTACT --> CHECKAREA{Has Area/<br/>Location?}
    SKIPCONTACT --> CHECKAREA
    
    CHECKAREA -->|Yes| ADDAREA[Score + 10]
    CHECKAREA -->|No| SKIPAREA[Skip]
    
    ADDAREA --> CHECKHOURS{Has Operating<br/>Hours?}
    SKIPAREA --> CHECKHOURS
    
    CHECKHOURS -->|Yes| ADDHOURS[Score + 10]
    CHECKHOURS -->|No| SKIPHOURS[Skip]
    
    ADDHOURS --> CHECKIMAGES{Has 3+<br/>Images?}
    SKIPHOURS --> CHECKIMAGES
    
    CHECKIMAGES -->|Yes| ADDIMAGES[Score + 10]
    CHECKIMAGES -->|No| SKIPIMAGES[Skip]
    
    ADDIMAGES --> CHECKPRICE{Has Price<br/>Range?}
    SKIPIMAGES --> CHECKPRICE
    
    CHECKPRICE -->|Yes| ADDPRICE[Score + 10]
    CHECKPRICE -->|No| SKIPPRICE[Skip]
    
    ADDPRICE --> CHECKSOCIAL{Has Social<br/>Metrics?}
    SKIPPRICE --> CHECKSOCIAL
    
    CHECKSOCIAL -->|Yes| ADDSOCIAL[Score + 5]
    CHECKSOCIAL -->|No| SKIPSOCIAL[Skip]
    
    ADDSOCIAL --> CHECKREVIEWS{Has<br/>Reviews?}
    SKIPSOCIAL --> CHECKREVIEWS
    
    CHECKREVIEWS -->|Yes| ADDREVIEWS[Score + 5]
    CHECKREVIEWS -->|No| SKIPREVIEWS[Skip]
    
    ADDREVIEWS --> FINALSCORE[Final Quality<br/>Score: 0-100]
    SKIPREVIEWS --> FINALSCORE
    
    FINALSCORE --> CLASSIFY{Score<br/>Range?}
    
    CLASSIFY -->|> 70| HIGH[High Quality<br/>Profile]
    CLASSIFY -->|40-70| MEDIUM[Medium Quality<br/>Profile]
    CLASSIFY -->|< 40| LOW[Low Quality<br/>Profile]
    
    HIGH --> SAVESTATUS[Save Quality<br/>Status to DB]
    MEDIUM --> SAVESTATUS
    LOW --> SAVESTATUS
    
    SAVESTATUS --> END([Complete])
```

---

## 10. Complete End-to-End Flow

```mermaid
flowchart LR
    subgraph "Discovery"
        A[Hashtag Search]
        B[FB Group Scrape]
        C[Network Crawl]
    end
    
    subgraph "Queue"
        Q[Redis Job Queue]
    end
    
    subgraph "Crawling"
        D[Fetch with Browser]
        E[Extract Data]
        F[Validate Fields]
    end
    
    subgraph "AI Processing"
        G[HBB Classification]
        H[Category Classification]
        I[Entity Extraction]
    end
    
    subgraph "Quality Control"
        J[Quality Scoring]
        K[Duplicate Check]
        L[Data Cleaning]
    end
    
    subgraph "Storage"
        M[(Database)]
        N[Search Index]
        O[CDN Images]
    end
    
    subgraph "Application"
        P[API]
        R[Web App]
        S[User Claims Profile]
    end
    
    A --> Q
    B --> Q
    C --> Q
    
    Q --> D
    D --> E
    E --> F
    
    F --> G
    G --> H
    H --> I
    
    I --> J
    J --> K
    K --> L
    
    L --> M
    M --> N
    E --> O
    
    M --> P
    N --> P
    P --> R
    R --> S
    S --> M
```

---

## Notes on Diagram Usage

### Mermaid Rendering
These diagrams use Mermaid syntax and will render automatically on:
- GitHub (native support)
- GitLab (native support)
- Visual Studio Code (with Mermaid extension)
- Many documentation platforms

### Diagram Purposes

1. **High-Level Architecture**: System overview for stakeholders
2. **Discovery & Crawl Flow**: For understanding crawler phases
3. **Extraction & Processing**: For developers implementing the pipeline
4. **AI Classification**: For AI integration and prompt engineering
5. **Duplicate Detection**: For data quality team
6. **Profile Claiming**: For product/UX team
7. **Refresh Strategy**: For operations team
8. **Error Handling**: For reliability engineering
9. **Quality Scoring**: For data quality metrics
10. **End-to-End Flow**: Quick reference for entire system

### Updating Diagrams

When system design changes:
1. Update relevant diagram(s)
2. Update "Last Updated" date at top
3. Add note in commit message about diagram changes
4. Ensure consistency across related diagrams

---

**Last Updated**: 2026-01-26
**Maintained By**: Engineering Team
**Related Documents**:
- [HBB Crawler System PRD](../prds/hbb-crawler-system.md)
- [HBB Directory Landscape Research](../research/hbb-directory-landscape-research.md)
