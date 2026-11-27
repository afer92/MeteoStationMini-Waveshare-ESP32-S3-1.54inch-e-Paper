# 🌦️ Station Météo e-Paper (ESPHome & Home Assistant) / e-Paper Weather Station

[![GitHub Release](https://img.shields.io/github/v/release/afer92/MeteoStationMini-Waveshare-ESP32-S3-1.54inch-e-Paper)](https://afer92.github.io/MeteoStationMini-Waveshare-ESP32-S3-1.54inch-e-Paper/)
[![License](https://img.shields.io/badge/GPL%20v3-grey)](https://github.com/afer92/MeteoStationMini-Waveshare-ESP32-S3-1.54inch-e-Paper/blob/main/LICENSE)
[![ESPHome](https://img.shields.io/badge/ESPHome-2026.11-grey)](https://esphome.io)
[![Build Firmware](https://github.com/afer92/MeteoStationMini-Waveshare-ESP32-S3-1.54inch-e-Paper/actions/workflows/deploy-pages.yml/badge.svg)](https://github.com/afer92/MeteoStationMini-Waveshare-ESP32-S3-1.54inch-e-Paper/actions/workflows/deploy-pages.yml)
[![Install via Browser](https://img.shields.io/badge/Install%20via%20browser--grey)](https://afer92.github.io/MeteoStationMini-Waveshare-ESP32-S3-1.54inch-e-Paper/)

![Aperçu de la Station Météo e-Paper / e-Paper Weather Station Preview](docs/screen.jpg)

[🇫🇷 Français](#-français) | [🇬🇧 English](#-english)

---

## 🇫🇷 Français

Station météo autonome sur batterie basée sur la carte **Waveshare ESP32-S3-ePaper-1.54 (Révision V2)** et **ESPHome**, conçue pour afficher les données de **Home Assistant** et les mesures intérieures locales avec une consommation ultra-faible.

### 📂 Fichiers du dépôt
* **`meteostation.yaml`** : Configuration complète de production (interface graphique par pictogrammes, météo Home Assistant, *Deep Sleep* configurable, verrouillage batterie `GPIO17`/`GPIO6`).
* **`minimal.yaml`** : Configuration minimale prête à l'emploi pour tester rapidement le matériel (écran, capteur SHTC3, batterie) sans *Deep Sleep* ni police externe.

### ✨ Fonctionnalités
* **Affichage e-Paper 1.54" (200×200 px) universel par pictogrammes** :
  * Heure synchronisée via Home Assistant et indicateur graphique du niveau de batterie (`GPIO4`).
  * Conditions météo extérieures (icône dynamique Material Design, température actuelle et prévisions Min / Max du jour).
  * Température et humidité intérieures mesurées par le capteur embarqué **SHTC3** (`I2C 0x70`) avec icônes maison, thermomètre et goutte d'eau.
* **Autonomie maximale et contrôles dans Home Assistant** :
  * Gestion matérielle spécifique à la **révision V2** (`GPIO6` pour l'écran, `GPIO42` pour le rail I2C).
  * Verrouillage matériel des broches pendant le *Deep Sleep* (`gpio_hold_en` sur `GPIO17` et `GPIO6`) évitant la coupure de l'alimentation batterie.
  * **Durée du Deep Sleep réglable** directement depuis Home Assistant (`1` à `1440 min`, sauvegardée en mémoire flash).
  * **Offsets d'étalonnage réglables** dans Home Assistant pour ajuster la température et l'humidité intérieures.
  * Interrupteur virtuel **« Bloquer Deep Sleep (Mode OTA) »** pour faciliter les mises à jour sans fil.

### 📌 Brochage matériel (Waveshare ESP32-S3-ePaper-1.54 V2)

| Composant | Fonction | Broche ESP32-S3 | Note matérielle |
| :--- | :--- | :--- | :--- |
| **Alimentation Écran** | `EPD_PWR_EN` | `GPIO6` | Actif à l'état bas (`inverted: true`) |
| **Alimentation Périph. V2** | `PERI_PWR_EN` | `GPIO42` | Alimente le bus I2C / SHTC3 (`inverted: true`) |
| **Maintien Batterie** | `BAT_PWR_LATCH` | `GPIO17` | Verrouillé à `HIGH` en veille via `gpio_hold_en` |
| **Mesure Batterie** | `BAT_ADC` | `GPIO4` | Pont diviseur ×2 (`attenuation: 12db`) |
| **Bouton Réveil** | `WAKE_BTN` | `GPIO18` | Réveil externe (`INPUT_PULLUP`, actif bas) |
| **Écran e-Paper (SPI)** | `CLK` / `MOSI` | `GPIO12` / `GPIO13` | Bus SPI matériel |
| **Écran e-Paper (Ctrl)** | `CS` / `DC` / `RST` / `BUSY` | `GPIO11` / `10` / `9` / `8` | Modèle `1.54inv2` (200×200 px) |
| **Capteur SHTC3 (I2C)** | `SDA` / `SCL` | `GPIO47` / `GPIO48` | Adresse I2C `0x70` |

### 🛠️ Prérequis Home Assistant
Ajoutez les capteurs suivants dans votre fichier `configuration.yaml` (ou `templates.yaml`) pour extraire les températures minimale et maximale du jour depuis votre entité météo :

```yaml
template:
  - trigger:
      - platform: time_pattern
        hours: "/1"
      - platform: homeassistant
        event: start
    action:
      - service: weather.get_forecasts
        data:
          type: daily
        target:
          entity_id: weather.forecast_maison
        response_variable: daily_forecast
    sensor:
      - name: "Météo Max Aujourd'hui"
        unique_id: meteo_max_aujourd_hui
        unit_of_measurement: "°C"
        device_class: temperature
        state: "{{ daily_forecast['weather.forecast_maison'].forecast[0].temperature }}"

      - name: "Météo Min Aujourd'hui"
        unique_id: meteo_min_aujourd_hui
        unit_of_measurement: "°C"
        device_class: temperature
        state: "{{ daily_forecast['weather.forecast_maison'].forecast[0].templow }}"
```

### 🚀 Installation
#### Option 1 : Installation directe via navigateur (Recommandée)
1. Branchez la carte en USB-C à votre ordinateur et passez-la en mode Bootloader (maintenir **BOOT** `GPIO0`, appuyer sur **RESET**, relâcher **BOOT**).
2. Ouvrez la page **[ESP Web Tools](https://afer92.github.io/MeteoStationMini-Waveshare-ESP32-S3-1.54inch-e-Paper/)** dans Google Chrome ou Microsoft Edge.
3. Cliquez sur **Connecter et Installer**, puis configurez votre réseau Wi-Fi directement dans la fenêtre après le flashage.
4. Débranchez le câble USB-C et effectuez un **appui long (~1,5 s) sur le bouton Power** pour enclencher le verrouillage batterie.

#### Option 2 : Compilation manuelle avec ESPHome
1. Téléchargez `materialdesignicons-webfont.ttf` dans `/config/esphome/fonts/materialdesignicons-webfont.ttf`.
2. Adaptez la substitution `weather_entity` et votre configuration réseau dans `meteostation.yaml`, puis compilez et téléversez via USB-C.

### 🔄 Mise à jour sans fil (OTA)
1. Activez l'interrupteur **« Bloquer Deep Sleep (Mode OTA) »** dans Home Assistant.
2. Appuyez sur le bouton de réveil (`GPIO18`) ou attendez le prochain réveil programmé.
3. Téléversez le nouveau firmware en OTA, puis désactivez l'interrupteur pour reprendre le cycle de veille.

---

## 🇬🇧 English

Battery-powered standalone weather station built on the **Waveshare ESP32-S3-ePaper-1.54 (Revision V2)** board and **ESPHome**, designed to display **Home Assistant** weather data and local indoor sensor readings with ultra-low power consumption.

### 📂 Repository Files
* **`meteostation.yaml`**: Full production configuration (universal icon-based UI, Home Assistant weather forecast, configurable *Deep Sleep*, hardware battery latch `GPIO17`/`GPIO6`).
* **`minimal.yaml`**: Ready-to-use minimal configuration to quickly test the hardware (display, SHTC3 sensor, battery) without *Deep Sleep* or external icon fonts.

### ✨ Features
* **Universal Icon-Based 1.54" e-Paper UI (200×200 px)**:
  * Time synced from Home Assistant and dynamic graphical battery level indicator (`GPIO4`).
  * Outdoor weather conditions (dynamic Material Design icon, current temperature, and daily Min / Max forecast).
  * Indoor temperature and humidity measured by the onboard **SHTC3** sensor (`I2C 0x70`) with home, thermometer, and humidity droplet icons.
* **Maximum Battery Life & Home Assistant Controls**:
  * Dedicated **V2 hardware power rail management** (`GPIO6` for display power, `GPIO42` for the I2C peripheral rail).
  * Hardware GPIO hold during *Deep Sleep* (`gpio_hold_en` on `GPIO17` and `GPIO6`) to keep the battery power latch active between wake cycles.
  * **Adjustable Deep Sleep duration** directly from Home Assistant (`1` to `1440 min`, persisted in flash memory).
  * **Adjustable calibration offsets** in Home Assistant for indoor temperature and humidity.
  * Virtual **"Bloquer Deep Sleep (Mode OTA)"** switch for seamless over-the-air updates.

### 📌 Hardware Pinout (Waveshare ESP32-S3-ePaper-1.54 V2)

| Component | Function | ESP32-S3 Pin | Hardware Note |
| :--- | :--- | :--- | :--- |
| **Display Power** | `EPD_PWR_EN` | `GPIO6` | Active-low (`inverted: true`) |
| **V2 Peripherals Power** | `PERI_PWR_EN` | `GPIO42` | Powers I2C bus / SHTC3 (`inverted: true`) |
| **Battery Power Latch** | `BAT_PWR_LATCH` | `GPIO17` | Held `HIGH` during sleep via `gpio_hold_en` |
| **Battery Voltage** | `BAT_ADC` | `GPIO4` | ×2 voltage divider (`attenuation: 12db`) |
| **Wake Button** | `WAKE_BTN` | `GPIO18` | External wake-up (`INPUT_PULLUP`, active-low) |
| **e-Paper Display (SPI)** | `CLK` / `MOSI` | `GPIO12` / `GPIO13` | Hardware SPI bus |
| **e-Paper Display (Ctrl)** | `CS` / `DC` / `RST` / `BUSY` | `GPIO11` / `10` / `9` / `8` | `1.54inv2` panel (200×200 px) |
| **SHTC3 Sensor (I2C)** | `SDA` / `SCL` | `GPIO47` / `GPIO48` | I2C address `0x70` |

### 🛠️ Home Assistant Prerequisites
Add the following template sensors to your `configuration.yaml` (or `templates.yaml`) to extract today's min and max temperatures from your weather entity:

```yaml
template:
  - trigger:
      - platform: time_pattern
        hours: "/1"
      - platform: homeassistant
        event: start
    action:
      - service: weather.get_forecasts
        data:
          type: daily
        target:
          entity_id: weather.forecast_maison
        response_variable: daily_forecast
    sensor:
      - name: "Météo Max Aujourd'hui"
        unique_id: meteo_max_aujourd_hui
        unit_of_measurement: "°C"
        device_class: temperature
        state: "{{ daily_forecast['weather.forecast_maison'].forecast[0].temperature }}"

      - name: "Météo Min Aujourd'hui"
        unique_id: meteo_min_aujourd_hui
        unit_of_measurement: "°C"
        device_class: temperature
        state: "{{ daily_forecast['weather.forecast_maison'].forecast[0].templow }}"
```

### 🚀 Installation
#### Option 1: One-Click Browser Installation (Recommended)
1. Connect the board via USB-C to your computer and enter Bootloader mode (hold **BOOT** `GPIO0`, press **RESET**, release **BOOT**).
2. Open **[ESP Web Tools](https://afer92.github.io/MeteoStationMini-Waveshare-ESP32-S3-1.54inch-e-Paper/)** in Google Chrome or Microsoft Edge.
3. Click **Connecter et Installer**, then configure your Wi-Fi credentials directly in the browser dialog after flashing.
4. Unplug the USB-C cable and **long-press (~1.5 s) the side Power button** to engage the battery power latch.

#### Option 2: Manual Compilation with ESPHome
1. Download `materialdesignicons-webfont.ttf` and place it at `/config/esphome/fonts/materialdesignicons-webfont.ttf`.
2. Customize `weather_entity` and your network settings in `meteostation.yaml`, then compile and flash over USB-C.

### 🔄 Over-The-Air (OTA) Updates
1. Turn ON the **"Bloquer Deep Sleep (Mode OTA)"** switch in Home Assistant.
2. Press the wake button (`GPIO18`) or wait for the next scheduled wake-up cycle.
3. Flash the updated firmware wirelessly, then turn the switch back OFF to resume deep sleep cycles.
