---
name: seo-meta-writer
description: >
  Generate SEO-optimized meta titles and descriptions for multiple web pages at scale.
  Use this skill whenever the user wants to write, rewrite, or optimize meta titles and/or
  meta descriptions for one or more URLs. Trigger on phrases like "meta title", "meta description",
  "title tag", "title ve description yaz", "meta tag hazırla", "SERP snippet optimize et",
  "sayfa başlıkları", or any request involving writing page titles and descriptions for SEO purposes.
  Also trigger when the user uploads a spreadsheet or CSV containing URLs and asks for SEO content,
  or when they paste a list of URLs and ask for title/description recommendations. This skill
  covers Turkish, English, German and other languages — it auto-detects or asks.
  Works with or without DataForSEO MCP integration for enhanced results.
---

# SEO Meta Writer

Generate high-quality, data-informed meta titles and descriptions for multiple pages.

## Execution Mode: AUTONOMOUS

This skill runs end-to-end without asking for permission at intermediate steps. Once the user provides URLs and brand info, execute the entire workflow silently and deliver the final output.

**Do NOT ask for confirmation before:**
- Making DataForSEO API calls (On-Page, SERP, Keyword Data, Labs)
- Crawling pages or fetching SERP data
- Selecting primary/secondary keywords
- Choosing modifiers based on volume or SERP data
- Generating titles and descriptions
- Creating Excel files
- Running cannibalization checks

**Only pause and ask the user if:**
- Brand name is missing and cannot be inferred
- Language is ambiguous (e.g., mixed TLDs, unclear URL patterns)
- A critical error persists after all retry/alternative attempts have been exhausted
- The user explicitly asks for a review before final output

**Error behavior — DO NOT SKIP, FIX:**
- If a DataForSEO call fails, DO NOT silently skip it. Stop, diagnose the error, try alternatives (different location code, different endpoint, retry), and only report to the user if all alternatives fail.
- Common issue: location-based errors. If location_code 2792 (Turkey) fails, try location_name "Turkey" instead, or try without location parameter, or try a different location_code format.
- If an API returns unexpected structure, try parsing differently before giving up.
- Deliver the complete result set in one response, not piece by piece.
- When the user provides enough context (URLs + brand), skip Step 1 questions entirely and go straight to work.

**General behavior:**
- If DataForSEO MCP tools are available, use them immediately — do not ask "Shall I use DataForSEO?" or "Do you want me to analyze the SERP?"
- Batch all API calls efficiently — do not narrate each call or ask between them

## Before you start

Read these reference files based on what you need:

- **Always read:** `references/style-rules.md` — Core SEO rules, character limits, formulas, and anti-patterns
- **If the brand is in the portfolio:** `references/brand-profiles.md` — Brand-specific separator, tone, CTA, and modifier preferences
- **If DataForSEO MCP is available:** `references/dataforseo-integration.md` — How to use DataForSEO for page analysis, keyword discovery, and SERP competitor intelligence

## Workflow Overview

```
Step 1: Collect configuration from user
Step 2: Ingest URL + keyword data
        ├─ If full-list provided: separate active vs reference pages
        └─ Build reserved keyword map from reference pages
Step 3: [If DataForSEO] Page content analysis + keyword discovery
        └─ If page topic unclear: run fallback chain (URL → Schema → H1 → Title → Content)
Step 4: [If DataForSEO] SERP competitor title/description analysis
Step 5: Generate titles and descriptions
Step 6: Cannibalization check (active batch + full-site reference)
Step 7: Output (chat table + optional Excel)
```

---

## Step 1: Collect Configuration

Gather essentials from context — only ask what's truly missing:

1. **Brand name** (required) — will be appended to titles with `|` separator
2. **Language** — auto-detect from URL TLD or path (`.com.tr` → Turkish, `/en/` → English, `/de/` → German). Only ask if ambiguous.
3. **Sector/vertical** — infer from brand profile or URL patterns. Only ask if unknown brand.
4. **DataForSEO** — if MCP tools are available, use them silently. Do not ask.

If the brand is in `references/brand-profiles.md`, load its defaults and proceed — do not confirm with the user unless something seems off.

**Shortcut:** If the user provides URLs + brand name in their first message, that's enough. Skip all questions and start working immediately.

## Step 2: Ingest Data

Accept data in two ways:

