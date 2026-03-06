# DataForSEO Integration Guide

This document explains how to use DataForSEO APIs (via MCP or direct API) to enhance meta title/description generation with real data.

## Table of Contents
1. Prerequisites
2. API Endpoints Used
3. Step-by-Step Usage in the Workflow
4. SERP Competitor Analysis Deep Dive
5. Cost Estimation
6. Error Handling & Fallbacks

---

## 1. Prerequisites

### MCP Integration (Preferred)
DataForSEO MCP server should be connected. The relevant modules needed:
- `ONPAGE` — for page content crawling
- `SERP` — for SERP competitor data
- `KEYWORDS_DATA` — for search volume
- `DATAFORSEO_LABS` — for ranked keywords and competitor analysis

If the user has DataForSEO MCP configured, the tools will be available as MCP tools in the conversation.

### Direct API (Fallback)
If using via script/code instead of MCP, credentials are needed:
- Base URL: `https://api.dataforseo.com/v3/`
- Auth: Basic authentication (username:password)
- Rate limit: 2000 calls/minute

---

## 2. API Endpoints Used

### A. On-Page API — Instant Pages
**Purpose:** Crawl a URL and extract its current content, headings, and metadata.
**Endpoint:** `POST /v3/on_page/instant_pages`
**Capacity:** Up to 20 URLs per request, max 5 identical domains per request.

**What we extract:**
- `title` — current page title
- `description` — current meta description
- `h1` — all H1 tags
- `h2` — all H2 tags
- `content.plain_text_word_count` — content length
- `content.automated_readability_index` — readability
- Keyword density data

**Request example:**
```json
[{
  "url": "https://www.vitra.com.tr/c-lavabolar",
  "enable_javascript": true
}]
```

### B. DataForSEO Labs — Ranked Keywords
**Purpose:** Find what keywords a specific URL or domain already ranks for.
**Endpoint:** `POST /v3/dataforseo_labs/google/ranked_keywords/live`

**What we extract:**
- Keywords the URL ranks for
- Position for each keyword
- Search volume per keyword
- Estimated traffic per keyword
- Keyword difficulty

**Request example:**
```json
[{
  "target": "vitra.com.tr/c-lavabolar",
  "language_name": "Turkish",
  "location_code": 2792,
  "limit": 20,
  "order_by": ["keyword_data.keyword_info.search_volume,desc"]
}]
```

**Location codes:**
- Turkey: 2792
- United Kingdom: 2826
- Germany: 2276
- United States: 2840
- France: 2250

### C. SERP API — Live Advanced
**Purpose:** Get the current SERP for a keyword, including all organic results with their titles and descriptions.
**Endpoint:** `POST /v3/serp/google/organic/live/advanced`

**What we extract:**
- Top 10 organic results: URL, title, description, position
- SERP features present (featured snippets, people also ask, etc.)
- AI overview presence (if any)

**Request example:**
```json
[{
  "keyword": "lavabo modelleri",
  "location_code": 2792,
  "language_code": "tr",
  "device": "desktop",
  "depth": 10
}]
```

### D. Keyword Data API — Search Volume
**Purpose:** Get accurate search volume for specific keywords.
**Endpoint:** `POST /v3/keywords_data/google_ads/search_volume/live`

**What we extract:**
- Monthly search volume
- CPC (cost per click)
- Competition level

**Request example:**
```json
[{
  "keywords": ["lavabo modelleri", "lavabo fiyatları", "banyo lavabo"],
  "location_code": 2792,
  "language_code": "tr"
}]
```

---

## 3. Step-by-Step Usage in the Workflow

### Phase 1: Page Understanding (On-Page API)
For the batch of URLs provided by the user:

1. Group URLs by domain (max 5 identical domains per Instant Pages request)
2. Send in batches of 20 URLs
3. For each URL, extract: H1, H2 list, content summary, current title/desc
4. Determine page topic from H1 + content

### Phase 2: Keyword Discovery (DataForSEO Labs)
For each URL:

1. Call Ranked Keywords with the specific URL as target
2. Sort results by search_volume descending
3. Select the top keyword that:
   - Has the highest volume
   - Matches the page's actual content topic (from Phase 1)
   - Is in the "opportunity zone" (position 4-20) if possible
4. This becomes the **Primary Keyword**

For secondary keywords:
1. From the SERP results (Phase 3), identify top 3-5 competitor URLs
2. Call Ranked Keywords for each competitor URL
3. Find keywords that competitors rank for but our URL doesn't
4. Select 1-2 that are semantically related to the primary keyword

### Phase 3: SERP Intelligence (SERP API)
For each primary keyword:

