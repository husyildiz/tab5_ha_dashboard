# M5Stack Tab5 Home Assistant Dashboard

ESPHome tabanlı, Home Assistant ile entegre çalışan M5Stack Tab5 dashboard örneği.

## İçerik

- `tab5_dashboard.yaml`: ESPHome yapılandırması
- `secrets.yaml`: WiFi bilgileri
- `lovelace_tab5_dashboard.yaml`: Home Assistant Lovelace örneği

## Not

ST7121 ekranın doğrudan ESPHome sürücüsü kartın gerçek modeline göre değişebilir. `display` bölümünde `model` ve pinler gerçek devre şemasına göre ayarlanmalıdır.

## Hızlı Başlangıç

1. `tab5_dashboard.yaml` dosyasını ESPHome cihazına yükleyin.
2. `secrets.yaml` içindeki WiFi bilgilerini doldurun.
3. Home Assistant'a cihazı ekleyin.
4. Lovelace'e örnek dashboard kartlarını ekleyin.

## Kodlar

### `tab5_dashboard.yaml`

```yaml
substitutions:
  device_name: "tab5_dashboard"
  friendly_name: "Tab5 Dashboard"

esphome:
  name: ${device_name}
  friendly_name: ${friendly_name}
  comment: "M5Stack Tab5 Home Assistant dashboard"
  project:
    name: "m5stack.tab5_dashboard"
    version: "1.0.0"

esp32:
  board: esp32dev
  framework:
    type: arduino

logger:
  level: DEBUG

api:
ota:
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
  sda: GPIO21
  scl: GPIO22
  scan: true

switch:
  - platform: gpio
    name: "Lamba 1"
    id: relay_1
    pin:
      number: GPIO27
      inverted: false
    restore_mode: ALWAYS_OFF

  - platform: gpio
    name: "Fan 1"
    id: relay_2
    pin:
      number: GPIO26
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
    pin: GPIO4
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

font:
  - file: "gfonts://Roboto"
    id: font_small
    size: 18
  - file: "gfonts://Roboto"
    id: font_medium
    size: 26
  - file: "gfonts://Roboto"
    id: font_big
    size: 34

color:
  - id: bg_color
    hex: "000000"
  - id: panel_color
    hex: "1F1F1F"
  - id: accent_color
    hex: "00A8FF"
  - id: good_color
    hex: "00FF88"
  - id: warn_color
    hex: "FFB000"
  - id: text_color
    hex: "FFFFFF"

display:
  - platform: ili9xxx
    model: ST7789V
    cs_pin: GPIO5
    dc_pin: GPIO15
    reset_pin: GPIO33
    rotation: 90
    update_interval: 2s
    lambda: |-
      auto width = it.get_width();
      auto height = it.get_height();

      it.filled_rectangle(0, 0, width, height, id(bg_color));

      it.filled_rectangle(0, 0, width, 40, id(accent_color));
      it.print(10, 8, id(font_small), id(text_color), "TAB5 DASHBOARD");
      it.printf(200, 8, id(font_small), id(text_color), "%.0f dBm", id(wifi_signal).state);

      it.filled_rectangle(10, 55, 130, 100, id(panel_color));
      it.print(20, 65, id(font_small), id(text_color), "SICAKLIK");
      it.printf(20, 95, id(font_big), id(text_color), "%.1fC", room_temp.state);

      it.filled_rectangle(150, 55, 130, 100, id(panel_color));
      it.print(165, 65, id(font_small), id(text_color), "NEM");
      it.printf(165, 95, id(font_big), id(text_color), "%.0f%%", room_humidity.state);

      it.filled_rectangle(10, 170, 130, 100, id(panel_color));
      it.print(22, 180, id(font_small), id(text_color), "LAMBA");
      if (id(relay_1).state) {
        it.print(20, 210, id(font_medium), id(good_color), "AÇIK");
      } else {
        it.print(20, 210, id(font_medium), id(warn_color), "KAPALI");
      }

      it.filled_rectangle(150, 170, 130, 100, id(panel_color));
      it.print(170, 180, id(font_small), id(text_color), "FAN");
      if (id(relay_2).state) {
        it.print(170, 210, id(font_medium), id(good_color), "AÇIK");
      } else {
        it.print(170, 210, id(font_medium), id(warn_color), "KAPALI");
      }

      it.filled_rectangle(0, height - 28, width, 28, id(panel_color));
      it.print(10, height - 21, id(font_small), id(text_color), "HA: ONLINE");
```

### `secrets.yaml`

```yaml
wifi_ssid: "WIFI_ADINIZ"
wifi_password: "WIFI_ŞİFRENİZ"
```

### `lovelace_tab5_dashboard.yaml`

```yaml
type: grid
columns: 2
square: false
cards:
  - type: sensor
    entity: sensor.oda_sicakligi
    name: Oda Sıcaklığı
    graph: line

  - type: sensor
    entity: sensor.oda_nem_orani
    name: Oda Nem Oranı
    graph: line

  - type: switch
    entity: switch.lamba_1
    name: Lamba 1

  - type: switch
    entity: switch.fan_1
    name: Fan 1

  - type: sensor
    entity: sensor.wifi_sinyal_gucu
    name: WiFi Gücü

  - type: binary_sensor
    entity: binary_sensor.cihaz_durumu
    name: Cihaz Durumu
```

## Sonraki adım

Bir sonraki mesajda bunu bir GitHub repo halinde düzenleyip, dosyaları net şekilde ekleyebiliriz. Ayrıca sana `README.md` ve `platformio.ini` gibi ek dosyaları da ekleyebilirim.