### Option A: File upload (CSV/Excel)
Expected columns (flexible naming):
- `URL` (required)
- `Keyword` or `Hedef Keyword` or `Target Keyword` (optional if DataForSEO available)
- `Page Type` or `Sayfa Tipi` (optional — will be auto-detected)
- `Active` or `Aktif` or `İşlem` (optional — marks which pages to generate title/desc for)
- Any existing title/description columns are noted but not used as basis for new ones

### Option B: Chat paste
User pastes URL list (one per line) with optional keyword next to each URL.
Parse flexibly — accept tab-separated, comma-separated, or even natural language lists.

### Full-List + Subset Mode (Cannibalization Prevention)

User can provide a **complete site category list** but request title/description generation for only a subset. This is critical for cannibalization prevention — the skill needs to know the full keyword landscape of the site.

**How it works:**

1. User provides ALL URLs (e.g., full category tree of 200 pages)
2. User marks which ones to process — in one of these ways:
   - A column: `Active = yes/no` or `İşlem = evet/hayır`
   - Explicitly says: "Sadece şu 15 URL için yap, diğerleri referans"
   - Marks with `*`, `>`, or any indicator in the list
   - Says: "İlk 20'si için title/desc yaz, kalanlar cannibalization referansı"

3. Skill separates the list into two groups:
   - **Active pages** → Full workflow: page analysis, keyword discovery, SERP analysis, title/desc generation
   - **Reference pages** → Lightweight analysis only: determine their likely primary keyword (from URL slug, page type, or quick data check) to build the "reserved keyword map"

4. The **reserved keyword map** feeds into Step 6 (Cannibalization Check):
   - Active pages cannot use keywords that reference pages already own
   - If an active page's best keyword candidate conflicts with a reference page, pick the next best alternative

**Example:**
```
User provides 50 category URLs. Says: "İlk 10'u için title/desc yaz."

Skill:
├─ 10 active pages → full analysis + title/desc generation
├─ 40 reference pages → extract likely keywords from URL slugs
├─ Reserved keyword map: {"lavabo": /c-lavabolar, "küvet": /c-kuvetler, ...}
└─ Active page /c-kose-kuvet gets "Köşe Küvet" not "Küvet" (because "Küvet" is reserved by /c-kuvetler)
```

**If user provides only active pages (no reference list):**
Proceed normally — cannibalization check runs only within the active batch.

### Page Type Auto-Detection
Infer page type from URL pattern:

| Pattern | Type | Turkish Label |
|---------|------|--------------|
| `/c-`, `/kategori/`, `/collection/`, `/magaza/`, `/pasaj/` | PLP | Kategori |
| `/p-`, `/urun/`, `/product/`, SKU/model codes in URL | PDP | Ürün |
| `/blog/`, `/makale/`, `/article/`, `/news/`, `/haber/` | Blog | Blog/İçerik |
| Root `/`, `/tr`, `/en`, `/de` (alone) | Homepage | Ana Sayfa |
| `/tr/odalar`, `/en/dining`, `/spa`, `/hakkimizda`, `/about` | Service/Info | Hizmet/Bilgi |
| Brand-filtered URL (brand name in path, e.g., `-altus`, `-samsung`) | Brand PLP | Marka Filtreli |
| Color/material/attribute in path (e.g., `-siyah-`, `-ahsap-`) | Ghost/Filter PLP | Nitelik Filtreli |

## Step 3: Page & Keyword Analysis (DataForSEO Enhanced)

> Skip this step if DataForSEO MCP is not available. Proceed to Step 5 with user-provided keywords.

Read `references/dataforseo-integration.md` for API details. In summary:

### 3a. Page Content Crawl
Use **On-Page API → Instant Pages** to fetch each URL's:
- Current H1, H2 headings
- Body text content and keyword density
- Current title and description (for reference only, not as basis)

This tells us what the page is actually about.

### 3b. Keyword Profile Discovery
Use **DataForSEO Labs → Ranked Keywords** for each URL to find:
- Keywords the page already ranks for (with positions and traffic)
- The strongest keyword candidate based on volume × relevance

### 3c. Page Understanding Fallback Chain

If after 3a and 3b the page's topic or primary keyword is still unclear (e.g., On-Page API returned thin data, Ranked Keywords returned empty, URL slug is ambiguous), apply this fallback chain. Each step is ordered from **least token-consuming to most** — stop as soon as you get a clear answer:

