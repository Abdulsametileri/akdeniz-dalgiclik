# Akdeniz Dalgıçlık — Proje Notları (CLAUDE.md)

Sanayi dalgıcı / sualtı bakım-onarım işi için **kurumsal tanıtım sitesi**. Amaç: QR
kartvizitten gelen kullanıcı 5–10 saniyede güven duysun ve WhatsApp/telefon ile iletişime geçsin.
Yayın: https://akdenizdalgiclik.com/ (Cloudflare Pages).

## Mimari kararı
- **Sıfırdan hafif custom Hugo** — tema YOK, `layouts/` + `partials/` ile elde örülmüş.
- **Sıfır JavaScript**: menü (CSS checkbox hack), galeri lightbox (`:target`), video (native
  `<video preload="none">`). Performans ve QR hızlı açılış için bilinçli tercih.
- Tek CSS dosyası: `assets/css/main.css` → build’de minify + fingerprint + SRI.
- Görseller Hugo asset pipeline ile **responsive WebP**’ye dönüşür (lazy + width/height → CLS yok).

## Komutlar
```bash
hugo server          # http://localhost:1313 canlı önizleme
hugo --gc --minify   # production build → public/
```
Hugo **extended** gerekir (WebP için). Geliştirildiği sürüm: v0.163.3.

## Cloudflare dağıtım (Workers Builds)
Proje **Cloudflare Workers** olarak kurulu (Pages değil) — assets-only Worker, `public/`'i
statik servis eder. Bağlantı: `wrangler.toml` (repo kökünde, `name` Worker adıyla aynı).
| Ayar (dashboard → Settings → Builds) | Değer |
|---|---|
| Build command | `hugo` |
| Deploy command | `npx wrangler deploy` ⚠️ (`hugo` OLMAZ — boş deploy → "Hello World" placeholder) |
| Root directory | `/` |
| Env var | `HUGO_VERSION = 0.163.3` |
| Branch | `main` |

> ⚠️ Deploy command `hugo` ise hiçbir şey yüklenmez, alan adı Worker'ın varsayılan
> "Hello World" çıktısını gösterir. `npx wrangler deploy` + `wrangler.toml` şart.

## İçerik nasıl düzenlenir (kod değil, veri)
İçeriğin neredeyse tamamı veri/parametre dosyalarındadır — layout’a dokunmadan düzenlenir:
- `hugo.toml` → `[params]`: **iletişim bilgileri (TEK KAYNAK)**, marka, bölge, OG görseli, menü.
- `data/services.yaml` → 6 hizmet (ana sayfa kartları + Hizmetler sayfası detayları).
- `data/gallery.yaml` → galeri foto’ları (dosya, alt, başlık, açıklama, opsiyonel `objpos`).
- `data/videos.yaml` → saha videoları (src, poster, w/h, başlık, açıklama).
- `data/trust.yaml` → “Neden biz?” güven unsurları.
- `content/*/_index.md` → sayfa başlığı, SEO `description`, `lead`, opsiyonel `ogImage`.

## ⚠️ BEKLEYEN PLACEHOLDER’LAR — `hugo.toml [params]`
Gerçek değerler gelince **sadece burayı** güncelle (tüm site otomatik günceller):
- `phoneDisplay` / `phoneDial` — gerçek telefon
- `whatsappNumber` — `905XXXXXXXXX` (ülke kodlu, **+ ve boşluk yok**)
- `email` — gerçek e-posta
- `region` — şu an "Antalya ve Akdeniz Bölgesi"

> Not: referans sitedeki numara (+905395594821) **başka kişiye** (Muhammed Gazi Vural) ait,
> kullanılmadı. Şu an CTA’lar placeholder’a gider.

## Medya kuralları & kararlar
- Kaynak medya `docs/` içinde ve **`.gitignore`’da** (repoya girmez). `prompt.txt`, `public/`,
  `resources/` de ignore.
- Dosya adları: Türkçe karakter/boşluk yok, küçük harf, tire ile (`iskele-kazik-sualti-01.jpg`).
- Yeni foto: `assets/images/`’e koy (max ~2400px) + `data/gallery.yaml`’a satır ekle.
- **Videolar:** sadece küçük H.264 MP4’ler self-host (`static/videos/`). Büyük dosyalar
  (`kum-basik.mov` 160MB/HEVC, `kaynak-kesim-uzun.mov` 256MB) **dış barındırma** bekliyor →
  Cloudflare Stream / YouTube (liste dışı) / Vimeo; link gelince `data/videos.yaml`’a embed.
- `kesim-video-o2.MP4` **çıkarıldı**: üzerinde “Dive+ @Fatih” filigranı var (başka uygulama/
  kullanıcı adı). Temiz versiyon gelirse eklenir.
- Video poster’ları `qlmanage` ile üretildi (ffmpeg kurulu değil).

## İçerik & dil kuralları (ÖNEMLİ)
- Türkçe, profesyonel, abartısız, güven veren. “Her iş / her derinlik / en iyi” gibi iddialar YOK.
- **UYDURMA:** sertifika, belge, yıl/tecrübe sayısı, referans müşteri adı, dalış derinliği, lisans,
  garanti. Bilinmiyorsa placeholder bırak.
- Görselde ne olduğu kesin değilse genel/güvenli ifade kullan (“liman bölgesi saha çalışması”).
- Anahtar kelime istifleme yok; "sualtı" formu tercih edilir (tutarlılık).

## SEO / erişilebilirlik konvansiyonları
- Her sayfada **tek `<h1>`**; benzersiz title; meta description ≤160 karakter.
- canonical + OG (sayfa bazlı `ogImage`) + Twitter card + JSON-LD (ProfessionalService) + robots.txt + sitemap.
- `lang="tr-TR"`, tüm `<img>`’lerde anlamlı alt.
- Renkler WCAG AA: WhatsApp butonu `--wa:#157F3C`, açık zemin üst-etiket `--eyebrow:#C2410C`.
  Yeni renkli metin eklerken beyaz/açık zeminde **≥4.5:1** kontrastı koru.
- Klavye `:focus-visible` halkaları tanımlı.

## Renk paleti
Navy `#071827` · Deep `#0B3A53` · Sea `#0EA5E9` · Safety Orange `#F97316` (koyu metin `#C2410C`) ·
Light `#F8FAFC` · WhatsApp aksiyon `#157F3C`.

## Test ipucu
Mobil screenshot için: `chrome --headless --window-size=390` **çalışmaz** (headless min ~500px
genişlik dayatır, görüntüyü kırpar). Gerçek mobil için CDP `Emulation.setDeviceMetricsOverride`
kullan.
