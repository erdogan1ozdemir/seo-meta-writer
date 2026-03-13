# SEO Meta Writer — Claude Skill

URL listesi verin, gerisini bu skill halleder: sayfa analizi, keyword keşfi, SERP rakip analizi, title/description üretimi.

DataForSEO MCP entegrasyonuyla çalışır. DataForSEO olmadan da kullanılabilir (temel mod).

---

## Ne İşe Yarar?

Bir sitenin birden fazla sayfası için SEO uyumlu **meta title** ve **meta description** üretir.

Sadece title/description yazmaz — arkada şunları yapar:

- Sayfayı crawl eder, içeriğini analiz eder (H1, H2, body text)
- Sayfanın rank aldığı keyword'leri bulur
- En yüksek hacimli primary keyword'ü seçer
- O keyword için Google SERP'ini çeker, ilk 10 rakibin title/description'larını analiz eder
- Rakiplerin kullandığı modifier'ları, CTA'ları, karakter uzunluklarını çıkarır
- Bu veriye göre title'daki modifier'ı seçer ("Fiyatları" mı, "Modelleri" mi, "Çeşitleri" mi — hangisinin hacmi yüksekse ve SERP'te ne baskınsa)
- Secondary keyword'leri rakip URL'lerden keşfeder
- Batch genelinde cannibalization kontrolü yapar
- Tam kategori listesi verilirse, sadece istenen sayfaları işler ama tüm listeye karşı cannibalization kontrolü uygular
- Sayfanın ne hakkında olduğunu anlayamazsa otomatik fallback zinciri çalıştırır (URL slug → Breadcrumb schema → Product schema → H1 → Title → İçerik)
- Sonucu tablo + Excel olarak verir

**35+ Türk markası** için hazır profiller içerir (VitrA, Pasaj, Özdilek, Flormar, D Maris Bay, Turkcell, QNB, vb.)

**Türkçe, İngilizce, Almanca** destekler — dili otomatik algılar.

---

## Kurulum (3 Adım)

### Adım 1: DataForSEO Hesabı

1. [app.dataforseo.com](https://app.dataforseo.com) → hesap oluştur
2. **API Access** sekmesinden `API login` ve `API password` bilgilerini kopyala
3. Ücretsiz deneme bakiyesi ile test edebilirsin

### Adım 2: DataForSEO MCP'yi Claude Code'a Bağla

Terminal'i aç ve şu komutu çalıştır:

```bash
claude mcp add dfs-mcp \
  --env DATAFORSEO_USERNAME=SENIN_API_LOGIN \
  --env DATAFORSEO_PASSWORD=SENIN_API_PASSWORD \
  -- npx -y dataforseo-mcp-server
```

> `SENIN_API_LOGIN` ve `SENIN_API_PASSWORD` kısımlarını kendi DataForSEO API bilgilerinle değiştir.

Çalıştığını doğrula:

```bash
claude mcp list
# "dfs-mcp" listelenmiş olmalı
```

### Adım 3: Skill'i Kur

```bash
# Repo'yu klonla
git clone https://github.com/erdogan1ozdemir/seo-meta-writer.git

# Global olarak kur (tüm Claude Code projelerinde kullanılabilir)
mkdir -p ~/.claude/skills
cp -r seo-meta-writer/ ~/.claude/skills/seo-meta-writer/
```

### İpucu: İzin Sormadan Çalıştırma

Bu skill çok sayıda API çağrısı, dosya okuma/yazma ve bash komutu çalıştırır. Claude Code her işlem için izin istemesin diye şu yöntemlerden birini kullan:

**Yöntem 1 — Oturum bazlı (önerilen):**
```bash
claude --dangerously-skip-permissions
```
Bu flag ile Claude Code mevcut oturumda hiçbir işlem için izin sormaz.

**Yöntem 2 — Proje bazlı izin listesi:**

Proje klasöründe `.claude/settings.json` dosyası oluştur:
```json
{
  "permissions": {
    "allow": [
      "mcp__dfs-mcp__*",
      "Bash(*)",
      "Read(*)",
      "Write(*)",
      "WebFetch(*)"
    ]
  }
}
```
Bu ayar sadece bu proje içinde geçerli olur ve tüm DataForSEO MCP çağrıları, bash komutları, dosya okuma/yazma ve web fetch işlemlerini otomatik onaylar.

Kurulum tamam. Claude Code'u aç ve kullanmaya başla.

---

## Nasıl Kullanılır?

### Örnek 1 — Keyword'leri otomatik bulsun

```
Bu URL'leri analiz et, keyword'leri sen belirle, title ve description öner.
Brand: VitrA

https://www.vitra.com.tr/c-lavabolar
https://www.vitra.com.tr/c-banyo-aynalari
https://www.vitra.com.tr/c-kuvetler
```

Claude şunları yapar (onay sormadan):
1. DataForSEO ile sayfaları tarar
2. Her sayfanın keyword profilini çıkarır
3. Her keyword için SERP'teki rakip title/description'ları analiz eder
4. Hacim ve SERP verisine göre en uygun modifier'ı seçer
5. Title ve description üretir
6. Cannibalization kontrolü yapar
7. Sonucu tablo olarak verir (10+ sayfa ise otomatik Excel de oluşturur)

### Örnek 2 — Keyword'leri elle ver

```
Şu URL'ler için meta title ve description yaz. Brand: Pasaj

/pasaj/cep-telefonu/android-telefonlar → android telefon
/pasaj/bilgisayar-tablet → bilgisayar tablet
/pasaj/tv-ses-sistemleri/televizyonlar → televizyon fiyatları
```

### Örnek 3 — CSV/Excel yükle

```
[Dosya yükle]
Bu listedeki URL'ler için title/description hazırla. Brand: Özdilekteyim. Çıktıyı Excel ver.
```

### Örnek 4 — İngilizce site

```
Generate meta titles and descriptions for these pages. Brand: D Maris Bay

https://www.dmarisbay.com/en/dining
https://www.dmarisbay.com/en/accommodation
https://www.dmarisbay.com/en/activities
```

### Örnek 5 — Tam liste ver, bir kısmını işle (cannibalization önleme)

Tüm kategori ağacını verip sadece belirli sayfalar için title/description isteyebilirsin. Skill, geri kalan sayfaların keyword'lerini "rezerve" olarak işaretler ve çakışma olmaz.

```
Aşağıdaki tüm kategori listesini ver. Sadece * ile işaretlilerin title/description'ını yaz.
Brand: Flormar

* /ruj
* /likit-ruj
/dudak-kalemi
/dudak-bakimi
* /fondoten
* /likit-fondoten
/pudra
/kapatici
* /likit-kapatici
/goz-fari
/maskara
* /rimel
```

Bu modda:
- `*` olan 6 sayfa → tam analiz + title/description üretimi
- Diğer 6 sayfa → sadece keyword haritası çıkarılır (cannibalization referansı)
- `/likit-kapatici` sayfasına "Kapatıcı" keyword'ü atanmaz çünkü `/kapatici` sayfası zaten var

---

## DataForSEO Olmadan Kullanım

Skill DataForSEO olmadan da çalışır, ama keyword'leri sen vermelisin:

```
Bu URL'ler için meta title ve description yaz.
Brand: Flormar

/ruj → ruj çeşitleri
/fondoten → fondöten fiyatları
/maskara → maskara modelleri
```

Bu modda SERP analizi ve keyword keşfi yapılmaz, ama stil kuralları, brand profili ve cannibalization kontrolü yine aktiftir.

---

## Dosya Yapısı

```
seo-meta-writer/
├── SKILL.md                              # Ana workflow ve kurallar
├── README.md                             # Bu dosya
├── LICENSE                               # MIT License
└── references/
    ├── style-rules.md                    # SEO kuralları, modifier karar ağacı, CTA kütüphanesi
    ├── brand-profiles.md                 # 35+ marka profili
    └── dataforseo-integration.md         # DataForSEO API rehberi ve hata yönetimi
```

## Maliyet

DataForSEO API kullanım maliyeti (skill'in kendisi ücretsiz):

| Sayfa Sayısı | Tahmini Maliyet |
|-------------|----------------|
| 10 sayfa | ~$0.50–0.70 |
| 50 sayfa | ~$2.50–3.50 |
| 100 sayfa | ~$5.00–7.00 |

---

## Claude.ai'de Kullanım (Alternatif)

Claude Code yerine claude.ai'de de kullanabilirsin:

1. Releases sayfasından `seo-meta-writer.skill` dosyasını indir
2. Claude.ai → `Settings > Capabilities` → Code execution açık olmalı
3. `Customize > Skills` → Upload skill → `.skill` dosyasını yükle
4. Toggle'ı aç ve yeni sohbette kullan

> Bu yöntemde DataForSEO MCP entegrasyonu yoktur. Keyword'leri elle vermeniz gerekir.

---

## Katkıda Bulunma

- Yeni marka profili → `references/brand-profiles.md`'ye PR at
- Yeni dil desteği → `references/style-rules.md`'deki modifier/CTA library'ye ekle
- Bug veya öneri → Issue aç

## Lisans

MIT
