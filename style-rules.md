# SEO Meta Title & Description — Style Rules

This document contains the detailed style rules derived from analysis of 1,000+ real title/description examples across multiple brands and languages.

## Table of Contents
1. Character Limits
2. Title Structure Rules
3. Description Structure Rules
4. Language-Specific Modifier Library
5. CTA Library by Tone Level
6. Anti-Patterns
7. Special Cases

---

## 1. Character Limits

### Title
| Zone | Characters | Guidance |
|------|-----------|----------|
| Ideal | 55–65 | Target this range for most pages |
| Acceptable | 50–70 | OK if needed for keyword coverage |
| Hard max | 70 | Never exceed — gets truncated in SERP |
| Minimum | 30 | Below this looks thin in SERP |

Note: Character count includes the brand suffix (e.g., ` | VitrA` = 8 chars, ` - Pasaj` = 8 chars). Always count brand as part of the limit.

### Description
| Zone | Characters | Guidance |
|------|-----------|----------|
| Ideal | 140–155 | Optimal SERP display |
| Acceptable | 120–160 | OK range |
| Hard max | 160 | Truncation risk above this |
| Minimum | 100 | Below this wastes SERP real estate |

---

## 2. Title Structure Rules

### Keyword Placement
Place the primary keyword as early in the title as possible. The first 40 characters are the most visible and carry the most SEO weight.

Good: `Lavabo Modelleri ve Fiyatları | VitrA`
Bad: `VitrA'da En Güzel Lavabo Modelleri ve Fiyatları`

### Brand Positioning
Brand ALWAYS goes at the end, separated by the brand's configured separator.

```
[Content portion] | [Brand]
```

Never put the brand at the beginning unless it's a homepage or a brand-awareness page.

### Separator Rules
All brands use **pipe `|`** as the standard separator. Separator always has spaces on both sides: ` | ` not `|`

### Multi-keyword Titles
When including multiple keyword variations in a title, use comma or ampersand:

```
Primary KW Modifiers, Secondary KW Variant | Brand
Primary KW & Secondary KW Modifier | Brand
```

Rules:
- Maximum 2-3 distinct keyword groups in one title
- Only if total stays within 70 character limit
- Each keyword group should be a natural phrase, not keyword-stuffed
- Comma for distinct concepts, `&` for closely related concepts

**Example (good):**
`Siyah Lavabo Fiyatları & Modelleri, Siyah Banyo Lavabosu | VitrA` (64 chars)

**Example (bad — too stuffed):**
`Siyah Lavabo, Siyah Banyo Lavabosu, Siyah Evye Fiyatları Modelleri | VitrA` (75 chars — too long, unnatural)

### "ve" Tekrar Yasağı — Çeşitlilik Kuralı

Title içinde **"ve" bağlacı en fazla 1 kez** kullanılmalı. Birden fazla "ve" title'ı tekdüze ve uzun yapar. Bunun yerine `,` ve `&` ile çeşitle:

**Kötü (tekrarlı "ve"):**
```
Ruj Çeşitleri ve Renkleri, Ruj Modelleri ve Fiyatları | Flormar
Maskara ve Rimel Çeşitleri ve Fiyatları, Rimel Modelleri | Flormar
```

**İyi (çeşitlendirilmiş bağlaçlar):**
```
Ruj Çeşitleri & Renkleri, Ruj Modelleri ve Fiyatları | Flormar
Maskara & Rimel Fiyatları, Rimel Modelleri | Flormar
```

**Bağlaç seçim kuralı:**
- `&` → Yan yana gelen ve birbirine çok yakın iki kavram: `Maskara & Rimel`, `Modelleri & Fiyatları`
- `,` → Farklı keyword gruplarını ayırırken: `Ruj Çeşitleri & Renkleri, Ruj Modelleri`
- `ve` → Title içinde en fazla 1 kez, genellikle son modifier çiftinde: `Modelleri ve Fiyatları`
- Bir title'da `ve` + `&` + `,` üçü birden varsa → çok kalabalık, sadeleştir