```
Step 1: URL Slug Analysis (0 tokens, 0 API calls)
│  Parse the URL path: /c-likit-kapatici → "likit kapatıcı"
│  Often sufficient for well-structured sites.
│  → Clear? STOP. Use this as primary keyword candidate.
│
Step 2: Breadcrumb Schema Check (minimal tokens)
│  Fetch the page and look for BreadcrumbList schema (JSON-LD or microdata).
│  Breadcrumbs reveal exact category hierarchy: Home > Makyaj > Kapatıcı > Likit Kapatıcı
│  This also tells you the parent category (for hierarchy awareness).
│  → Found? STOP. Use last breadcrumb item as primary keyword.
│
Step 3: Product/Category Schema Check (minimal tokens)
│  Look for Product, ProductGroup, or CollectionPage schema on the page.
│  Schema contains: name, category, description fields.
│  → Found? STOP. Use schema name/category as keyword.
│
Step 4: H1 Tag Check (minimal tokens)
│  Extract just the H1 from the page.
│  H1 is usually the most direct indicator of page topic.
│  → Clear and specific? STOP. Use H1 as keyword basis.
│
Step 5: Existing Title Tag (minimal tokens)
│  Read the current <title> tag.
│  Even if we're rewriting it, the current title reveals the intended topic.
│  → Informative? STOP. Extract keyword from current title.
│
Step 6: Page Content Scan (more tokens)
│  Fetch first 500 words of visible body text.
│  Extract the dominant topic from content.
│  This is the most expensive step — use only as last resort.
│  → STOP. Derive keyword from content analysis.
```

**For reference pages (Full-List + Subset Mode):**
Only run Steps 1-2 of this chain. URL slug + breadcrumb is enough to build the reserved keyword map. Do NOT crawl full content for reference pages — too expensive.

**For active pages:**
Run the full chain as needed until topic is clear.

**Implementation note:** When using web_fetch or browser tools to check schemas/H1, extract ONLY the specific data needed. Do not load the entire page into context. Use targeted extraction:
- For schema: look for `<script type="application/ld+json">` blocks only
- For H1: extract just the `<h1>` tag
- For breadcrumb: look for BreadcrumbList in schema or `.breadcrumb` / `nav[aria-label="breadcrumb"]` in HTML

### 3d. Primary Keyword Selection
Combine page content (3a), ranking data (3b), and fallback chain (3c) to select the **primary keyword**:
- Highest search volume among keywords that match the page's actual content
- Prefer keywords where the page already has ranking momentum (positions 4-20 = opportunity zone)
- If no ranking data exists, derive the primary keyword from H1 + page content topic
- **Check against reserved keyword map** (from reference pages) — if the best keyword is reserved, pick the next best alternative

### 3d. Secondary Keyword Discovery
Use **DataForSEO Labs → Ranked Keywords** on the top 3-5 SERP competitors for the primary keyword:
- Find keyword variations the competitors rank for that our page doesn't
- Select 1-2 secondary keywords that are semantically related and naturally fit into title/description
- These become modifiers or supporting terms

### 3e. Search Volume Validation
Use **Keyword Data API → Search Volume** to confirm volumes for primary + secondary keywords.
Attach volume data to the output.

## Step 4: SERP Competitor Analysis (DataForSEO Enhanced)

> Skip if DataForSEO MCP is not available.

Read `references/dataforseo-integration.md` for API details.

### 4a. Fetch SERP Data
Use **SERP API → Live Advanced** for each primary keyword. Extract from top 10 organic results:
- Title text
- Description text
- URL
- Position

### 4b. Structural Analysis
Analyze the collected titles and descriptions:

**Title patterns to detect:**
- Most common modifiers (Fiyatları, Modelleri, Best, Guide, Review, etc.)
- Average character length
- Keyword position distribution (front-loaded vs. mid vs. end)
- Separator usage (pipe vs. dash vs. none)
- Brand inclusion rate
- Use of numbers, years, superlatives

**Description patterns to detect:**
- Dominant CTA type (keşfedin, inceleyin, discover, shop now, etc.)
- USP mentions (free shipping, guarantee, discount percentages)
- Use of numerical data (100+ models, 50% off)
- Emotional vs. functional tone ratio
- Average character length

### 4c. Strategic Recommendations
Based on analysis, determine for each keyword:

1. **Must-match elements** — modifiers/CTAs that 70%+ of competitors use (table stakes)
2. **Differentiation opportunities** — angles no competitor is using but are relevant
3. **Length strategy** — match or slightly exceed competitor average to maximize SERP real estate
4. **Tone calibration** — align with sector expectations while staying on-brand

