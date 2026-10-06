# M5Stack Tab5 Home Assistant Dashboard + Kamera

ESPHome tabanlı, Home Assistant ile entegre çalışan M5Stack Tab5 dashboard. **Kamera desteği** ile canlı video akışı sağlar.

## Özellikler

- **Dashboard Sekmesi:**
  - Sıcaklık ve nem sensörleri
  - Lamba ve fan kontrolleri
  - WiFi sinyal gücü
  - Cihaz durumu

- **Kamera Sekmesi:**
  - 640x480 çözünürlükte canlı kamerası
  - Tab5 ekranında kamera bilgisi
  - Home Assistant'ta canlı video akışı

- **Çok Sayfalı Arayüz:**
  - Fiziksel butonlarla sekme değiştirme
  - Responsive dashboard kartları
  - Renkli durum göstergeleri

## Kurulum

### 1. ESPHome Dosyalarını Hazırla

```bash
cd esphome/
cp secrets.example.yaml secrets.yaml
# secrets.yaml dosyasını düzenle
```

### 2. Secrets Dosyasını Doldur

```yaml
wifi_ssid: "WIFI_ADINIZ"
wifi_password: "WIFI_ŞİFRENİZ"
```

### 3. ESPHome'a Yükle

```bash
esphome run tab5_dashboard.yaml
```

### 4. Home Assistant'ta Ekle

- Home Assistant → Ayarlar → Cihazlar ve Hizmetler → ESPHome
- Cihazı bul ve ekle

### 5. Lovelace Dashboard'ı Kuruluma Ekle

```yaml
# configuration.yaml
lovelace:
  mode: yaml
```

`lovelace_tab5_dashboard.yaml` dosyasını Home Assistant `config` dizininde kullan veya UI üzerinden manuel olarak kartları ekle.

## Dosya Yapısı

```
tab5_ha_dashboard/
├── esphome/
│   ├── tab5_dashboard.yaml       # Ana ESPHome konfigürasyonu
│   ├── secrets.yaml              # WiFi bilgileri (gitignore'da)
│   └── button_control.yaml       # Fiziksel buton konfigürasyonu
├── homeassistant/
│   └── lovelace_tab5_dashboard.yaml # HA Dashboard ve kamera kartları
├── README.md                     # Bu dosya
└── .gitignore
```

## Kamera Özellikleri

- **Çözünürlük:** 640x480
- **JPEG Kalitesi:** 10 (yüksek kalite)
- **FPS:** ~30 fps
- **Beyaz Balans:** Otomatik
- **Maruz Kalma:** Otomatik
- **Ayna/Çevirme:** Kapalı

## Fiziksel Butonlar

- **Sol Buton (GPIO35):** Önceki sekmesiye git
- **Orta Buton (GPIO37):** Lambaları aç/kapat
- **Sağ Buton (GPIO39):** Sonraki sekmeye git

## Pin Eşlemesi

| Bileşen | GPIO | Açıklama |
|---------|------|----------|
| Ekran CS | GPIO5 | SPI Chip Select |
| Ekran DC | GPIO15 | Data/Command |
| Ekran Reset | GPIO33 | Display Reset |
| Kamera VSYNC | GPIO33 | Görüntü Sync |
| Kamera HREF | GPIO36 | Satır Sync |
| Kamera Clock | GPIO32 | Pixel Clock |
| DHT Sensör | GPIO4 | Sıcaklık/Nem |
| Relay 1 | GPIO27 | Lamba Kontrolü |
| Relay 2 | GPIO26 | Fan Kontrolü |
| Sol Buton | GPIO35 | Sayfa Öncesi |
| Orta Buton | GPIO37 | Relay Toggle |
| Sağ Buton | GPIO39 | Sayfa Sonrası |

## Home Assistant Entitileri

### Sensörler
- `sensor.oda_sicakligi` - Oda sıcaklığı (°C)
- `sensor.oda_nem_orani` - Oda nem oranı (%)
- `sensor.wifi_sinyal_gucu` - WiFi sinyal gücü (dBm)

### Anahtar
- `switch.lamba_1` - Lamba kontrolü
- `switch.fan_1` - Fan kontrolü

### Kamera
- `camera.tab5_kamerasi` - Canlı kamera akışı

### İkili Sensörler
- `binary_sensor.tab5_dashboard_cihaz_durumu` - Cihaz online/offline

## Sorun Giderme

### Kamera Görüntüsü Görünmüyor
- Kamera pinlerini kontrol et
- Power supply yeterli mi kontrol et
- Serial monitor'da hata mesajlarını kontrol et

### Sekme Değiştirme Çalışmıyor
- Buton pinlerini kontrol et
- `button_control.yaml` eklentisini yükle
- Serial debug çıkışını kontrol et

### WiFi Bağlantısı Başarısız
- Şifresi ve SSID'si doğru mu kontrol et
- WiFi sinyali yeterli mi kontrol et

### Dashboard Kartları Görünmüyor
- Home Assistant'ta cihazı ekle
- Lovelace dashboard'ını yenile
- Browser cache'ini temizle

## Notlar

- ST7121 ekran sürücüsü kartın gerçek modeline göre değişebilir. `display:` bölümündeki `model` ve pinler gerçek devre şemasına göre ayarlanmalıdır.
- Kamera I2C pinleri varsayılan olarak GPIO26 (SDA) ve GPIO27 (SCL)'dir. Başka sensörler kullanıyorsan pin çatışmalarını kontrol et.
- Fiziksel buton GPIO pinleri Tab5 kartınıza göre değişebilir.

## Lisans

GPL-3.0

## Katkı

Bug raporları ve öneriler için issue açabilirsiniz.
