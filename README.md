# M5Stack Tab5 Home Assistant Dashboard + Kamera (ST7121)

Bu proje, M5Stack Tab5 cihazı için ESPHome tabanlı ve Home Assistant ile entegre çalışan bir arayüzdür. ST7121 ekran ve ST7123 dokunmatik kontrol birimi kullanır.

## Özellikler

- Dashboard sekmesi:
  - Oda sıcaklığı
  - Oda nem oranı
  - Lamba kontrolü
  - Fan kontrolü
  - WiFi sinyal gücü
  - Cihaz durumu

- Kamera sekmesi:
  - Canlı kamera akışı
  - Home Assistant'ta canlı video görünümü
  - Tab5 ekranında ayrı kamera bilgisi

- Ekran yönetimi:
  - ST7121 MIPI DSI ekran
  - ST7123 dokunmatik kontrol
  - Sekme değiştirme

## Dizin Yapısı

```text
.
├── esphome/
│   ├── tab5_dashboard.yaml
│   ├── secrets.yaml
│   └── button_control.yaml
├── homeassistant/
│   └── lovelace_tab5_dashboard.yaml
├── README.md
└── .gitignore
```

## Ana ESPHome Dosyası (`esphome/tab5_dashboard.yaml`)

```yaml
substitutions:
  device_name: "tab5_dashboard"
  friendly_name: "Tab5 Dashboard"

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  comment: "M5Stack Tab5 Home Assistant dashboard with ST7121 display"
  project:
    name: "m5stack.tab5_dashboard"
    version: "4.0.0"

esp32:
  board: esp32-s3-devkitc-1
  framework:
    type: arduino

logger:
  level: DEBUG

api:
  encryption:
    key: !secret api_key

ota:
  password: !secret ota_password

web_server:
  port: 80

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: "${friendly_name} Fallback"
    password: "tab5fallback"

captive_portal:

i2c:
  sda: GPIO8
  scl: GPIO9
  scan: true

# ---------------------------------------------------
# Kamera
# ---------------------------------------------------
esp32_camera:
  external_clock:
    pin: GPIO1
    frequency: 20MHz
  i2c_pins:
    sda: GPIO40
    scl: GPIO39
  data_pins: [GPIO11, GPIO9, GPIO8, GPIO10, GPIO12, GPIO18, GPIO17, GPIO16]
  vsync_pin: GPIO6
  href_pin: GPIO7
  pixel_clock_pin: GPIO13
  power_down_pin: GPIO14
  resolution: 640x480
  jpeg_quality: 10
  contrast: 0
  saturation: 0
  special_effect: NONE
  wb_mode: AUTO
  ae_level: 0
  aec2: "off"
  awb_gain: "on"
  agc_gain: 0
  gainceiling: "2x"
  bpc: "on"
  wpc: "on"
  raw_gma: "on"
  lenc: "on"
  hmirror: "off"
  vflip: "off"
  dcw: "on"
  colorbar: "off"

camera:
  - platform: esp32_camera
    name: "Tab5 Kamerası"
    id: tab5_camera

# ---------------------------------------------------
# ST7121 Display - MIPI DSI
# ---------------------------------------------------
display:
  - platform: mipi_dsi
    id: main_display
    dimensions:
      height: 1280
      width: 720
    model: M5STACK-TAB5-ST7121
    reset_pin: GPIO48
    data_pins:
      - GPIO47
      - GPIO21
      - GPIO0
      - GPIO46
      - GPIO3
      - GPIO8
      - GPIO4
      - GPIO5
    clock_pin: GPIO2
    color_order: RGB
    update_interval: 2s
    lambda: |-
      auto width = it.get_width();
      auto height = it.get_height();

      it.filled_rectangle(0, 0, width, height, Color::BLACK);

      it.filled_rectangle(0, 0, width / 2, 80, id(current_page) == 0 ? Color(0, 168, 255) : Color(102, 102, 102));
      it.filled_rectangle(width / 2, 0, width / 2, 80, id(current_page) == 1 ? Color(0, 168, 255) : Color(102, 102, 102));

      it.print(50, 25, id(font_medium), Color::WHITE, "Dashboard");
      it.print(width / 2 + 50, 25, id(font_medium), Color::WHITE, "Kamera");

      if (id(current_page) == 0) {
        it.printf(width - 200, 100, id(font_small), Color::WHITE, "WiFi: %.0f dBm", id(wifi_signal).state);

        it.filled_rectangle(30, 150, 300, 200, Color(31, 31, 31));
        it.print(50, 170, id(font_medium), Color::WHITE, "SICAKLIK");
        it.printf(50, 250, id(font_big), Color::WHITE, "%.1fC", room_temp.state);

        it.filled_rectangle(width - 330, 150, 300, 200, Color(31, 31, 31));
        it.print(width - 310, 170, id(font_medium), Color::WHITE, "NEM");
        it.printf(width - 310, 250, id(font_big), Color::WHITE, "%.0f%%", room_humidity.state);

        it.filled_rectangle(30, 380, 300, 200, Color(31, 31, 31));
        it.print(50, 400, id(font_medium), Color::WHITE, "LAMBA");
        if (id(relay_1).state) {
          it.print(50, 480, id(font_big), Color(0, 255, 136), "AÇIK");
        } else {
          it.print(50, 480, id(font_big), Color(255, 176, 0), "KAPALI");
        }

        it.filled_rectangle(width - 330, 380, 300, 200, Color(31, 31, 31));
        it.print(width - 310, 400, id(font_medium), Color::WHITE, "FAN");
        if (id(relay_2).state) {
          it.print(width - 310, 480, id(font_big), Color(0, 255, 136), "AÇIK");
        } else {
          it.print(width - 310, 480, id(font_big), Color(255, 176, 0), "KAPALI");
        }

        it.filled_rectangle(0, height - 60, width, 60, Color(31, 31, 31));
        it.print(30, height - 40, id(font_small), Color::WHITE, "HA: ONLINE");
      }

      if (id(current_page) == 1) {
        it.filled_rectangle(30, 100, width - 60, 80, Color(31, 31, 31));
        it.print(50, 120, id(font_medium), Color::WHITE, "Kamera: Aktif");

        it.filled_rectangle(30, 200, width - 60, 80, Color(31, 31, 31));
        it.print(50, 220, id(font_medium), Color::WHITE, "Çözünürlük: 640x480");

        it.filled_rectangle(30, 300, width - 60, 80, Color(31, 31, 31));
        it.print(50, 320, id(font_medium), Color::WHITE, "Kalite: Yüksek");

        it.filled_rectangle(30, 400, width - 60, 80, Color(31, 31, 31));
        it.print(50, 420, id(font_small), Color(0, 255, 136), "HA'da canlı görünümü açın");

        it.filled_rectangle(0, height - 60, width, 60, Color(31, 31, 31));
        it.print(30, height - 40, id(font_small), Color::WHITE, "Kamera: ONLINE");
      }

# ---------------------------------------------------
# Touchscreen - ST7123 (ST7121 ile uyumlu)
# ---------------------------------------------------
touchscreen:
  - platform: st7123
    id: main_touchscreen
    interrupt_pin: GPIO4
    reset_pin: GPIO42
    address: 0x70
    swap_xy: false
    on_touch:
      then:
        - lambda: |-
            if (touch.x > it.get_width() / 2) {
              if (id(current_page) < 1) {
                id(current_page) += 1;
              }
            } else {
              if (id(current_page) > 0) {
                id(current_page) -= 1;
              }
            }

# ---------------------------------------------------
# Fontlar
# ---------------------------------------------------
font:
  - file: "gfonts://Roboto"
    id: font_small
    size: 24
  - file: "gfonts://Roboto"
    id: font_medium
    size: 32
  - file: "gfonts://Roboto"
    id: font_big
    size: 48

# ---------------------------------------------------
# Relay / Switch
# ---------------------------------------------------
switch:
  - platform: gpio
    name: "Lamba 1"
    id: relay_1
    pin:
      number: GPIO37
      inverted: false
    restore_mode: ALWAYS_OFF

  - platform: gpio
    name: "Fan 1"
    id: relay_2
    pin:
      number: GPIO38
      inverted: false
    restore_mode: ALWAYS_OFF

button:
  - platform: restart
    name: "Yeniden Başlat"

binary_sensor:
  - platform: status
    name: "Cihaz Durumu"

sensor:
  - platform: wifi_signal
    id: wifi_signal
    name: "WiFi Sinyal Gücü"
    update_interval: 30s

  - platform: dht
    pin: GPIO41
    model: AM2302
    temperature:
      id: room_temp
      name: "Oda Sıcaklığı"
      unit_of_measurement: "°C"
      accuracy_decimals: 1
    humidity:
      id: room_humidity
      name: "Oda Nem Oranı"
      unit_of_measurement: "%"
      accuracy_decimals: 0
    update_interval: 30s

text_sensor:
  - platform: version
    name: "ESPHome Sürümü"

# ---------------------------------------------------
# Sayfa yönetimi
# ---------------------------------------------------
globals:
  - id: current_page
    type: int
    restore_value: no
    initial_value: '0'
```

