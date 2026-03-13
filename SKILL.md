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

This skill runs end-to-end without asking for permission at ANY intermediate step. Once the user provides URLs and brand info, execute the entire workflow silently and deliver the final output.

**Do NOT ask for confirmation before:**
- Making DataForSEO API calls (On-Page, SERP, Keyword Data, Labs)
- Crawling pages or fetching SERP data
- Selecting primary/secondary keywords
- Choosing modifiers based on volume or SERP data
- Generating titles and descriptions
- Creating Excel files
- Running cannibalization checks
- Reading reference files (style-rules.md, brand-profiles.md, dataforseo-integration.md)
- Reading skill files or any bundled resource
- Running bash commands (for data processing, file operations, etc.)
- Saving files to disk (keyword data, SERP results, intermediate outputs, final Excel)
- Fetching/visiting web pages (for page content analysis, schema checks, H1 extraction)
- Writing temporary files during processing
- Installing packages needed for Excel generation or data processing

**File & tool operations — JUST DO IT:**
- Need to read a reference file? Read it. Don't say "Let me check the style rules" — just read and apply.
- Need to save ranked keywords to a file? Save it. Don't ask "Shall I save this data?"
- Need to run a bash command to process data? Run it. Don't ask permission.
- Need to create an Excel file? Create it. Don't ask "Excel oluşturayım mı?"
- Need to fetch a URL to check its content/schema/H1? Fetch it. Don't ask "Sayfayı ziyaret edeyim mi?"
- Need to write intermediate results? Write them. Don't narrate file operations.

**Only pause and ask the user if:**
- Brand name is missing and cannot be inferred
- Language is ambiguous (e.g., mixed TLDs, unclear URL patterns)
- A critical error persists after all retry/alternative attempts have been exhausted
- The user explicitly asks for a review before final output

**Error behavior — DO NOT SKIP, FIX:**
- If a DataForSEO call fails, DO NOT silently skip it. Stop, diagnose the error, try alternatives (different location code, different endpoint, retry), and only report to the user if all alternatives fail.
- Common issue: location-based errors. ALWAYS use `location_code: 2792` (integer) for Turkey, or `location_name: "Turkiye"` (NOT "Turkey"). See `references/dataforseo-integration.md` Section "Location Parameter Rules" for details.
- If an API returns unexpected structure, try parsing differently before giving up.
- If a result is too large for context, save it to a file and process from there — do not ask permission.
- Deliver the complete result set in one response, not piece by piece.
- When the user provides enough context (URLs + brand), skip Step 1 questions entirely and go straight to work.

**General behavior:**
- If DataForSEO MCP tools are available, use them immediately — do not ask "Shall I use DataForSEO?" or "Do you want me to analyze the SERP?"
- Batch all API calls efficiently — do not narrate each call or ask between them
- Do not explain what you're about to do — just do it and present the results

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
4. **Hitap dili** — "siz" (default) veya "sen". Kullanıcı belirtmezse "siz" kullan. Brand profile'dan da çıkarılabilir.
5. **DataForSEO** — if MCP tools are available, use them silently. Do not ask.

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

Primary keyword, o sayfayı/kategoriyi **en iyi şekilde ifade eden keyword** olmalıdır. Hacim önemlidir ama tek kriter değildir — keyword sayfanın tam karşılığı olmalı.

**Seçim süreci:**

1. Sayfa içeriği (3a) + ranking verisi (3b) + fallback chain (3c) sonuçlarını birleştir
2. Sayfayı en doğru ifade eden keyword adayını belirle
3. Bu adayın arama hacmini kontrol et

**Eğer hacim düşük veya sıfır çıkıyorsa — DURMA, ekstra araştırma yap:**
- Aynı konseptin farklı aranma şekillerini dene:
  - Eş anlamlılar: "berjer" vs "berjer koltuk", "baza" vs "somya"
  - Türkçe-İngilizce: "kapatıcı" vs "concealer"
  - Tekil-çoğul: "koltuk" vs "koltuklar"
  - Ekli-eksiz: "koltuk fiyatları" vs "koltuk fiyat"
  - Farklı yazımlar: "şezlong" vs "şezlöng", "gardırop" vs "gardrop"
- SERP'teki rakiplerin title'larında hangi terimi kullandığına bak
- DataForSEO'da keyword suggestions/related keywords endpoint'lerini kullan
- Her varyasyonun hacmini karşılaştır, en yüksek hacimli VE sayfayı doğru ifade edeni seç

**Çatı/birleşik kategori durumları:**
Eğer URL veya kategori iki kavramı birleştiriyorsa (ör. `/oturma-grubu-ve-koltuk-takimi`), primary keyword her iki konsepti kapsamalı. Tek bir terimi seçip diğerini düşürme.