**Description'larda** "ve" tekrarı daha tolere edilebilir çünkü doğal cümle yapısı gerektirir, ama yine de aynı cümle içinde 2'den fazla "ve" kullanılmamalı.

### Kategori Hiyerarşisi Farkındalığı — Kapsayıcı Keyword Hatası

Bir alt kategori sayfası için title yazarken, **o keyword'ün üst kategoriye ait olup olmadığını kontrol et.** Aynı sitede üst kategori sayfası zaten o genel keyword'ü hedefliyordur — alt kategori sayfası daha spesifik olmalı.

**Temel kural:** Eğer yazdığın keyword tek başına bir üst kategoriyi tanımlayabilecek kadar genişse, o keyword bu alt kategori için yanlış seçimdir. Keyword'ü daralt ve spesifikleştir.

**Kötü — üst kategori keyword'ünü çalıyor:**
```
URL: /likit-kapatici
Title: Kapatıcı Çeşitleri ve Fiyatları | Flormar
→ YANLIŞ: "Kapatıcı" genel kategorinin keyword'ü, bu sadece likit kapatıcı sayfası
```

**İyi — spesifik ve daraltılmış:**
```
URL: /likit-kapatici
Title: Likit Kapatıcı Çeşitleri ve Fiyatları, Likit Concealer | Flormar
→ DOĞRU: "Likit" niteleyicisi her keyword grubunda korunuyor
```

**Nasıl kontrol edilir:**
1. Yazdığın primary keyword'ü al: ör. "kapatıcı"
2. Kendin sor: "Bu sitede sadece 'kapatıcı' diye ayrı bir kategori sayfası var mı veya olabilir mi?"
3. Evetse → keyword'ü daralt: "likit kapatıcı", "stick kapatıcı", "göz altı kapatıcısı"
4. URL slug'ı sana ipucu verir: `/likit-kapatici` → keyword'de "likit" mutlaka olmalı

**Yaygın hata örnekleri ve düzeltmeleri:**

| URL | Kötü Title | İyi Title |
|-----|-----------|-----------|
| `/likit-kapatici` | Kapatıcı Çeşitleri ve Fiyatları | Likit Kapatıcı Çeşitleri ve Fiyatları, Likit Concealer |
| `/goz-farı-paleti` | Göz Farı Modelleri ve Fiyatları | Göz Farı Paleti Fiyatları & Modelleri |
| `/erkek-parfum` | Parfüm Çeşitleri ve Fiyatları | Erkek Parfüm Çeşitleri & Fiyatları |
| `/kose-banyo-dolabi` | Banyo Dolabı Modelleri ve Fiyatları | Köşe Banyo Dolabı Modelleri ve Fiyatları |
| `/cocuk-scooter` | Scooter Çeşitleri ve Fiyatları | Çocuk Scooter Fiyatları & Modelleri |
| `/ahsap-puzzle` | Puzzle Çeşitleri ve Fiyatları | Ahşap Puzzle Çeşitleri ve Fiyatları |

**DataForSEO ile otomatik kontrol:**
Ranked Keywords verisinde, eğer daha kısa/genel bir keyword varyantı (ör. "kapatıcı") ayrı bir URL'de rank alıyorsa, bu URL için o genel keyword'ü seçme — daha spesifik varyantı (ör. "likit kapatıcı") seç.

---

## 3. Description Structure Rules

### Three-Part Formula
Every description follows this structure:

```
[Opening with primary keyword] + [Value proposition / features / variants] + [CTA with brand]
```

**Part 1 — Opening (primary keyword, first 50 chars):**
Integrate the primary keyword into a natural opening sentence. This portion is most likely to appear in SERP even if description gets truncated.

**Part 2 — Middle (differentiator, features, or secondary keyword):**
Add value — mention product variety, key features, secondary keywords, or USPs like "ücretsiz kargo" or "hızlı teslimat" if relevant to the brand.

