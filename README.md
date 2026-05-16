# Rapor Denetim Botu

Telegram üzerinden haftalık faaliyet raporunu adım adım toplayan, **Google Gemini 2.5 Flash** ile her bölümü profesyonel kurumsal dile dönüştüren, yapılandırılmış bir **DOCX belgesi** oluşturup belirli bir **Google Drive klasörüne** yükleyen ve hem dosyayı hem de Drive linkini kullanıcıya Telegram'dan ileten Python botudur. Tüm raporlar ayrıca yerel bir **SQLite veritabanında** arşivlenir.

---

## Özellikler

- Inline klavye butonlarıyla tek tıkla rapor başlatma
- 7 adımlı konuşma akışıyla detaylı veri toplama
- Gemini 2.5 Flash ile JSON tabanlı bölüm bazlı AI düzenlemesi
- Yönetici özeti dahil tam yapılandırılmış DOCX çıktısı
- DOCX'i belirli bir Drive klasörüne yükleme ve herkese açık link oluşturma
- Raporu Telegram üzerinden doğrudan dosya olarak gönderme
- Tüm rapor verilerini SQLite'a kaydetme (arşivleme)
- Cloud deployment için çevre değişkeniyle credentials yönetimi
- Gemini hatasında ham veriyle çalışmaya devam eden fallback mekanizması

---

## Teknoloji Yığını

| Katman | Araç |
|--------|------|
| Bot framework | python-telegram-bot 20.x |
| Yapay zeka | Google Gemini 2.5 Flash |
| Belge oluşturma | python-docx |
| Bulut depolama | Google Drive API v3 |
| Kimlik doğrulama | OAuth 2.0 — google-auth-oauthlib |
| Veritabanı | SQLite3 (yerel) |

---

## Proje Yapısı

```
Rapordenetimbotu/
├── rapordenetim_v3.py   # Tüm bot mantığı tek dosyada
├── requirements.txt     # Python bağımlılıkları
├── .env.example         # Ortam değişkeni şablonu
├── .gitignore
├── credentials.json     # Google OAuth kimlik bilgisi (siz oluşturursunuz, commit edilmez)
├── token.json           # OAuth oturum tokeni (otomatik oluşturulur, commit edilmez)
└── raporlar.db          # SQLite arşivi (otomatik oluşturulur, commit edilmez)
```

---

## Gereksinimler

- Python 3.10 veya üzeri
- Telegram Bot Token
- Google Gemini API anahtarı
- Google Drive API için `credentials.json`

---

## Kurulum

### 1. Depoyu klonla

```bash
git clone https://github.com/hamzamalikyildiz/Rapordenetimbotu.git
cd Rapordenetimbotu
```

### 2. Sanal ortam oluştur ve etkinleştir

```bash
python -m venv .venv

# Linux / macOS
source .venv/bin/activate

# Windows
.venv\Scripts\activate
```

### 3. Bağımlılıkları kur

```bash
pip install -r requirements.txt
```

---

## Yapılandırma

### .env Dosyası

`.env.example` dosyasını kopyalayıp `.env` olarak düzenle:

```bash
cp .env.example .env
```

```env
TELEGRAM_TOKEN=buraya_telegram_bot_tokenini_yaz
GOOGLE_API_KEY=buraya_gemini_api_keyini_yaz
```

