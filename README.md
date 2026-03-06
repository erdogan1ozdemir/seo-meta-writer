# SEO Meta Writer — Claude Skill

Birden fazla web sayfası için **veri odaklı SEO meta title ve description** üreten Claude skill'i.

Sadece URL listesi verin — skill sayfayı analiz eder, keyword belirler, SERP rakiplerini inceler ve optimum title/description önerileri üretir.

## Ne Yapar?

```
URL Listesi
  ├─ Sayfa içerik analizi (H1, H2, body text)
  ├─ Primary keyword keşfi (search volume bazlı)
  ├─ SERP rakip title/description analizi
  ├─ Secondary keyword tespiti
  ├─ Modifier seçimi (hacim + SERP verisi ile)
  ├─ Cannibalization kontrolü (batch geneli)
  └─ Title & Description üretimi
      → Excel + Chat çıktısı
```

### Temel Özellikler

- **Volume-first modifier seçimi** — "Fiyatları", "Modelleri", "Çeşitleri" arasında en yüksek hacimli olanı seçer, SERP verisiyle doğrular
- **SERP rakip analizi** — Top 10 organik sonucun title/description pattern'lerini analiz eder (modifier frekansı, CTA dağılımı, karakter uzunlukları, differentiator gap'ler)
- **Otomatik sayfa tipi tespiti** — URL pattern'inden PLP, PDP, Blog, Homepage, Service sayfası ayrımı yapar
- **Cannibalization kontrolü** — Aynı batch'teki URL'ler arası keyword çakışmalarını tespit eder
- **Çoklu dil desteği** — Türkçe, İngilizce, Almanca (otomatik algılama veya kullanıcı tercihi)
- **35+ marka profili** — Hazır brand konfigürasyonları (separator, ton, sektör, özel kurallar)
- **DataForSEO ile veya DataForSEO olmadan çalışır** — MCP bağlıysa zengin analiz, yoksa temel üretim

---

## Kurulum

### Ön Gereksinimler

| Gereksinim | Zorunlu mu? | Açıklama |
|-----------|------------|----------|
| Claude Pro/Max/Team hesabı | ✅ Evet | Skill yüklemek için ücretli plan gerekli |
| Code Execution açık | ✅ Evet | Settings > Capabilities'den aktif olmalı |
| DataForSEO hesabı | ⭐ Önerilen | Keyword discovery ve SERP analizi için (opsiyonel) |
| Claude Code | ❌ Hayır | İsteğe bağlı, DataForSEO MCP için avantajlı |

### Yol 1: Claude.ai'de Kullanım (Kolay Yol)

**1. Skill dosyasını indir**

Releases sayfasından en güncel `seo-meta-writer.skill` dosyasını indir.

Ya da repo'yu klonla ve klasörü zip'le:

```bash
git clone https://github.com/KULLANICI_ADIN/seo-meta-writer.git
cd seo-meta-writer
# Klasörü zip'leyin (SKILL.md root'ta olmalı)
zip -r seo-meta-writer.skill SKILL.md references/
```

**2. Claude.ai'de yükle**

1. `Settings > Capabilities` → **Code execution and file creation** açık olmalı
2. `Customize > Skills` → **Upload skill** (veya "Add personal skill")
3. `seo-meta-writer.skill` dosyasını yükle
4. Skill listesinde görünen toggle'ı **ON** yap

**3. Kullanmaya başla**

Yeni sohbet aç ve doğal dille iste:
```
Bu URL'ler için meta title ve description hazırla, brand: VitrA

/c-lavabolar → lavabo modelleri
/c-banyo-dolaplari → banyo dolabı
/c-kuvetler → küvet modelleri
```

> ⚠️ Bu yöntemde DataForSEO entegrasyonu yoktur. Skill, kullanıcının sağladığı keyword'lerle çalışır.

---

### Yol 2: Claude Code + DataForSEO MCP (Tam Güç)

Bu yöntem skill'in tüm özelliklerini açar: otomatik keyword keşfi, SERP rakip analizi, search volume doğrulama.

**1. DataForSEO hesabı oluştur**