**Part 3 — Closing CTA (last 30-40 chars):**
End with an appropriate call-to-action that includes the brand name when possible.

### Description Tone by Sector

**E-commerce (Pasaj, n11, Boyner, Sephora, Özdilek):**
- Functional, benefit-driven
- Mention variety ("birçok seçenek", "farklı modeller")
- Price/deal awareness ("uygun fiyat", "avantajlı")
- Action-oriented CTA

**Premium / Manufacturer (VitrA, Doğtaş, Kelebek, Lova):**
- Inspirational, quality-focused
- Design and craftsmanship language ("şık tasarım", "kaliteli yapı")
- Softer CTAs
- Brand confidence signals

**Hospitality (D Maris, Doğuş Hospitality, Coral Travel):**
- Experiential, sensory language
- Emotional triggers ("unutulmaz", "eşsiz", "nefes kesen")
- Ambiance descriptions
- Invitation-style CTAs

**Finance / Insurance (QNB, Anadolu Hayat, Ray Sigorta, EnerjiSA):**
- Trust-building, authoritative
- Clear benefit statements
- Avoid aggressive sales language
- Professional CTAs

**Beauty / Fashion (Flormar, Kiko Milano, Sephora, Dagi, Derimod, Benetton, Vilebrequin, Oxxo):**
- Trend-aware, aspirational
- Seasonal and style-conscious
- Product discovery language
- Mix of functional and emotional

**Technology (Turkcell, Superonline, Thomson, Ecovacs, Bitalih, SPX):**
- Feature-comparison oriented
- Specification mentions where relevant
- Practical benefit language
- Moderate CTAs

---

## 4. Modifier Selection — Volume-First, SERP-Validated

### Core Principle: Modifiers are chosen by DATA, not by default

"Fiyatları", "Modelleri" and "Çeşitleri" are NOT interchangeable defaults. The correct modifier is the one with the **highest search volume** for that specific category. If volume data is unavailable, SERP competitor analysis determines which modifier to use.

### Modifier Selection Decision Tree

```
1. Do we have search volume data? (DataForSEO Keyword API)
   │
   ├─ YES → Compare volumes:
   │   "lavabo fiyatları" = 8.100
   │   "lavabo modelleri" = 12.100
   │   "lavabo çeşitleri" = 2.400
   │   → Winner: "modelleri" (highest volume)
   │   → If space allows, combine: "Lavabo Modelleri ve Fiyatları"
   │     (put higher-volume modifier first)
   │
   └─ NO → Fall back to SERP competitor analysis:
       │
       ├─ Do we have SERP data? (DataForSEO SERP API)
       │   │
       │   ├─ YES → Count modifier frequency in top 10 titles:
       │   │   7/10 use "Modelleri" → use "Modelleri"
       │   │   3/10 use "Fiyatları" → add as secondary if space allows
       │   │   0/10 use "Çeşitleri" → skip
       │   │
       │   │   BUT: If competitors overwhelmingly use one modifier
       │   │   and a DIFFERENT modifier has volume potential,
       │   │   consider using the less-common one for differentiation
       │   │   (only if volume supports it)
       │   │
       │   └─ NO → Use category-appropriate defaults (see table below)
       │
       └─ Neither available → Use the generic defaults below as last resort
```

### When to Override the Default Modifier

The whole point of SERP analysis is to catch cases where the "obvious" modifier is wrong:

**Example 1 — SERP says something different:**
Category: "robot süpürge"
- Default assumption: "Fiyatları" or "Modelleri"
- SERP reality: 6/10 competitors use "Özellikleri" or "Karşılaştırma"
- → Follow SERP, not the default

**Example 2 — A better modifier exists:**
Category: "akıllı saat"
- Volume: "akıllı saat modelleri" = 14.800
- Volume: "akıllı saat fiyatları" = 9.900
- Volume: "en iyi akıllı saat" = 22.100
- → Consider "En İyi Akıllı Saat" if SERP also supports it