**Check against reserved keyword map** (from reference pages) — if the best keyword is reserved, pick the next best alternative.

### 3e. Secondary Keyword Discovery

Use **DataForSEO Labs → Ranked Keywords** on the top 3-5 SERP competitors for the primary keyword:
- Find keyword variations the competitors rank for that our page doesn't
- **Secondary keyword sayısı sınırsızdır** — relevan olan tüm varyasyonları topla
- Title'a sığanlar title'a, sığmayanlar description'a yerleştirilir
- Aynı hücreye virgülle yazılır: `berjer koltuk, tekli koltuk, berjer modelleri`
- Farklı yazım şekillerini de dahil et (ör. "TV ünitesi" ve "televizyon ünitesi" ayrı secondary keyword olabilir)

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

### 4d. Keyword Variant Research

SERP verisi ve rakip keyword'leri incelerken, hedef keyword'ün **farklı yazım şekillerini, eş anlamlılarını ve dil varyasyonlarını** tespit et:

**Tespit edilecek varyasyon türleri:**
- **Türkçe-İngilizce karışımlar:** "kapatıcı" vs "concealer", "ruj" vs "lipstick", "fondöten" vs "foundation", "maskara" vs "rimel"
- **Yazım varyasyonları:** "şezlong" vs "şezlöng", "klozet" vs "tuvalet"
- **Eş anlamlılar / Kullanıcı gözünde aynı anlam:** "koltuk" vs "kanepe", "lavabo" vs "evye", "küvet" vs "banyo küveti", "dolap" vs "ünite"
- **Renk/materyal/özellik varyantları:** "siyah lavabo", "ahşap dolap", "köşe küvet"
- **Modifier varyasyonları:** "x modelleri", "x fiyatları", "x çeşitleri", "x fiyat" (eksiz kullanım)

**Varyasyon keşif yöntemleri:**
1. SERP'teki rakip title'lardan farklı yazım/terim kullanımlarını çıkar
2. DataForSEO Labs → Ranked Keywords'te rakip URL'lerin rank aldığı keyword'lerdeki varyasyonları bul
3. Search volume API ile varyasyonların hacimlerini karşılaştır

**Varyasyonları title/description'a yerleştirme kuralı:**

Her varyasyonun arama hacmini kontrol et. Hacim sıralamasına göre yerleştir:
- **En yüksek hacimli varyasyon → Title'da başta** (primary keyword olarak)
- **İkinci yüksek hacimli → Title'da virgülden sonra** (yer varsa, 70 char limiti içinde)
- **Düşük hacimli ama relevan varyasyonlar → Description'a** doğal şekilde yerleştir