Format this as a brief internal note that informs title/description generation.

## Step 5: Generate Titles and Descriptions

Read `references/style-rules.md` for the full rule set. Key principles:

### Title Generation Rules
- **Max 70 characters** (brand included), ideal range 55-65
- **Primary keyword at the front** whenever grammatically natural
- **Brand at the end** with `|` separator: `Primary KW + Modifiers | Brand`
- **Modifier selection is data-driven** — read `references/style-rules.md` Section 4 for the full decision tree:
  - First choice: pick the modifier with the **highest search volume** (e.g., if "lavabo modelleri" has 12K and "lavabo fiyatları" has 8K, use "modelleri" first)
  - Second choice: if no volume data, pick the modifier **most used by SERP competitors** (e.g., 7/10 competitors use "Fiyatları" → use it)
  - Third choice: if competitors use something unexpected (e.g., "Özellikleri" or "Karşılaştırma" dominate instead of "Fiyatları"), **follow the SERP**, not the default
  - Last resort only: use category-based defaults from the style rules
- **"ve" en fazla 1 kez** — title'da "ve" tekrar etme. İkinci bağlaç için `&` veya `,` kullan:
  - Kötü: `Ruj Çeşitleri ve Renkleri, Ruj Modelleri ve Fiyatları | Flormar`
  - İyi: `Ruj Çeşitleri & Renkleri, Ruj Modelleri ve Fiyatları | Flormar`
- **Kategori hiyerarşisine dikkat** — alt kategori sayfası için üst kategorinin genel keyword'ünü kullanma. URL slug'ındaki niteleyici (likit, ahşap, köşe, çocuk vb.) keyword'de mutlaka korunmalı:
  - Kötü: `/likit-kapatici` → `Kapatıcı Çeşitleri ve Fiyatları`
  - İyi: `/likit-kapatici` → `Likit Kapatıcı Çeşitleri ve Fiyatları, Likit Concealer`
- **No year** in titles unless explicitly requested or blog content
- **No .com/.com.tr** in titles
- **No subjective superlatives** like "En İyi" unless SERP data shows competitors using it heavily
- **Language-appropriate modifiers** — Turkish: "Fiyatları", "Modelleri", "Çeşitleri"; English: "Prices", "Models", "Guide"

### Title Formulas by Page Type

**PLP (Category):**
```
[Primary KW] [Modifier1] ve [Modifier2] | Brand
Example: Lavabo Modelleri ve Fiyatları | VitrA
```

**Brand-Filtered PLP:**
```
[Brand Filter] [Category KW] [Modifier] | Platform
Example: Altus Blender & Mikser Fiyatları - Pasaj
```

**Ghost/Filter PLP (attribute-based):**
```
[Attribute] [Category KW] [Modifier], [Variant KW] | Brand
Example: Siyah Lavabo Fiyatları & Modelleri, Siyah Banyo Lavabosu | VitrA
```

**PDP (Product):**
```
[Product Name] [Key Feature/Spec] | Brand
Example: V-Care Comfort Akıllı Klozet Özellikleri | VitrA
```

**Blog/Content:**
```
[Topic KW] [Hook/Angle] | Brand
Example: Küçük Banyo Dekorasyon Fikirleri ve İpuçları | VitrA Blog
```

**Service/Info Page:**
```
[Descriptive phrase with KW] | Brand
Example: Marmaris'te Lüks Konaklama Seçenekleri | D Maris Bay
```

**Homepage:**
```
[Brand Value Proposition] | Brand
Example: Exclusive Beachfront Resort in Marmaris, Turkey | D Maris Bay
```

### Description Generation Rules
- **Max 160 characters**, ideal range 140-155
- **Primary keyword in first sentence** — natural integration, not stuffed
- **Secondary keyword woven in** if space allows
- **End with CTA** appropriate to brand tone (see style-rules.md for CTA ladder)
- **No repetition** of the exact title text in description
- **Include a differentiator** when SERP analysis reveals opportunity

### CTA Tone Ladder (sector-based)
```
Aggressive (e-commerce):  "hemen satın alın", "şimdi keşfedin", "fırsatları kaçırmayın"
Moderate (retail/tech):   "keşfedin", "inceleyin", "karşılaştırın", "göz atın"
Soft (premium/luxury):    "keşfedin", "ilham alın", "deneyimleyin"
Informational (corporate): "detaylı bilgi için", "daha fazlasını öğrenin"
```

