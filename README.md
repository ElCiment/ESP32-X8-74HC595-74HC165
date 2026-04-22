# ESP32 X8 Relay Board (Type-C) - ESPHome Template

[![ESPHome](https://img.shields.io/badge/ESPHome-2026.4+-blue.svg)](https://esphome.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-Compatible-green.svg)](https://www.home-assistant.io/)

> 🇫🇷 Configuration ESPHome fonctionnelle pour la nouvelle carte ESP32 X8 Relay (version Type-C) avec bus partagé via registres à décalage 74HC595/74HC165.
>
> 🇬🇧 Working ESPHome configuration for the new ESP32 X8 Relay board (Type-C version) with shared bus via 74HC595/74HC165 shift registers.

---

## 🇫🇷 Description (Français)

Cette carte ESP32 8 relais (version Type-C) est particulièrement difficile à configurer car elle n'utilise **pas** de GPIOs directs pour les relais. À la place, elle utilise un **bus partagé** avec des registres à décalage :

- `74HC595` pour les 8 sorties (relais)
- `74HC165` pour les 8 entrées optocouplées
- Les lignes RCK (latch) et SCK (clock) sont **partagées** entre les deux

Ce template fournit une configuration ESPHome **testée et fonctionnelle** qui :

- ✅ Contrôle individuellement les 8 relais depuis Home Assistant
- ✅ Lit les 8 entrées optocouplées
- ✅ **Fonctionne en local** : les switches physiques togglent les relais même sans HA
- ✅ Synchronisation temps réel entre les switches physiques et Home Assistant
- ✅ Interface web intégrée pour debug
- ✅ Initialisation propre (tous les relais OFF au démarrage)

### 🛒 Carte compatible

Carte communément vendue sous le nom **"ESP32 X8 Relay Modbus Board"** (version Type-C, identifiable par son port USB-C et l'absence de GPIOs directs aux relais).

### 📍 Brochage utilisé

| Signal | GPIO ESP32 | Fonction |
|--------|-----------|----------|
| `DATA_OUT` | GPIO33 | Données vers les relais (74HC595) |
| `RCK / LATCH` | GPIO25 | Latch partagé |
| `SCK / CLOCK` | GPIO26 | Clock partagé |
| `DATA_IN` | GPIO27 | Données depuis les entrées (74HC165) |
| `ENABLE` | GPIO13 | Activation des chips (actif LOW) |

### ⚠️ Câblage des entrées (TRÈS IMPORTANT)

Les entrées sont **optocouplées et attendent une tension**, pas un contact sec à GND !

```
   +12V (de votre alim 9-24V)
        │
        │
   [SWITCH momentané]
        │
        │
       IN1 (sur le bornier vert)
```

**Le commun (GND) de l'alim doit être connecté à la carte via le bornier 9-24V.**

### 🚀 Installation

#### 1. Prérequis

- [ESPHome](https://esphome.io/) installé (`pip install esphome`)
- Une carte ESP32 X8 Relay (Type-C)
- Une alimentation 9-24V pour la carte
- Câble USB-C pour le premier flash

#### 2. Cloner le dépôt

```bash
git clone https://github.com/VOTRE-USER/esp32-x8-relay-esphome.git
cd esp32-x8-relay-esphome
```

#### 3. Configurer vos secrets

```bash
# Copier le template
cp secrets.yaml.template secrets.yaml

# Editer secrets.yaml et remplir vos valeurs
# - WiFi SSID/password
# - Cle API (32 bytes base64)
# - Mot de passe OTA
```

Pour générer une clé API, vous pouvez utiliser :
```bash
openssl rand -base64 32
```

#### 4. Compiler et flasher

```bash
# Premier flash (USB)
esphome run esp32-x8-relay.yaml

# Mises a jour suivantes (OTA, sans fil)
esphome run esp32-x8-relay.yaml
```

#### 5. Si le flash USB ne se connecte pas

Mettez la carte en mode flash manuel :
1. Maintenez **BOOT**
2. Appuyez et relâchez **RESET**
3. Relâchez **BOOT**
4. Lancez le flash

### 🏠 Intégration Home Assistant

1. Dans HA → **Settings** → **Devices & Services**
2. **Add Integration** → **ESPHome**
3. Entrez l'IP ou `esp32-x8-relay.local`
4. Collez votre `api_encryption_key`
5. **16 entités** apparaissent : 8 switches Relais + 8 binary_sensors Entrées

### 🐛 Interface web

Accessible directement sur `http://esp32-x8-relay.local` (ou IP de la carte) :
- État des 8 relais et 8 entrées en temps réel
- Boutons de test (TEST - Tous ON/OFF)
- Logs en direct pour debug

### 🎯 Logique de fonctionnement

- **Au démarrage** : tous les relais OFF (sécurité)
- **Switch momentané sur IN*** : toggle le relais R* correspondant à chaque appui
- **Bouton dans HA** : contrôle direct on/off
- **Synchronisation bidirectionnelle** : HA et physique restent synchronisés

### 🔧 Customisation

Pour changer le mapping switch → relais, éditez le tableau `bit_to_input` dans le YAML :

```cpp
static const int bit_to_input[8] = {8, 6, 4, 2, 1, 3, 5, 7};
//                                  ^ Bit 0 du registre = IN8
//                                     ^ Bit 1 = IN6, etc.
```

Pour désactiver le toggle automatique (et n'utiliser HA que comme observateur), commentez la section `if (pressed) { ... }` dans le bloc `interval`.

---

## 🇬🇧 Description (English)

This ESP32 8-relay board (Type-C version) is particularly tricky to configure because it does **not** use direct GPIOs for the relays. Instead, it uses a **shared bus** with shift registers:

- `74HC595` for the 8 outputs (relays)
- `74HC165` for the 8 optocoupled inputs
- RCK (latch) and SCK (clock) lines are **shared** between both chips

This template provides a **tested and working** ESPHome configuration that:

- ✅ Individual control of all 8 relays from Home Assistant
- ✅ Reads all 8 optocoupled inputs
- ✅ **Works locally**: physical switches toggle relays even without HA
- ✅ Real-time sync between physical switches and Home Assistant
- ✅ Built-in web interface for debugging
- ✅ Clean initialization (all relays OFF on boot)

### 🛒 Compatible board

Commonly sold as **"ESP32 X8 Relay Modbus Board"** (Type-C version, identifiable by its USB-C port and lack of direct GPIOs to relays).

### 📍 Pinout used

| Signal | ESP32 GPIO | Function |
|--------|-----------|----------|
| `DATA_OUT` | GPIO33 | Data to relays (74HC595) |
| `RCK / LATCH` | GPIO25 | Shared latch |
| `SCK / CLOCK` | GPIO26 | Shared clock |
| `DATA_IN` | GPIO27 | Data from inputs (74HC165) |
| `ENABLE` | GPIO13 | Chip enable (active LOW) |

### ⚠️ Input wiring (VERY IMPORTANT)

Inputs are **optocoupled and expect a voltage**, not a dry contact to GND!

```
   +12V (from your 9-24V supply)
        │
        │
   [Momentary SWITCH]
        │
        │
       IN1 (on green terminal)
```

**The supply GND must be connected to the board via the 9-24V terminal.**

### 🚀 Installation

#### 1. Requirements

- [ESPHome](https://esphome.io/) installed (`pip install esphome`)
- An ESP32 X8 Relay board (Type-C)
- A 9-24V power supply for the board
- USB-C cable for first flash

#### 2. Clone the repository

```bash
git clone https://github.com/YOUR-USER/esp32-x8-relay-esphome.git
cd esp32-x8-relay-esphome
```

#### 3. Configure your secrets

```bash
# Copy the template
cp secrets.yaml.template secrets.yaml

# Edit secrets.yaml and fill in your values
# - WiFi SSID/password
# - API key (32 bytes base64)
# - OTA password
```

To generate an API key:
```bash
openssl rand -base64 32
```

#### 4. Compile and flash

```bash
# First flash (USB)
esphome run esp32-x8-relay.yaml

# Subsequent updates (OTA, wireless)
esphome run esp32-x8-relay.yaml
```

#### 5. If USB flash doesn't connect

Put the board in manual flash mode:
1. Hold **BOOT**
2. Press and release **RESET**
3. Release **BOOT**
4. Launch flash

### 🏠 Home Assistant integration

1. In HA → **Settings** → **Devices & Services**
2. **Add Integration** → **ESPHome**
3. Enter IP or `esp32-x8-relay.local`
4. Paste your `api_encryption_key`
5. **16 entities** appear: 8 Relay switches + 8 Input binary_sensors

### 🐛 Web interface

Accessible at `http://esp32-x8-relay.local` (or board IP):
- Real-time state of 8 relays and 8 inputs
- Test buttons (TEST - All ON/OFF)
- Live logs for debugging

### 🎯 Operating logic

- **At boot**: all relays OFF (safety)
- **Momentary switch on IN***: toggles the corresponding relay R* on each press
- **Button in HA**: direct on/off control
- **Bidirectional sync**: HA and physical stay synchronized

---

## 🤝 Contributing

PRs and issues welcome! If you find a bug or have improvements, please open an issue.

## 📄 License

[MIT License](LICENSE) - Free to use, modify and distribute.

## 🙏 Credits

- Hardware reverse engineering originally documented by [bertstep](https://github.com/bertstep) (ESP32-X8-Relay-Board--NEW--Type-C)
- ESPHome configuration developed and tested by the community
- Thanks to ESPHome team for the amazing platform

## 📚 Useful links

- [ESPHome documentation](https://esphome.io/)
- [Home Assistant](https://www.home-assistant.io/)
- [74HC595 datasheet](https://www.ti.com/product/SN74HC595)
- [74HC165 datasheet](https://www.ti.com/product/SN74HC165)
