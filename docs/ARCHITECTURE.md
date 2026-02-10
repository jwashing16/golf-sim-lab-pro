# Golf Sim Lab Pro - Data Architecture

## Philosophy
- **Radical transparency**: We use AI to aggregate and analyze, never to fabricate
- **Signal over noise**: Extract patterns from many sources, not opinions from one
- **Living content**: Reviews update as new data arrives, with clear timestamps
- **Ethical by design**: Attribution, corrections, and honest limitations

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    DATA COLLECTION LAYER                         │
├─────────────────────────────────────────────────────────────────┤
│  Reddit API (PRAW)          │  YouTube API (transcripts)         │
│  • r/golf, r/golfsimulator  │  • Review channels                  │
│  • Rate: 30/min (OAuth)     │  • Rate: 10k quota/day              │
│  • Store: Raw posts + metadata                                   │
├─────────────────────────────────────────────────────────────────┤
│  GolfWRX Scraper (respectful)  │  Amazon Reviews API              │
│  • forums.golfwrx.com          │  • Affiliate API (limited)        │
│  • Rate: 1 req/5 sec           │  • Verified purchases only        │
│  • robots.txt compliant        │  • 1-5 star + text                │
├─────────────────────────────────────────────────────────────────┤
│  Independent Tests (manual entry)                               │
│  • MyGolfSpy data              │  • GSPro community tests         │
│  • PGA professional reviews    │  • Store: URL + summary          │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PROCESSING LAYER                              │
├─────────────────────────────────────────────────────────────────┤
│  RAW DATA STORE (SQLite)                                        │
│  ├── sources: url, type, date, raw_text, checksum              │
│  ├── sentiment: score (-1 to 1), confidence, model_version     │
│  └── extraction: claims, complaints, praise (with quotes)      │
├─────────────────────────────────────────────────────────────────┤
│  PATTERN EXTRACTION (AI-assisted)                               │
│  • Not: "This is the best LM"                                  │
│  • Yes: "23% of Reddit threads mention RCT balls improve R10"  │
│  • Yes: "Common complaint: Mevo+ battery dies at 2.5h vs 3h"   │
│  • Track: Source count, date range, confidence level           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CONTENT GENERATION LAYER                      │
├─────────────────────────────────────────────────────────────────┤
│  REVIEW TEMPLATES (Jinja2)                                      │
│  ├── Product header: Specs from OEM                             │
│  ├── What users say: Aggregated patterns                        │
│  ├── Common complaints: With source attribution                 │
│  ├── Best for: Use case clusters                                │
│  └── Data freshness: Last updated timestamp                     │
├─────────────────────────────────────────────────────────────────┤
│  HUMAN REVIEW WORKFLOW                                          │
│  1. AI generates draft from data                                │
│  2. Jeffrey reviews for accuracy (15 min/product)              │
│  3. Publish or flag for more research                           │
│  4. Corrections tracked in git history                          │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PUBLICATION LAYER                             │
├─────────────────────────────────────────────────────────────────┤
│  SITE GENERATION                                                │
│  • Static HTML (current) → Future: Incremental updates         │
│  • Git-based versioning                                         │
│  • Automated deploy on data refresh                             │
├─────────────────────────────────────────────────────────────────┤
│  TRANSPARENCY PAGE                                              │
│  • "How we research" - explain the AI+human process             │
│  • Data sources list with sample links                          │
│  • Limitations: "We can't test durability (6mo min)"           │
│  • Correction log: Date + what changed + why                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Data Schema

### Source Table
```sql
CREATE TABLE sources (
    id INTEGER PRIMARY KEY,
    url TEXT UNIQUE NOT NULL,
    source_type TEXT,  -- 'reddit', 'youtube', 'golfwrx', 'amazon'
    product_slug TEXT, -- 'garmin-r10', 'mevo-plus'
    date_collected TEXT,
    raw_content TEXT,
    sentiment_score REAL,  -- -1.0 to 1.0
    confidence REAL,       -- 0.0 to 1.0
    checksum TEXT,         -- detect changes
    status TEXT            -- 'active', 'deleted', 'paywalled'
);
```

### Pattern Table
```sql
CREATE TABLE patterns (
    id INTEGER PRIMARY KEY,
    product_slug TEXT,
    pattern_type TEXT,  -- 'complaint', 'praise', 'use_case', 'comparison'
    statement TEXT,     -- "Users report 2-3 hour battery life"
    source_count INTEGER,
    first_seen TEXT,
    last_seen TEXT,
    confidence TEXT,    -- 'high' (10+ sources), 'medium' (5-9), 'low' (2-4)
    sample_quotes TEXT  -- JSON array of attributed quotes
);
```

---

## Collection Schedule

| Source | Frequency | Rate Limit | Priority |
|--------|-----------|------------|----------|
| Reddit | Weekly | 30 req/min | High |
| YouTube | Monthly (transcripts) | 10k/day | Medium |
| GolfWRX | Bi-weekly | 1 req/5s | Medium |
| Amazon | Weekly | Affiliate quota | High |
| Manual tests | As available | N/A | Low |

---

## Example Output (JSON)

```json
{
  "product": "garmin-r10",
  "last_updated": "2026-02-10",
  "patterns": {
    "praise": [
      {
        "statement": "Accuracy improved significantly after 2024 software update",
        "sources": 23,
        "confidence": "high",
        "sample": "r/golf user (Sep 2024): 'R10 was all around in a really good spot and close to Trackman'"
      }
    ],
    "complaints": [
      {
        "statement": "Spin calculation less accurate than measured systems",
        "sources": 15,
        "confidence": "high",
        "sample": "GolfWRX (Jan 2025): 'Spin is about 15% off compared to GC3 on driver'"
      }
    ],
    "use_cases": [
      {
        "statement": "Best for garage simulators with 9ft+ ceilings",
        "sources": 31,
        "confidence": "high"
      }
    ]
  },
  "limitations": [
    "We have not conducted 6-month durability testing",
    "Indoor accuracy data limited to 8-10ft ceiling heights"
  ]
}
```

---

## MVP Implementation (This Week)

1. **SQLite schema** - Create tables
2. **Reddit collector** - PRAW script, 5 products, historical data
3. **Pattern extractor** - OpenAI API to extract claims from raw text
4. **Review generator** - Template + patterns → HTML
5. **Transparency page** - "How Golf Sim Lab Pro works"

**Est. time**: 4-6 hours of dev
**Ongoing**: 30 min/week for data refresh

---

## Honest AI Disclosure (for site)

> "We use AI to read thousands of user reviews, forum threads, and test videos. 
> The AI extracts patterns like '23% of users mention X' but never invents facts. 
> A human reviews every article before publication. 
> We disclose when we haven't tested a product ourselves."

---

## Differentiation from Generic Review Sites

| Generic Site | Golf Sim Lab Pro |
|--------------|------------------|
| One person tests, writes opinion | Hundreds of users aggregated |
| Static review (published, forgotten) | Living content (updates monthly) |
| "Best launch monitor" (subjective) | "Best for garage sims under $600" (pattern-based) |
| Hides methodology | Full transparency page |
| Affiliate links hidden | Clear disclosure, data-first |

---

_This is the foundation. Build this right, scale to any product category._