| Değişken | Açıklama | Nereden Alınır |
|----------|----------|----------------|
| `TELEGRAM_TOKEN` | Telegram bot kimlik bilgisi | @BotFather → `/newbot` |
| `GOOGLE_API_KEY` | Gemini API anahtarı | [Google AI Studio](https://aistudio.google.com/apikey) |

---

### Google Drive Bağlantısı

#### Adım adım kurulum

1. [Google Cloud Console](https://console.cloud.google.com/) adresine git
2. Yeni bir proje oluştur (ya da mevcut projeyi seç)
3. **APIs & Services → Library** bölümünden **Google Drive API**'yi etkinleştir
4. **APIs & Services → Credentials** bölümüne git
5. **+ Create Credentials → OAuth client ID** seçeneğini tıkla
6. Uygulama türü olarak **Desktop app** seç ve oluştur
7. İndirilen JSON dosyasını `credentials.json` adıyla proje kök dizinine koy

#### İlk çalıştırmada

Bot ilk kez çalıştırıldığında tarayıcı otomatik açılır. Google hesabınla giriş yap ve Drive iznini ver. Onayın ardından `token.json` otomatik oluşturulur — sonraki çalıştırmalarda tekrar giriş gerekmez.

#### Drive klasörü

Bot yüklediği dosyaları kodda tanımlı olan `DRIVE_KLASOR_ID` değişkenindeki klasöre kaydeder. Farklı bir klasör kullanmak istiyorsan bu değeri değiştir:

```python
DRIVE_KLASOR_ID = "buraya_drive_klasor_id"  # rapordenetim_v3.py içinde
```

Drive klasörü ID'sini Drive URL'sinden alabilirsin:
`https://drive.google.com/drive/folders/` **bu-kisim-klasor-id**'dir

---

### Cloud Deployment (Opsiyonel)

Railway, Render veya benzeri platformlarda `credentials.json` ve `token.json` dosyası yerine çevre değişkeni kullanabilirsin:

```env
CREDENTIALS_JSON={"installed":{"client_id":"...","client_secret":"...",...}}
TOKEN_JSON={"token":"...","refresh_token":"...",...}
```

Bot başlarken bu değişkenler varsa dosyaları otomatik oluşturur.

---

## Botu Çalıştırma

```bash
python rapordenetim_v3.py
```

Konsol çıktısı:

```
Bot calisiyor...
```

Telegram'ı aç ve botuna `/start` yaz.

---

## Bot Konuşma Akışı

```
Kullanıcı  →  /start
Bot        →  [Inline klavye: "Yeni Rapor Ekle" | "İptal Et"]

Kullanıcı  →  Yeni Rapor Ekle (butona tıklar)
Bot        →  Rapor hangi tarihleri kapsıyor? (Ör: 20-27 Kasım 2025)

Kullanıcı  →  12-16 Mayıs 2025
Bot        →  1. Bu hafta hangi PROJELER üzerinde çalıştın?

Kullanıcı  →  Mobil uygulama API entegrasyonu
Bot        →  2. TAMAMLANAN görevlerin neler?

Kullanıcı  →  Login ekranı tamamlandı, birim testler yazıldı
Bot        →  3. Şu an DEVAM EDEN işler neler?

Kullanıcı  →  Dashboard geliştirmesi sürüyor
Bot        →  4. Karşılaştığın SORUNLAR veya engeller var mı?

Kullanıcı  →  Token yenileme sorunu çözüldü
Bot        →  5. GELECEK HAFTA için planların neler?

Kullanıcı  →  Bildirim sistemi entegrasyonu
Bot        →  6. Eklemek istediğin NOTLAR var mı? (Yoksa 'Yok' yazabilirsin)

Kullanıcı  →  Yok
Bot        →  Veriler alındı. AI her başlığı tek tek düzenliyor, rapor hazırlanıyor...
              [DOCX dosyası Telegram'dan gönderilir]
              [Drive linki mesaj olarak iletilir]
```

---

## Oluşturulan DOCX Yapısı

```
HAFTALIK FAALİYET RAPORU

Rapor Sahibi: [Ad]
Tarih Aralığı: [Tarih]

YÖNETİCİ ÖZETİ
  Kısa ve öz AI özeti...

PROJELER
  • Düzenlenmiş metin...

TAMAMLANAN GÖREVLER
  • Düzenlenmiş metin...

DEVAM EDEN İŞLER
  • Düzenlenmiş metin...

SORUNLAR VE ENGELLER
  • Düzenlenmiş metin...

GELECEK HAFTA PLANI
  • Düzenlenmiş metin...

EK NOTLAR
  • Düzenlenmiş metin...
```

Dosya adı: `Rapor_{KullanıcıAdı}_{Tarih}.docx`

---

## Komutlar

| Komut | Açıklama |
|-------|----------|
| `/start` | Ana menüyü açar |
| `/yardim` | Kullanım kılavuzunu gösterir |
| `/iptal` | Devam eden rapor sürecini iptal eder |

---

## Veritabanı

Her tamamlanan rapor `raporlar.db` dosyasına kaydedilir:

| Sütun | Açıklama |
|-------|----------|
| `id` | Otomatik artan birincil anahtar |
| `user_id` | Telegram kullanıcı ID'si |
| `tarih_araligi` | Raporun kapsadığı tarih aralığı |
| `projeler` | Ham proje girişi |
| `tamamlanan_gorevler` | Ham tamamlanan görev girişi |
| `devam_eden_gorevler` | Ham devam eden iş girişi |
| `sorunlar` | Ham sorun/engel girişi |
| `gelecek_planlar` | Ham gelecek hafta planı |
| `ek_notlar` | Ham ek not girişi |

---

## Güvenlik

Aşağıdaki dosyalar `.gitignore` kapsamındadır, asla commit edilmez:

| Dosya | İçerik |
|-------|--------|
| `.env` | API anahtarları ve bot token |
| `credentials.json` | Google OAuth istemci kimlik bilgisi |
| `token.json` | Kullanıcı oturum tokeni |
| `raporlar.db` | Kişisel rapor arşivi |

Bu dosyalardan birini yanlışlıkla commit ettiysen git geçmişini temizle ve ilgili token'ı Google Cloud Console'dan hemen iptal et.

---

## Sorun Giderme

**`Bot calisiyor...` çıktısından sonra Telegram'da yanıt yok**
→ `TELEGRAM_TOKEN` değerinin doğru olduğunu kontrol et.

**`credentials.json bulunamadı` hatası**
→ Google Cloud Console'dan indirdiğin JSON dosyasını proje kök dizinine `credentials.json` adıyla koy.

**`token.json` geçersiz / süresi dolmuş**
→ `token.json` dosyasını sil, botu yeniden başlat ve tarayıcıda tekrar giriş yap.

**Gemini düzenlemesi başarısız, ham veri geldi**
→ `GOOGLE_API_KEY` değerini kontrol et. Bot fallback olarak ham veriyle DOCX oluşturmaya devam eder.

**Drive'a yükleme `None` döndürüyor**
→ `credentials.json` ve `token.json` dosyalarını kontrol et. Google Drive API'nin Cloud Console'da etkin olduğundan emin ol.

---

## Geliştirici

**hamzamalikyildiz** — [github.com/hamzamalikyildiz](https://github.com/hamzamalikyildiz)