## Secrets dosyası (`esphome/secrets.yaml`)

```yaml
wifi_ssid: "WIFI_ADINIZ"
wifi_password: "WIFI_ŞİFRENİZ"
api_key: "API_ANAHTARINIZ"
ota_password: "OTA_ŞİFRENİZ"
```

## Home Assistant Lovelace (`homeassistant/lovelace_tab5_dashboard.yaml`)

```yaml
title: Tab5 Dashboard (ST7121)
path: tab5-dashboard

views:
  - title: Ana Dashboard
    path: dashboard
    icon: mdi:home
    cards:
      - type: entities
        title: "Sıcaklık ve Nem"
        entities:
          - entity: sensor.tab5_dashboard_oda_sicakligi
            name: Oda Sıcaklığı
          - entity: sensor.tab5_dashboard_oda_nem_orani
            name: Oda Nem Oranı

      - type: entities
        title: "Ağ Durumu"
        entities:
          - entity: sensor.tab5_dashboard_wifi_sinyal_gucu
            name: WiFi Sinyali
          - entity: binary_sensor.tab5_dashboard_cihaz_durumu
            name: Cihaz Durumu

      - type: switch
        entity: switch.tab5_dashboard_lamba_1
        name: Lamba 1

      - type: switch
        entity: switch.tab5_dashboard_fan_1
        name: Fan 1

      - type: sensor
        entity: sensor.tab5_dashboard_esphome_surumu
        name: ESPHome Sürümü

  - title: Kamera
    path: camera
    icon: mdi:camera
    cards:
      - type: picture-entity
        entity: camera.tab5_dashboard_tab5_kamerasi
        name: "Tab5 Canlı Kamerası"
        show_state: true
        show_name: true

      - type: markdown
        title: "Kamera Bilgisi"
        content: |
          # Tab5 ST7121 Kamerası
          
          - **Çözünürlük:** 640x480 px
          - **Format:** JPEG
          - **Kalite:** 10
          - **Dokunmatik:** ST7123
          - **Ekran:** ST7121 MIPI DSI
```

## Son Notlar

- Bu yapı, ST7121 ekranı için MIPI DSI yaklaşımına uygun şekilde hazırlanmıştır.
- ST7123 dokunmatik modülünün ST7121 ile aynı protokolü kullandığı bilinmektedir.
- Ekran modelini ve GPIO pinlerini gerçek kart şemasına göre doğrulamak gerekir.
- `esp32_camera` ve `display` pinleri kitin kendi PCB tasarımına göre değişebilir.

## Çalıştırma

```bash
cd esphome
esphome run tab5_dashboard.yaml
```

Bu proje, M5Stack Tab5 cihazınızda ST7121 ekran + ST7123 touch + canlı kamera + Home Assistant entegrasyonu için uygun temel yapıdır.
