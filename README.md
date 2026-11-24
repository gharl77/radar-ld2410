# radar-ld2410

**English** · [Français](README.fr.md)

ESPHome configuration for a room presence sensor built from an ESP32 and a
HLK-LD2410C mmWave radar, tuned for Home Assistant. The same board also acts as
a Bluetooth proxy.

The config in this repo runs a real device (`presence-soussol`, a basement).
Entity names and comments are in French; this page explains everything in
English.

## What it does

- **Presence, motion and still-presence** binary sensors, published instantly.
- **Distances and energies** averaged and de-duplicated before they are sent,
  so the Home Assistant database is not flooded (see [Why the filters](#why-the-filters)).
- **Per-gate energies (g0–g8)** next to their thresholds, for calibration,
  with a ready-made dashboard card.
- **Engineering mode kept on** automatically, including after the radar restarts.
- **Bluetooth proxy** for Home Assistant (ESP-IDF framework).
- **Diagnostics**: Wi-Fi signal, uptime, IP address, free heap, last reset reason.
- **No secrets in the YAML**: API key, OTA and fallback-AP passwords are
  `!secret` references.

## Hardware

| Part | Notes |
|---|---|
| ESP32 dev board (`esp32dev`, WROOM) | A WROVER module uses GPIO16/17 for PSRAM. |
| HLK-LD2410C | Radar firmware 2.04 here. UART at 256000 baud, 8N1. |

Wiring used by this device:

| LD2410C | ESP32 |
|---|---|
| VCC | 5V |
| GND | GND |
| TX | GPIO3 (RX) |
| RX | GPIO1 (TX) |

GPIO1/GPIO3 is UART0, the USB serial port. That is why the config sets
`logger: baud_rate: 0` (no serial logs), and why a cable flash usually needs
the radar unplugged. **On a new build, prefer GPIO16 (RX) / GPIO17 (TX)** and
remove the `baud_rate: 0` line.

## Files

| File | Purpose |
|---|---|
| [`presence-soussol.yaml`](presence-soussol.yaml) | The live device config. |
| [`modeles/radar-ld2410.yaml`](modeles/radar-ld2410.yaml) | Template: same config with `NOM` placeholders. |
| [`modeles/README.md`](modeles/README.md) | How to create a new sensor from the template. |
| [`home-assistant/carte-seuils.yaml`](home-assistant/carte-seuils.yaml) | Dashboard card: gate energy vs threshold. |
| [`reglages-radar.md`](reglages-radar.md) | Snapshot of the radar's thresholds (they live in the radar, not in the YAML). |
| [`autres/`](autres) | Backup of the two other ESPHome devices in the house (a Bluetooth proxy and an IR blaster). Unrelated to the radar. |

## Quick start

1. Copy [`modeles/radar-ld2410.yaml`](modeles/radar-ld2410.yaml) into ESPHome
   Device Builder as `presence-<room>.yaml`.
2. Replace `NOM` everywhere (5 places).
3. Create the three secrets, next to your existing `wifi_ssid` / `wifi_password`:
   `presence_<room>_api_key` (`openssl rand -base64 32`),
   `presence_<room>_ota_password`, `presence_<room>_ap_password`.
4. Check the UART pins in the `substitutions` block.
5. Flash once over USB; later updates go over the air.

Details: [`modeles/README.md`](modeles/README.md).

## Entities

Names are prefixed by the device name in Home Assistant.

| Entity (French name) | Meaning |
|---|---|
| Présence détectée | Any target. 5 s `delayed_off` to avoid flicker. |
| Mouvement détecté / Présence immobile | Moving target / still target. |
| Distance de détection, Distance (mouvement), Distance (immobile) | cm, 5 s average, sent on a 10 cm change. |
| Énergie du mouvement / Énergie immobile | %, 5 s average, sent on a 3-point change. |
| Luminosité | Radar's light sensor, raw 0–255 (not lux). |
| gN énergie mouvement / immobile | Per-gate energy, 10 s average, sent on a 2-point change. |
| gN seuil mouvement / immobile | Per-gate thresholds (stored in the radar). |
| Radar - Délai, Distance max mouvement / immobile | Absence delay (s) and last active gate. |
| Radar - Mode ingénierie, Radar - Bluetooth | Radar switches. |
| Radar - Redémarrer, Relire les paramètres, Remise à zéro usine | Radar buttons. Factory reset is disabled by default. |
| Statut, Signal Wi-Fi, Durée de fonctionnement, Adresse IP | Device health. |
| Mémoire libre, Cause du dernier redémarrage | Diagnostics for unexplained reboots. |
| Borne Wi-Fi (BSSID), Réseaux Wi-Fi visibles | Access point in use and visible networks with their signal, to diagnose weak Wi-Fi. |

Each gate covers 0.75 m: g0 is 0–0.75 m, g8 is 6–6.75 m.

## Calibrating thresholds

1. Add [`home-assistant/carte-seuils.yaml`](home-assistant/carte-seuils.yaml)
   to a dashboard (Add card → Manual), replacing `presence_soussol` with your
   device name.
2. Leave the room empty for a while and read the energy of each gate: that is
   the noise floor.
3. Set each threshold a little above the noise. A 🔴 marks a gate whose energy
   is above its threshold, i.e. one that triggers detection.
4. Sit still where people usually sit and check that the matching gate turns
   red in the "Immobile" column. If not, lower that still threshold.

## Things learned the hard way

### Engineering mode does not stick

The light sensor and the per-gate energies only publish in engineering mode,
and the radar leaves that mode every time it restarts: on power-up, after its
restart button, and after toggling its Bluetooth. `restore_mode: ALWAYS_ON` on
the switch had no effect here. What works is turning it on 10 s after boot and
re-checking every 60 s with an `interval`.

### Why the filters

The radar reports about 10 times per second. Unfiltered, a single energy
sensor produced about 32,000 state changes per day in an empty room. With a
`throttle_average`, a `round` and a `delta` filter, the same sensor produces
about 1,400. The binary sensors are not filtered, so detection stays instant.

### Home Assistant unit warnings

The LD2410 component gives its threshold numbers the `signal_strength` device
class with a `%` unit, the max-gate numbers the `distance` class with no unit,
and the light sensor the `illuminance` class with no unit. Home Assistant logs
a warning for each, and says the first combination will stop working in
2027.5. The config clears these with `device_class: ""`.

### Radar Bluetooth

The radar's own Bluetooth is only useful for the manufacturer's app
(HLKRadarTool), for example to update the radar firmware. Keep it off
otherwise. A radar firmware update or a factory reset can erase the
thresholds, hence [`reglages-radar.md`](reglages-radar.md).

## Tested with

ESPHome 2026.10.2 (ESP-IDF) and LD2410C firmware
2.04.23022511.