**Çift hedefleme örnekleri (her iki terimi de title'a koy):**

| Durum | Kötü (tek terim) | İyi (çift hedefleme) |
|-------|-----------------|---------------------|
| Türkçe-İngilizce karışım | `Kapatıcı Fiyatları \| Flormar` | `Kapatıcı, Concealer Modelleri ve Fiyatları \| Flormar` |
| Eş anlamlı kullanım | `3'lü Koltuk Modelleri \| Doğtaş` | `3'lü Koltuk & 3'lü Kanepe Modelleri ve Fiyatları \| Doğtaş` |
| Türkçe-İngilizce yakın terim | `Maskara Fiyatları \| Flormar` | `Maskara & Rimel Fiyatları, Rimel Modelleri \| Flormar` |
| Aynı ürün farklı isim | `Baza Fiyatları \| Lova` | `Baza & Somya Modelleri ve Fiyatları \| Lova` |

**Ne zaman sayfayı ziyaret et:**
Eğer hangi varyasyonun bu sayfa için daha uygun olduğuna karar veremiyorsan (ör. `/concealer` sayfası "kapatıcı" mı "concealer" mı odaklı?), sayfayı ziyaret et:
- H1'e bak: marka hangi terimi tercih etmiş?
- Ürün isimlerine bak: ürün kartlarında hangi terim kullanılıyor?
- Breadcrumb'a bak: kategori yapısında hangi isimle yer alıyor?
- Bu bilgiyle birincil terimi belirle, diğerini ikincil olarak title'a veya description'a ekle

## Step 5: Generate Titles and Descriptions

Read `references/style-rules.md` for the full rule set. Key principles:

### Title Generation Rules
- **Max 575px / 70 karakter** (brand dahil). Piksel birincil ölçü — Türkçe geniş karakterler (ş, ğ, ü, ö, m, w) daha fazla piksel kaplar. Detaylar: `references/style-rules.md` Section 1.
- **Title'ı KISA BIRAKMA** — 575px sınırını sonuna kadar kullan. 400px'lik bir title varsa, yer boşa gidiyor demektir. Kalan alana keyword varyasyonu, secondary keyword veya ek modifier ekle.
- **Primary keyword at the front** whenever grammatically natural
- **Brand at the end** with `|` separator: `Primary KW + Modifiers | Brand`
- **URL slug'a körü körüne güvenme** — URL'deki beden, parametre, İngilizce terim veya teknik ifade yanıltıcı olabilir. URL'yi ipucu olarak kullan ama sayfanın gerçek içeriğine göre karar ver:
  - `/bebek-kiyafetleri-50104-beden` → beden bilgisini title'a taşıma, "Bebek Kıyafetleri, Kız & Erkek Bebek Kıyafetleri" daha iyi hedefleme
  - `/leggings` → TR site ise "tayt" hacmini kontrol et, Türkçe hacim yüksekse onu primary yap
- **TR sitede İngilizce terime dikkat** — URL'de İngilizce terim olsa bile Türkçe karşılığının hacmini kontrol et. Türkçe hacim ≥ İngilizce ise Türkçeyi primary yap. İngilizce terimi sadece ek hacim getirecekse secondary olarak ekle.
- **Modifier selection is data-driven** — read `references/style-rules.md` Section 4 for the full decision tree
- **"ve" en fazla 1 kez** — ikinci bağlaç için `&` veya `,` kullan
- **Kategori hiyerarşisine dikkat** — alt kategori sayfası için üst kategorinin genel keyword'ünü kullanma
- **No year** in titles unless explicitly requested or blog content
- **No .com/.com.tr** in titles
- **No subjective superlatives** like "En İyi" unless SERP data shows competitors using it heavily

### Keyword-First Pattern — Tek Kelimelik Kategoriler

Bazı kategoriler tek bir keyword ile tanınır (berjer, puf, sifonyer, komodin, karyola vb.). Bu tür sayfalarda keyword'ü önce tek başına yaz, sonra modifier'lı versiyonunu ekle. Bu sayede hem exact-match hem modifier'lı sorguları yakalar:

```
[Keyword], [Keyword + Varyasyon] [Modifier1] ve [Modifier2] | Brand
```

**Örnekler:**
| URL | Kötü (kısa, fırsat kaçırıyor) | İyi (sınır sonuna kadar kullanılmış) |
|-----|------|-----|
| `/berjer` | `Berjer Koltuk Modelleri \| Brand` (33 char) | `Berjer, Berjer Koltuk Modelleri ve Fiyatları \| Brand` (54 char) |
| `/puf` | `Puf Modelleri \| Brand` (22 char) | `Puf, Puf Koltuk & Oturma Pufu Modelleri ve Fiyatları \| Brand` (61 char) |
| `/sifonyer` | `Sifonyer Modelleri \| Brand` (28 char) | `Sifonyer, Sifonyer Modelleri ve Fiyatları \| Brand` (51 char) |
| `/komodin` | `Komodin Fiyatları \| Brand` (27 char) | `Komodin, Komodin Modelleri ve Fiyatları \| Brand` (49 char) |

### Çatı / Birleşik Kategori Sayfaları

URL veya kategori birden fazla konsepti kapsıyorsa, title'da her iki konsepti de yansıt:

```
[Konsept1] & [Konsept2] [Modifier] | Brand
```

**Örnekler:**
| URL | Title |
|-----|-------|
| `/oturma-grubu-ve-koltuk-takimi` | `Oturma Grubu & Koltuk Takımı Modelleri ve Fiyatları \| Brand` |
| `/yatak-ve-baza` | `Yatak & Baza Modelleri ve Fiyatları, Yatak Çeşitleri \| Brand` |
| `/mutfak-ve-banyo-mobilyasi` | `Mutfak & Banyo Mobilyası Modelleri ve Fiyatları \| Brand` |

Tek bir tarafı seçip diğerini düşürme — iki keyword de hacim taşıyorsa ikisini de title'da tut.

### Title Formulas by Page Type

**PLP (Category):**
```
[Primary KW], [Primary KW + Varyasyon] [Modifier1] ve [Modifier2] | Brand
Example: Berjer, Berjer Koltuk Modelleri ve Fiyatları | Kelebek
Example: Lavabo Modelleri ve Fiyatları, Banyo Lavabo Çeşitleri | VitrA
```

**Brand-Filtered PLP:**
```
[Brand Filter] [Category KW] [Modifier] | Platform
Example: Altus Blender & Mikser Fiyatları | Pasaj
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
- **Max 980px / 160 karakter**, ideal range 140-155 char. Piksel birincil ölçü.
- **Primary keyword in first sentence** — natural integration, not stuffed
- **Secondary keyword woven in** if space allows
- **CTA pozisyonu esnek** — CTA her zaman sonda olmak zorunda değil. Yerine göre:
  - İlk cümle ürün/fayda → ikinci cümle CTA
  - İlk cümle CTA → ikinci cümle detay
  - Tek cümlede doğal entegre CTA
  - Farklı cümle yapıları kullanarak batch genelinde çeşitlilik sağla
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
Only if total stays under 575px / 70 characters
```

### Alternatif Title Önerisi (A/B)

Karmaşık sayfalarda (varyasyon kararı, çatı kategori, çift hedefleme, birden fazla güçlü strateji) tek title yerine **2 alternatif title** sun:

**Ne zaman A/B öner:**
- İki farklı primary keyword adayı birbirine çok yakın hacimde
- Çift hedefleme (TR/EN) ile tek hedefleme arasında karar verilemiyor
- Çatı kategoride hangi konsepti öne çıkaracağın belirsiz
- SERP'te iki farklı dominant pattern var

**Format:**
```
Title A: Berjer, Berjer Koltuk Modelleri ve Fiyatları | Kelebek (575px)
Title B: Berjer Koltuk & Tekli Koltuk Modelleri ve Fiyatları | Kelebek (570px)
Not: A → exact-match "berjer" yakalar + modifier'lı versiyon. B → "tekli koltuk" varyasyonunu da kapsar (8K ek hacim). A önerisi: berjer odaklı sayfaysa. B önerisi: geniş oturma grubu sayfasıysa.
```

Her sayfa için A/B sunma — sadece gerçekten iki güçlü stratejinin olduğu satırlar için. Basit sayfalarda tek title yeterli.

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
- **Yok** — no conflicts
- **Risk** — near match or semantic overlap, review recommended
- **Var** — exact duplicate target or reserved by reference page, must resolve

## Step 7: Output

### Chat Output (always)
Display a table with:
```
| # | URL | Page Type | Primary KW | Sec. KW | New Title | T.Len | T.Px | New Description | D.Len | D.Px | Cannib. | Öncelik | Notlar |
```

If DataForSEO was used, add:
```
| Pri. Vol | Sec. Vol | SERP Top Modifier | Differentiation Note |
```

If alternative titles were suggested (A/B), add a second row for that URL with "Alt. Title" label.

### Öncelik Kolonu — DataForSEO Bazlı Değerlendirme

Her satıra **High / Medium / Low** öncelik ata. Bu, kullanıcıya "önce şunları uygula" diyebilmesi için:

**Öncelik belirleme kriterleri (DataForSEO verisiyle):**

**High (Yüksek):**
- Primary keyword arama hacmi yüksek (≥5K) VE
- Sayfa SERP'te iyi pozisyonda değil (rank >10 veya rank yok) VEYA
- Mevcut title ile önerilen title arasında ciddi fark var (farklı keyword hedefi, eksik modifier) VEYA
- Rakipler bu keyword'de güçlü title'larla sıralanıyor, bizim title zayıf

**Medium (Orta):**
- Primary keyword orta hacimde (1K-5K) VEYA
- Sayfa zaten kısmen iyi pozisyonda (rank 5-10) ama title optimize edilebilir VEYA
- Mevcut title makul ama iyileştirme potansiyeli var

**Low (Düşük):**
- Primary keyword düşük hacimde (<1K) VEYA
- Sayfa zaten iyi pozisyonda (rank 1-5) ve mevcut title yeterli VEYA
- Mevcut title ile önerilen title arasında minimal fark

**DataForSEO olmadan (manual keyword sağlandıysa):**
- Sadece keyword + URL pattern'e göre kabaca ata
- Hacim bilinmiyorsa Medium default yap

### Notlar Kolonu — Seçim Metodolojisi

**Notlar kolonu her satır için zorunludur.** Özellikle şu durumlarda detaylı açıklama yaz:

- **Varyasyon tercihi yapıldıysa:** Hangi varyasyonlar değerlendirildi, hacimleri ne, neden bu seçim yapıldı
  - Örnek: `"kapatıcı" (14K) vs "concealer" (8K). İkisi de title'a eklendi. Primary olarak "kapatıcı" seçildi (daha yüksek hacim). Concealer description ve title'da secondary olarak yer aldı.`
- **Çift hedefleme yapıldıysa:** Neden iki terimi birden title'a koydun
  - Örnek: `"koltuk" ve "kanepe" kullanıcı gözünde eş anlamlı. SERP'te 6/10 rakip her ikisini de kullanıyor. İkisi de title'a eklendi.`
- **URL slug'ı yok sayıldıysa:** URL'deki bilgi neden title'a taşınmadı
  - Örnek: `URL'de "50-104 beden" var ama bu kullanıcı arama terimi değil. Beden bilgisi çıkarılıp "Bebek Kıyafetleri, Kız & Erkek Bebek Kıyafetleri" hedeflemesi yapıldı.`
- **EN→TR dönüşüm yapıldıysa:** Neden İngilizce terim yerine Türkçe tercih edildi
  - Örnek: `URL'de "leggings" var ama "tayt" hacmi (18K) >> "leggings" (1.2K). Primary olarak "Çocuk Tayt" seçildi.`
- **Sayfa ziyaret edildiyse:** Ne gördüğün ve bu bilginin kararını nasıl etkilediği
- **SERP'ten farklı bir yol seçildiyse:** Rakiplerden neden ayrıştın
- **Cannibalization riski varsa:** Detaylandır
- **A/B title sunduysan:** Her iki alternatifin mantığı

**Basit, standart title/description'lar için kısa not yeterli:**
- `Standart PLP. "Modelleri ve Fiyatları" modifier'ı SERP hacim verisine göre seçildi. Rank 12, hacim 8K → High.`

### Excel Output (automatically for 10+ pages, or when user asks)
Read the xlsx skill (`/mnt/skills/public/xlsx/SKILL.md`) before creating the Excel file. Do not ask "Excel oluşturayım mı?" — if there are 10+ pages, just create it.

**Sheet 1: "Title & Description"** — Main output table with all columns above, including Notlar column
**Sheet 2: "SERP Analysis"** (if DataForSEO used) — Per-keyword breakdown:
- Keyword, Top 10 competitor titles, Top 10 competitor descriptions
- Modifier frequency, CTA frequency, Avg title/desc length
- Keyword varyasyonları ve hacimleri
- Differentiation notes

**Sheet 3: "Cannibalization Report"** (if flags exist) — Flagged pairs with recommendations

### Formatting standards for Excel:
- Header row frozen and bold
- URL column as hyperlinks
- T.Px column: green ≤500px, yellow 501-575px, red >575px
- D.Px column: green ≤850px, yellow 851-980px, red >980px
- Cannibalization column: `Yok` = green, `Risk` = yellow, `Var` = red
- Öncelik column: `High` = red background, `Medium` = yellow, `Low` = green

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

- [ ] Title ≤ 575px / 70 characters (brand + separator dahil)
- [ ] Title ≥ 400px / 50 characters — kısa bırakma, keyword varyasyonu ekle
- [ ] Description ≤ 980px / 160 characters
- [ ] Primary keyword sayfayı/kategoriyi doğru ifade ediyor (hacim düşükse alternatif araştırılmış)
- [ ] URL slug'ındaki yanıltıcı bilgiler (beden, parametre, EN terim) title'a körü körüne taşınmamış
- [ ] TR hedefli sayfada İngilizce terim varsa Türkçe hacim kontrol edilmiş
- [ ] Primary keyword appears in title (preferably front-loaded)
- [ ] Primary keyword appears in description
- [ ] Secondary keywords virgülle ayrılmış, aynı hücrede
- [ ] Brand name is at the end of title with `|` separator
- [ ] No year in title (unless blog or explicitly requested)
- [ ] No .com/.com.tr in title
- [ ] "ve" title'da en fazla 1 kez kullanılmış
- [ ] Alt kategori sayfası üst kategorinin genel keyword'ünü çalmıyor
- [ ] Keyword varyasyonları (TR/EN, eş anlamlılar, farklı yazımlar) hacim sırasına göre yerleştirilmiş
- [ ] Tek kelimelik kategorilerde "Keyword, Keyword Varyasyon" pattern'i uygulanmış
- [ ] Çatı/birleşik kategorilerde her iki konsept de title'da yer alıyor
- [ ] CTA pozisyonu esnek tutulmuş, her description aynı yapıda değil
- [ ] Siz/sen dili tutarlı
- [ ] Cannibalization: Var / Yok / Risk olarak işaretlenmiş
- [ ] Öncelik: High / Medium / Low atanmış (DataForSEO verisine göre)
- [ ] Description does not repeat title verbatim
- [ ] Description'lar arasında cümle yapısı ve CTA çeşitliliği var
- [ ] Karmaşık satırlarda A/B alternatif title sunulmuş (gerekiyorsa)
- [ ] Notlar kolonu doldurulmuş