1. Call SERP API Live Advanced
2. Extract top 10 organic results
3. Run the SERP Analysis (see Section 4 below)
4. Generate strategic notes for title/description generation

### Phase 4: Volume Validation (Keyword Data API)
1. Collect all primary + secondary keywords across the batch
2. Send in one bulk request (up to 1000 keywords per call)
3. Attach volume data to output

---

## 4. SERP Competitor Analysis Deep Dive

This is the most valuable analytical step. For each keyword's SERP results:

### 4a. Title Pattern Analysis

**Extract and analyze:**

```
For each of the top 10 titles:
├── Character length
├── Keyword position (first 20 chars? middle? end?)
├── Separator used (|, -, :, none)
├── Brand included? (yes/no)
├── Modifiers present (Fiyatları, Modelleri, Best, Guide, etc.)
├── Year included? (2025, 2026, etc.)
├── Numbers present? (10+, 100, etc.)
└── Superlatives present? (En İyi, Best, Top, etc.)
```

**Aggregate into:**
- Most common modifier (e.g., "Fiyatları appears in 7/10 titles")
- Average title length (e.g., "Avg: 57 chars")
- Keyword position trend (e.g., "8/10 have keyword in first 20 chars")
- Separator distribution (e.g., "6 pipe, 3 dash, 1 none")
- Differentiation gaps (what NO competitor is doing that could work)

### 4b. Description Pattern Analysis

**Extract and analyze:**

```
For each of the top 10 descriptions:
├── Character length
├── CTA type (keşfedin, satın alın, inceleyin, etc.)
├── CTA position (beginning, middle, end)
├── USP mentions (free shipping, guarantee, price, etc.)
├── Numbers/stats present
├── Emotional vs. functional tone
├── Question format used?
└── Brand mentioned in description?
```

**Aggregate into:**
- Dominant CTA pattern
- Average description length
- USP frequency (which selling points competitors emphasize)
- Tone distribution
- Opportunities (USPs or angles no one is using)

### 4c. Strategic Output Format

For each keyword, produce a brief strategy note:

```
Keyword: lavabo modelleri
SERP Profile:
- 8/10 titles use "Fiyatları" modifier → MUST INCLUDE
- 6/10 titles use pipe separator → matches our brand style ✓
- Avg title: 58 chars → we can go up to 65 for more info
- 0/10 mention "2026" → confirms no-year approach ✓
- Gap: no one mentions "ücretsiz kargo" in desc → potential differentiator
- 7/10 descriptions use "keşfedin" → table stakes CTA
- Only 2/10 mention specific count ("100+ model") → number opportunity
Recommendation: Use "Modelleri ve Fiyatları" modifier, add product count in desc if data available
```

This note is internal — used to inform generation, not shown to user unless they ask.

---

## 5. Cost Estimation

### Per-page cost breakdown:
| API Call | Cost per call | Calls per page |
|----------|--------------|----------------|
| Instant Pages | ~$0.001 | 0.05 (20 URLs/call) |
| Ranked Keywords (own URL) | ~$0.01 | 1 |
| SERP Live Advanced | ~$0.002 | 1 |
| Ranked Keywords (competitors) | ~$0.01 | 3-5 |
| Search Volume | ~$0.001 | 0.05 (batch) |

**Estimated cost per page: ~$0.05–0.07**

### Batch estimates:
| Batch Size | Estimated Cost |
|-----------|---------------|
| 10 pages | ~$0.50–0.70 |
| 50 pages | ~$2.50–3.50 |
| 100 pages | ~$5.00–7.00 |
| 200 pages | ~$10.00–14.00 |

These are estimates — actual costs depend on DataForSEO pricing tier and specific API versions used.

---

## 6. Error Handling & Fallbacks

### URL not crawlable
- Instant Pages returns error → Fall back to URL pattern analysis only
- Note in output: "Sayfa içeriği çekilemedi — URL pattern'e göre tahmin"

### No ranking data
- Ranked Keywords returns empty → URL is new or not indexed
- Fall back to user-provided keyword, or derive from URL slug
- Note: "Sıralama verisi bulunamadı — keyword kullanıcı tarafından sağlandı"

### SERP API timeout or error
- Retry once, then skip SERP analysis for that keyword
- Generate title/description using style rules only
- Note: "SERP analizi yapılamadı — stil kurallarına göre üretildi"

### Rate limiting
- If approaching 2000 calls/minute, add delays between batches
- For large batches (200+ URLs), suggest splitting into multiple sessions

### Empty search volume
- Some niche keywords may return 0 volume from Google Ads
- Note as "< 10" (Google doesn't report volumes under 10)
- Don't discard the keyword — it may still be the best match for the page