**Example 3 — No one uses modifiers:**
Category: "iPhone 16"
- SERP: 8/10 titles are just "iPhone 16 Fiyat" or "iPhone 16 Özellikleri"
- → Don't force "Modelleri ve Fiyatları" pattern; adapt to the SERP

### Last-Resort Defaults (only when no volume or SERP data)

| Category Type | Default Modifier (TR) | Reasoning |
|--------------|----------------------|-----------|
| Standard PLP | `Modelleri ve Fiyatları` | Historically most common combo |
| Sub-category PLP | `Çeşitleri ve Fiyatları` | Variety emphasis for narrower pages |
| Brand-filtered PLP | `Fiyatları & Modelleri` | Price-first for brand shoppers |
| Tech/Electronics PLP | `Fiyatları & Özellikleri` | Spec-comparison intent |
| Accessory/Small items | `Çeşitleri ve Fiyatları` | Variety-first for low-consideration items |

### Modifier Library by Language

**Turkish (TR):**
- `Fiyatları` — price focused
- `Modelleri` — model/variety focused
- `Çeşitleri` — variety/assortment focused
- `Özellikleri` — features/specs focused
- `Markaları` — brand-focused (multi-brand PLPs)
- `Karşılaştırma` — comparison content
- `Seti` / `Setleri` — bundle/set pages
- `Renkleri` / `Tasarımları` — aesthetic-driven categories
- `İncelemesi` / `Yorumları` — review-oriented content

**English (EN):**
- `Prices & Models` / `Models & Prices`
- `Guide` / `Buying Guide` — informational content
- `Reviews` / `Review` — comparison/review pages
- `Best [Category]` — use only if SERP volume supports it
- `Features & Specifications` — for PDP/tech
- `Deals` / `Offers` — promotional context

**German (DE):**
- `Preise & Modelle` / `Modelle & Preise`
- `Kaufberatung` — buying guide
- `im Vergleich` — comparison
- `Übersicht` — overview
- `Test` / `Testbericht` — review/test content

---

## 5. CTA Library by Tone Level

### Aggressive CTAs (e-commerce, marketplace)
Turkish:
- "Hemen satın alın!"
- "Şimdi keşfedin!"
- "Avantajlı fiyatlarla hemen satın alın."
- "Fırsatları kaçırmayın, hemen inceleyin!"
- "Turkcell güvencesiyle hemen satın alın."

English:
- "Shop now!"
- "Discover the best deals today."
- "Buy now with free shipping."

### Moderate CTAs (retail, tech, general)
Turkish:
- "Hemen keşfedin."
- "Fiyatları karşılaştırın, size uygun modeli bulun."
- "Seçenekleri inceleyin."
- "Özdilekteyim'de hemen keşfedin."

English:
- "Explore the collection."
- "Compare models and find yours."
- "Browse options now."

### Soft CTAs (premium, luxury, manufacturer)
Turkish:
- "Keşfedin."
- "İnceleyin."
- "VitrA kalitesiyle tanışın."
- "İlham veren ürünler için tıklayın."

English:
- "Discover our collection."
- "Explore the range."
- "Experience the difference."

### Informational CTAs (corporate, finance, insurance)
Turkish:
- "Detaylı bilgi için tıklayın."
- "Daha fazla bilgi edinin."
- "Başvuru için hemen ziyaret edin."

English:
- "Learn more."
- "Get the details."
- "Find out how."

---

## 6. Anti-Patterns — Things to NEVER Do

