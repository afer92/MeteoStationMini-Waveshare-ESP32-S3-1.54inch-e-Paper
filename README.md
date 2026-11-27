# 🌦️ Station Météo e-Paper (ESPHome & Home Assistant) / e-Paper Weather Station

[🇫🇷 Français](#-français) | [🇬🇧 English](#-english)

---

## 🇫🇷 Français

Station météo autonome sur batterie basée sur la carte **Waveshare ESP32-S3-ePaper-1.54 (Révision V2)** et **ESPHome**, conçue pour afficher les données de **Home Assistant** et les mesures intérieures locales avec une consommation ultra-faible.

### ✨ Fonctionnalités
* **Affichage e-Paper 1.54" (200×200 px)** :
  * Heure synchronisée via Home Assistant.
  * Pourcentage de batterie (`GPIO4`) et prévisions Min / Max du jour.
  * Conditions météo extérieures (icône dynamique Material Design + température).
  * Température et humidité intérieures mesurées par le capteur embarqué **SHTC3** (`I2C 0x70`).
* **Multilingue** : Changement de langue via substitution (`FR`, `EN`, `ES`, `DE`, `IT`).
* **Autonomie maximale sur batterie** :
  * Gestion matérielle spécifique à la **révision V2** (`GPIO6` pour l'écran, `GPIO42` pour le rail I2C).
  * Verrouillage matériel des broches pendant le *Deep Sleep* (`gpio_hold_en` sur `GPIO17` et `GPIO6`) évitant la coupure de l'alimentation batterie.
  * Connexion Wi-Fi ultra-rapide via IP fixe (`manual_ip`) et rafraîchissement complet toutes les 60 minutes (ou réveil manuel via le bouton `GPIO18`).
  * Interrupteur virtuel **« Bloquer Deep Sleep (Mode OTA) »** dans Home Assistant pour faciliter les mises à jour sans fil.

### 🛠️ Prérequis matériels & logiciels
1. **Carte** : Waveshare ESP32-S3-ePaper-1.54 (**V2**, écran Noir & Blanc `1.54inv2`).
2. **Police d'icônes** : Téléchargez `materialdesignicons-webfont.ttf` et placez-le dans `/config/esphome/fonts/materialdesignicons-webfont.ttf`.
3. **Capteurs Home Assistant** :
   * Une entité météo (ex. `weather.forecast_maison`).
   * Deux capteurs d'entrée/template fournissant les températures extrêmes du jour : `sensor.meteo_min_aujourd_hui` et `sensor.meteo_max_aujourd_hui`.

### 🚀 Installation et premier flashage
1. Adaptez les substitutions (`lang`, `weather_entity`) et la configuration réseau (`manual_ip`) dans le fichier YAML.
2. Compilez et téléversez le firmware via USB-C en mode Bootloader (maintenir **BOOT** `GPIO0`, appuyer sur **RESET**, relâcher **BOOT**).
3. Ajoutez l'appareil dans **Home Assistant > Paramètres > Appareils et services > ESPHome**.
4. Débranchez le câble USB-C et effectuez un **appui long (~1,5 s) sur le bouton Power** pour enclencher le verrouillage batterie.

### 🔄 Mise à jour sans fil (OTA)
1. Activez l'interrupteur **« Bloquer Deep Sleep (Mode OTA) »** dans Home Assistant.
2. Appuyez sur le bouton de réveil (`GPIO18`) ou attendez le prochain réveil programmé.
3. Téléversez le nouveau firmware en OTA, puis désactivez l'interrupteur pour reprendre le cycle de veille.

---

## 🇬🇧 English

Battery-powered standalone weather station built on the **Waveshare ESP32-S3-ePaper-1.54 (Revision V2)** board and **ESPHome**, designed to display **Home Assistant** weather data and local indoor sensor readings with ultra-low power consumption.

### ✨ Features
* **1.54" e-Paper Display (200×200 px)**:
  * Time synced from Home Assistant.
  * Battery percentage (`GPIO4`) and daily Min / Max forecast temperatures.
  * Outdoor weather conditions (dynamic Material Design icon + current temperature).
  * Indoor temperature and humidity measured by the onboard **SHTC3** sensor (`I2C 0x70`).
* **Multi-language support**: Easy language switching via YAML substitution (`FR`, `EN`, `ES`, `DE`, `IT`).
* **Maximum Battery Life**:
  * Dedicated **V2 hardware power rail management** (`GPIO6` for display power, `GPIO42` for the I2C peripheral rail).
  * Hardware GPIO hold during *Deep Sleep* (`gpio_hold_en` on `GPIO17` and `GPIO6`) to keep the battery power latch active between wake cycles.
  * Fast Wi-Fi connection using static IP (`manual_ip`) and full display refresh every 60 minutes (or manual wake-up via button `GPIO18`).
  * Virtual **"Bloquer Deep Sleep (Mode OTA)"** switch in Home Assistant for seamless over-the-air updates.

### 🛠️ Hardware & Software Prerequisites
1. **Board**: Waveshare ESP32-S3-ePaper-1.54 (**V2**, Black & White `1.54inv2` panel).
2. **Icon Font**: Download `materialdesignicons-webfont.ttf` and place it at `/config/esphome/fonts/materialdesignicons-webfont.ttf`.
3. **Home Assistant Entities**:
   * A weather entity (e.g., `weather.forecast_maison`).
   * Two template/helper sensors providing today's min/max temperatures: `sensor.meteo_min_aujourd_hui` and `sensor.meteo_max_aujourd_hui`.

### 🚀 Installation & Initial Flashing
1. Customize the substitutions (`lang`, `weather_entity`) and network settings (`manual_ip`) in the YAML file.
2. Compile and flash the firmware over USB-C in Bootloader mode (hold **BOOT** `GPIO0`, press **RESET**, release **BOOT**).
3. Add the discovered device in **Home Assistant > Settings > Devices & Services > ESPHome**.
4. Unplug the USB-C cable and **long-press (~1.5 s) the side Power button** to engage the battery power latch.

### 🔄 Over-The-Air (OTA) Updates
1. Turn ON the **"Bloquer Deep Sleep (Mode OTA)"** switch in Home Assistant.
2. Press the wake button (`GPIO18`) or wait for the next scheduled wake-up cycle.
3. Flash the updated firmware wirelessly, then turn the switch back OFF to resume deep sleep cycles.