- [app.dataforseo.com](https://app.dataforseo.com) adresinden hesap aç
- API Access sekmesinden **API login** ve **API password** bilgilerini al
- Ücretsiz deneme bakiyesi ile test edebilirsin

**2. DataForSEO MCP'yi Claude Code'a ekle**

En kolay yol — remote server olarak ekle:

```bash
# Base64 token oluştur (login:password formatında)
echo -n "API_LOGIN:API_PASSWORD" | base64
# Çıkan token'ı kopyala

# MCP server ekle
claude mcp add \
  --header "Authorization: Basic BURAYA_BASE64_TOKEN" \
  --transport http \
  dfs-mcp https://mcp.dataforseo.com/http
```

Alternatif — NPM üzerinden lokal kurulum:

```bash
claude mcp add dfs-mcp \
  --env DATAFORSEO_USERNAME=API_LOGIN \
  --env DATAFORSEO_PASSWORD=API_PASSWORD \
  -- npx -y dataforseo-mcp-server
```

**Sadece gerekli modülleri aktif etmek için** (context window tasarrufu):

```bash
claude mcp add dfs-mcp \
  --env DATAFORSEO_USERNAME=API_LOGIN \
  --env DATAFORSEO_PASSWORD=API_PASSWORD \
  --env ENABLED_MODULES=SERP,KEYWORDS_DATA,ONPAGE,DATAFORSEO_LABS \
  -- npx -y dataforseo-mcp-server
```

**3. MCP'nin çalıştığını doğrula**

```bash
claude mcp list
# dataforseo-mcp listelenmiş olmalı
```

**4. Skill'i Claude Code'a ekle**

```bash
# Repo'yu klonla
git clone https://github.com/KULLANICI_ADIN/seo-meta-writer.git

# Skill'i kur (proje bazlı)
mkdir -p .claude/skills
cp -r seo-meta-writer/ .claude/skills/seo-meta-writer/

# Veya global kur (tüm projelerde kullanılabilir)
mkdir -p ~/.claude/skills
cp -r seo-meta-writer/ ~/.claude/skills/seo-meta-writer/
```

**5. Kullanmaya başla**

```bash
claude
# Sohbet başladığında:
```

```
Bu URL'leri analiz et, keyword'leri sen belirle ve meta title/description öner.
Brand: VitrA

https://www.vitra.com.tr/c-lavabolar
https://www.vitra.com.tr/c-banyo-aynalari
https://www.vitra.com.tr/c-kuvetler
```

Claude otomatik olarak:
1. DataForSEO On-Page API ile sayfaları tarar
2. DataForSEO Labs ile keyword profilini çıkarır
3. SERP API ile rakip title/description'ları analiz eder
4. Hacim bazlı modifier seçimi yapar
5. Cannibalization kontrolü uygular
6. Title & description üretir

---

## Kullanım Örnekleri

### Temel kullanım — keyword'ler elle verilir
```
Şu URL'ler için Pasaj formatında meta title ve description yaz:

https://www.turkcell.com.tr/pasaj/cep-telefonu/android-telefonlar → android telefon
https://www.turkcell.com.tr/pasaj/bilgisayar-tablet → bilgisayar tablet
https://www.turkcell.com.tr/pasaj/tv-ses-sistemleri → televizyon
```

### Gelişmiş kullanım — DataForSEO ile tam analiz
```
Bu 5 URL'yi analiz et. Keyword'leri DataForSEO ile belirle, 
SERP rakiplerini incele ve buna göre title/description öner.
Brand: Özdilekteyim

https://www.ozdilekteyim.com/magaza/oyuncaklar
https://www.ozdilekteyim.com/magaza/aydinlatma
https://www.ozdilekteyim.com/magaza/bahce-urunleri
https://www.ozdilekteyim.com/magaza/masa-sandalyeler
https://www.ozdilekteyim.com/magaza/nevresim-takimlari
```

### Dosya upload ile toplu işlem
```
[CSV dosyası yükle — URL ve keyword kolonları ile]
Bu listedeki tüm URL'ler için meta title ve description hazırla.
Brand: Flormar
Çıktıyı Excel olarak ver.
```

### İngilizce site
```
Generate meta titles and descriptions for these D Maris Bay pages:

https://www.dmarisbay.com/en/dining
https://www.dmarisbay.com/en/accommodation
https://www.dmarisbay.com/en/activities
https://www.dmarisbay.com/en/beaches-and-pools
```

---

## Dosya Yapısı

```
seo-meta-writer/
├── SKILL.md                              # Ana skill dosyası — workflow ve kurallar
└── references/
    ├── style-rules.md                    # SEO kuralları, modifier karar ağacı, CTA kütüphanesi
    ├── brand-profiles.md                 # 35+ marka profili (ton, sektör, özel notlar)
    └── dataforseo-integration.md         # DataForSEO API endpoint'leri ve kullanım rehberi
```

## DataForSEO Maliyet Tahmini

| Batch Boyutu | Tahmini Maliyet |
|-------------|----------------|
| 10 sayfa | ~$0.50–0.70 |
| 50 sayfa | ~$2.50–3.50 |
| 100 sayfa | ~$5.00–7.00 |
| 200 sayfa | ~$10.00–14.00 |

DataForSEO olmadan skill ücretsiz çalışır, sadece kullanıcının keyword sağlaması gerekir.

---

## Katkıda Bulunma

- Yeni marka profili eklemek isterseniz `references/brand-profiles.md`'ye PR atabilirsiniz
- Yeni dil desteği (Fransızca, İspanyolca vb.) için `references/style-rules.md`'deki modifier library'ye ekleme yapabilirsiniz
- Bug veya öneriler için Issue açabilirsiniz

## Lisans

MIT License — detaylar için [LICENSE](LICENSE) dosyasına bakın.