### Title Anti-Patterns
1. **No year in titles** — unless explicitly blog content or user-requested. "2026" ages fast and wastes characters.
2. **No .com or .com.tr** — old pattern, wastes characters, unnecessary.
3. **No brand at beginning** — except homepage. Always end with brand.
4. **No duplicate brand** — don't write "Özdilek Ürünleri | Özdilekteyim". Pick one form.
5. **No "En İyi"** — subjective superlative; avoid unless SERP analysis shows 50%+ competitors using it.
6. **No all-caps or excessive punctuation** — "LAVABO FİYATLARI!!!" is never acceptable.
7. **No keyword stuffing** — "Lavabo Lavabo Fiyatları Lavabo Modelleri" is unnatural.
8. **No emoji in titles** — for most brands. Exception: if the brand specifically uses them (rare, case-specific like Pasaj's 🖥️ — only if explicitly in brand profile).
9. **No generic filler** — "Kaliteli ve birbirinden güzel" is old-school and wastes characters.
10. **No mixing languages** — don't put English modifiers in a Turkish title or vice versa.
11. **No repeated "ve"** — title'da "ve" en fazla 1 kez. İkinci bağlaç `&` veya `,` olmalı. "Çeşitleri ve Renkleri, Modelleri ve Fiyatları" → "Çeşitleri & Renkleri, Modelleri ve Fiyatları"
12. **No parent-category keywords on child pages** — alt kategori sayfasında üst kategorinin genel keyword'ünü kullanma. URL'deki niteleyiciyi (likit, ahşap, çocuk vb.) keyword'de koru. `/likit-kapatici` → "Kapatıcı" değil "Likit Kapatıcı" olmalı.

### Description Anti-Patterns
1. **No verbatim title repetition** — description should complement, not repeat the title.
2. **No "ziyaret edin" as sole CTA** — too generic and wastes space; pair with a more specific action.
3. **No generic openings** — avoid "Kaliteli ve birbirinden güzel X çeşitlerini incelemek için..." (template-sounding).
4. **No exceeding 160 characters** — truncated descriptions look unprofessional.
5. **No false claims** — "En ucuz fiyat garantisi" unless the brand actually offers this.
6. **No orphaned brand mentions** — if you mention the brand in description, it should serve the CTA, not just sit there.

---

## 7. Special Cases

### Brand-Filtered Pages (Sub-brand under Marketplace)
When a page shows products of a specific brand within a larger platform:
```
Title: [Sub-brand] [Category] [Modifier] | Platform
Desc: [Sub-brand] [category] modellerini [feature/variety] ile [Platform]'da keşfedin. [CTA]
```
Example:
- Title: `Altus Blender & Mikser Fiyatları, Altus Blender Seti - Pasaj`
- Desc: `Altus blender ve mikserleri hız seçenekleri ve tasarımlarıyla Pasaj'da keşfedin. İhtiyacınız olan Altus blender setini Turkcell güvencesiyle hemen satın alın.`

### Ghost / Attribute-Filter Categories
Pages filtered by color, material, style, or other attributes:
```
Title: [Attribute] [Product] [Modifier], [Full Variant Name] | Brand
Desc: [Emotional hook about attribute] + [product variety] + [CTA with brand]
```
Example:
- Title: `Siyah Lavabo Fiyatları & Modelleri, Siyah Banyo Lavabosu | VitrA`
- Desc: `Banyonuza şıklık katacak modern görünümlü siyah lavabo modelleri VitrA'da. Aradığın siyah banyo lavabosu modelleri uygun fiyatlarla sizlerle.`

### Hospitality / Service Pages
These don't follow e-commerce modifier patterns. Instead:
```
Title: [Descriptive phrase with KW] | Brand
Desc: [Sensory/experiential description] + [what's offered] + [invitation CTA]
```
Example:
- Title: `Marmaris'te Rüya Gibi Düğün Organizasyonları | D Maris Bay`
- Desc: `Masmavi deniz manzarası eşliğinde hayalinizdeki düğünü gerçekleştirin. Profesyonel ekibimiz, özel gününüzün kusursuz olması için yanınızda.`

### Pages Where No Title Change is Needed
If the existing title already perfectly matches all rules, output `-` in the proposed title column and note "Mevcut title uygun, değişiklik önerilmiyor."

### Empty/Thin Content Pages
If a URL returns very thin content or errors:
- Flag as "⚠️ Thin content — title önerisi sayfa içeriğine göre doğrulanmalı"
- Still generate a best-effort title/description based on URL pattern and keyword
