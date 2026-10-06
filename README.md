# M5Stack Tab5 Home Assistant Dashboard + Kamera (ST7121 + LVGL)

Bu proje, M5Stack Tab5 cihazı için ESPHome tabanlı, Home Assistant ile entegre çalışan ve ST7121 ekran kullanan bir arayüzdür. Ana hedef, cihazın kendi ekranında bir kontrol paneli göstermek ve aynı anda Home Assistant üzerinde canlı kamera akışı sunmaktır.

## Özellikler

- ST7121 ekran desteği (MIPI DSI tabanlı)
- ST7123 dokunmatik destek
- Home Assistant ile entegrasyon
- Canlı kamera akışı
- Dashboard + Kamera sekmeleri
- Oda sıcaklığı, nem, WiFi sinyali ve cihaz durumu
- Lamba ve fan kontrolü
- LVGL tabanlı modern arayüz

## Kullanılan Temel Yapı

- Ekran: ST7121
- Dokunmatik: ST7123
- Çözünürlük: 1280x720 (MIPI DSI)
- Kamera: ESPHome `esp32_camera`
- Arayüz: LVGL
- Yönetim: Home Assistant + ESPHome

## Dizin Yapısı

```text
.
├── esphome/
│   ├── tab5_dashboard_lvgl.yaml
│   └── secrets.yaml
├── homeassistant/
│   └── lovelace_tab5_dashboard.yaml
├── README.md
├── .gitignore
└── LICENSE
```

## Gereksinimler

- M5Stack Tab5 cihazı
- ESPHome kurulumu
- Home Assistant kurulumu
- WiFi erişimi
- USB / serial bağlantı
- Home Assistant üzerinde ESPHome eklentisi

## Hazırlık

### 1) Repo'yu klonlayın

```bash
git clone https://github.com/husyildiz/tab5_ha_dashboard.git
cd tab5_ha_dashboard
```

### 2) ESPHome dosyasını inceleyin

Ana ESPHome yapılandırması şu dosyadadır:

```text
esphome/tab5_dashboard_lvgl.yaml
```

Bu dosyada:
- ekran modeli `M5STACK-TAB5-ST7121`
- dokunmatik `st7123`
- kamera ve WiFi ayarları
- dashboard ve kamera ekranları
bulunur.

### 3) WiFi bilgilerini girin

`esphome/secrets.yaml` dosyasını oluşturun veya düzenleyin:

```yaml
wifi_ssid: "WIFI_ADINIZ"
wifi_password: "WIFI_ŞİFRENİZ"
api_key: "API_ANAHTARINIZ"
ota_password: "OTA_ŞİFRENİZ"
```

Not: `api_key` ve `ota_password` boş bırakmayın. ESPHome için gerekli olabilir.

## ESPHome ile Yükleme

### 1) ESPHome Add-on'u kurun

Home Assistant içinde:
- Ayarlar
- Eklentiler
- ESPHome
- Kurulum

### 2) Cihazı ekleyin

- ESPHome tarafında "Yeni cihaz" oluşturun
- ESPHome cihazınıza gerekli serial bağlantıyı kurun
- `esphome/tab5_dashboard_lvgl.yaml` dosyasını yükleyin

### 3) Derleyip yükleyin

```bash
cd esphome
esphome run tab5_dashboard_lvgl.yaml
```

Alternatif olarak Home Assistant içindeki ESPHome arayüzünden "Compile" ve "Upload" işlemlerini yapabilirsiniz.

## Home Assistant Dashboard Kurulumu

`homeassistant/lovelace_tab5_dashboard.yaml` dosyasındaki kartları kullanın.

Bu dosya, aşağıdaki iki sekme içerir:
- Ana Dashboard
- Kamera

### Lovelace yükleme adımları

1. Home Assistant içinde sol menüden "Görünüm (Dashboard)" seçin
2. Sağ üstte üç noktaya tıklayıp "Dashboard'ı düzenle"
3. "YAML düzeni" aktif edin
4. Dosyadaki içeriği kopyalayıp yapıştırın

Veya `configuration.yaml` içinde `lovelace:` bloklarını ya da ayrı dashboard dosyasını kullanabilirsiniz.

## Çalışan UI Mantığı

### Dashboard ekranı
- Oda sıcaklığı
- Oda nem oranı
- WiFi sinyal gücü
- Lamba durumu
- Fan durumu
- Cihaz durumu

### Kamera ekranı
- Canlı kamera görünümü
- Kamera bilgisi ve ekran durumu

## ST7121 ve ST7123 Notu

Bu cihaz için ekran ve dokunmatik bilgileri, üretici kartına göre değişebilir. Bu nedenle aşağıdakiler mutlaka kontrol edilmelidir:

- gerçek ekran controller modeli
- SPI / MIPI DSI pinleri
- dokunmatik entegre chipi
- pin eşlemeleri

Bu proje, Axellum'un M5 Tab5 ESPHome LVGL örneğine yakın bir yapı kurar ve ST7121/ST7123 uyumlu şekilde düşünülmüştür. Fakat gerçek kart revision'ına göre küçük pin veya model farklılıkları ortaya çıkabilir.

## Sorun Giderme

### 1) Ekran görünmüyor
- `display` bloğundaki model ve pinleri kontrol edin
- ST7121 için `mipi_dsi` yaklaşımı doğru mu kontrol edin
- reset ve clock pinlerini doğrulayın

### 2) Dokunmatik çalışmıyor
- `touchscreen` blokunda `st7123` kullanımını doğrulayın
- interrupt ve reset pinlerini kontrol edin
- I2C bus bağlantısını kontrol edin

### 3) Kamera yok
- `esp32_camera` pinlerini doğrulayın
- I2C pinleri doğru mu kontrol edin
- `camera.tab5_kamerasi` entity'sini Home Assistant'ta arayın

### 4) WiFi bağlanmıyor
- `secrets.yaml` dosyasındaki SSID ve şifre doğru olmalı
- cihazın WiFi sinyali yeterli olmalı
- ESPHome loglarını kontrol edin

## Gelişmiş Notlar

- Bu proje, referans olarak ST7121 + ST7123 + MIPI DSI + LVGL yaklaşımını kullanır.
- Gerçek cihaz üzerinde test sırasında küçük ayarlamalar gerekebilir.
- Eğer farklı Tab5 revision'ı kullanıyorsanız, ekran modelini ve GPIO pin eşlemelerini doğrulamanız gerekir.

## Lisans

GPL-3.0

## Katkıda Bulunma

- Kodları düzenleyebilirsiniz
- Hata raporları açabilirsiniz
- Yeni widgetler veya menüler ekleyebilirsiniz

## Son Söz

Bu proje, M5Stack Tab5 cihazında ST7121 ekran kullanımına uygun modern bir ESPHome dashboard örneğidir. Ekran, touch, kamera ve Home Assistant entegrasyonu birlikte düşünülerek hazırlanmıştır.