### Multi-keyword Title Strategy
When space allows and SERP shows multi-intent results, combine keywords:
```
[Primary KW], [Secondary KW variant] | Brand
Max 2-3 keywords, separated by comma or &
Only if total stays under 70 characters
```

## Step 6: Cannibalization Check

After generating all titles, run a **two-layer** cannibalization check:

### Layer 1: Active Batch Check
Check within the active pages being processed:

1. **Exact match:** Same primary keyword assigned to 2+ active URLs → **BLOCK** — reassign one
2. **Near match:** Very similar keywords (e.g., "lavabo modelleri" vs "lavabo çeşitleri") → **WARNING** with note
3. **Semantic overlap:** Different keywords but same search intent → **INFO** flag

### Layer 2: Full-Site Check (if reference pages provided)
Check active page keywords against the **reserved keyword map** built from reference pages:

1. **Active page keyword matches a reference page's keyword** → **BLOCK** — the reference page owns that keyword. Pick an alternative for the active page.
2. **Active page keyword is a parent of a reference page's keyword** → **WARNING** — e.g., active targets "kapatıcı" but reference page `/likit-kapatici` exists. Active page may be too broad.
3. **Active page keyword is a child of a reference page's keyword** → **OK** — this is expected (alt kategori daha spesifik olmalı zaten).

### Resolution Actions
For each flagged pair, suggest:
- Which URL should keep the keyword (based on content relevance + ranking data + URL hierarchy)
- Alternative keyword for the conflicting URL
- Or consolidation recommendation if pages should be merged

Mark cannibalization status in output:
- ✅ Clean — no conflicts
- ⚠️ Near match — review recommended  
- 🔴 Duplicate target — must resolve
- 🟡 Reserved by reference page — alternative keyword assigned

## Step 7: Output

### Chat Output (always)
Display a table with:
```
| # | URL | Page Type | Primary KW | Sec. KW | New Title | T.Len | New Description | D.Len | Cannib. |
```

If DataForSEO was used, add:
```
| Pri. Vol | Sec. Vol | SERP Top Modifier | Differentiation Note |
```

### Excel Output (automatically for 10+ pages, or when user asks)
Read the xlsx skill (`/mnt/skills/public/xlsx/SKILL.md`) before creating the Excel file. Do not ask "Excel oluşturayım mı?" — if there are 10+ pages, just create it.

**Sheet 1: "Title & Description"** — Main output table with all columns above
**Sheet 2: "SERP Analysis"** (if DataForSEO used) — Per-keyword breakdown:
- Keyword, Top 10 competitor titles, Top 10 competitor descriptions
- Modifier frequency, CTA frequency, Avg title/desc length
- Differentiation notes

**Sheet 3: "Cannibalization Report"** (if flags exist) — Flagged pairs with recommendations

### Formatting standards for Excel:
- Header row frozen and bold
- URL column as hyperlinks
- Conditional formatting on character length (green ≤60 / ≤155, yellow 61-70 / 156-160, red >70 / >160)
- Cannibalization column color-coded (✅ green, ⚠️ yellow, 🔴 red)

---

## Working Without DataForSEO

When DataForSEO MCP is not available, the skill still works — just with user-provided data:

1. User provides URL + keyword pairs (required in this mode)
2. Skip Steps 3 and 4
3. Page type is auto-detected from URL pattern
4. Titles and descriptions are generated using the style rules, brand profile, and page type formulas
5. Cannibalization check still runs on the provided keywords
6. No search volume or SERP analysis columns in output

Mention briefly at the start of output that keyword discovery and SERP analysis were skipped because DataForSEO is not connected. Do not over-explain or ask if the user wants to set it up.

---

## Quality Checklist (run before delivering output)

Before presenting results, verify every row against:

- [ ] Title ≤ 70 characters (including brand + separator)
- [ ] Description ≤ 160 characters
- [ ] Primary keyword appears in title (preferably front-loaded)
- [ ] Primary keyword appears in description (first sentence)
- [ ] Brand name is at the end of title with correct separator
- [ ] No year in title (unless blog or explicitly requested)
- [ ] No .com/.com.tr in title
- [ ] CTA tone matches brand/sector profile
- [ ] No exact cannibalization within the batch
- [ ] Description does not repeat title verbatim
- [ ] Language is consistent throughout (no mixed Turkish/English)
