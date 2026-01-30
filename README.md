# Roborock Abwasser-Tankstation → Home Assistant (ESP32-C3 + ESPHome)

Dieses Projekt zeigt, wie man bei einer Roborock-Abwasserstation das **Original-Signal “VOLL/LEER”** (Hall-Sensor / Schwimmer+Magnet) **parallel abgreift** und über einen **ESP32-C3** in **Home Assistant** sichtbar macht.

✅ **Wichtig:** Die Roborock bleibt vollständig funktionsfähig – der ESP „lauscht“ nur mit.  
✅ OTA Updates (später ohne USB)  
✅ Optional: WLAN-Status LEDs (grün/rot)

---

## Was wir bauen

Die Roborock-Station erkennt über ihren internen Sensor nur **zwei Zustände**:

- **LEER**
- **VOLL**

Dieses Digitalsignal (an/aus) wird parallel abgegriffen und als Binary-Sensor + Textsensor in Home Assistant angezeigt.

Zusätzlich (optional):
- 🟢 **Grün:** WLAN verbunden (kurz an), bei WLAN-Suche blinkt
- 🔴 **Rot:** WLAN getrennt

---

## 0) Du brauchst

### Hardware
- 1× **ESP32-C3 DevKitM-1** (oder kompatibel)
- 1× USB-C Kabel (Flashen + Strom)
- Kabel / Wago / Lötzeug
- Optional:
  - 1× grüne LED
  - 1× rote LED  
  *(Hinweis: normalerweise mit Vorwiderstand betreiben – siehe Sicherheitshinweis weiter unten)*

### Software
- Home Assistant + **ESPHome Add-on**
- (optional) USB Treiber (meist automatisch)

---

## 1) WICHTIG: Sensor-Kabel richtig verstehen (3-polig) ⚠️

Am Roborock-Hall-Sensor sind **3 Adern** (bei vielen Geräten z. B. Schwarz / Weiß / Rot oder Orange):

- **Schwarz = GND (Minus)**
- **Weiß = Signal**
- **Rot/Orange = 3.3V Versorgung (nur für Roborock intern)**

✅ Wir greifen **nur Schwarz + Weiß** ab  
❌ **Rot/Orange niemals** an den ESP anschließen

**Warum?**  
Der ESP darf **NICHT** aus dem Roborock-Sensor-Kabel versorgt werden.  
Der ESP bekommt seinen Strom **separat per USB**.

---

## 2) Verdrahtung (Übersicht)

### Sensor → ESP32-C3
- **Schwarz (GND)** → **GND** am ESP
- **Weiß (Signal)** → **GPIO4** am ESP

**Optional (empfohlen):** 1k–4.7k Widerstand in Reihe zwischen Signal und GPIO4 (Schutz).

### LEDs (optional)
Damit es keine Bootprobleme gibt, **nicht GPIO8/9**, sondern:
- 🟢 grün → **GPIO5**
- 🔴 rot → **GPIO6**

**Einfacher Aufbau (ohne Vorwiderstand NICHT empfohlen):**
- LED Minus (−) an **GND**
- LED Plus (+) an **GPIO5 / GPIO6**

> ⚠️ **Sicherheit:** In der Praxis sollten LEDs immer mit Vorwiderstand (z. B. 220–470Ω) betrieben werden, sonst kann die LED oder der GPIO beschädigt werden.  
> Wenn du es “direkt” machst, ist das auf eigenes Risiko.

---

## 2.1 ASCII-Skizze (Schaltbild)

Roborock Hall Sensor (ONYX3)  
Schwarz (GND)  ———————––>  ESP32-C3 GND  
Weiß (SIGNAL)  ———————––>  ESP32-C3 GPIO4  
Rot/Orange (3.3V)  – NICHT anfassen / bleibt an Roborock –

LEDs (optional)  
GPIO5  —————–>  Grün LED (+)  
GPIO6  —————–>  Rot  LED (+)  
GND   —————–>  Beide LEDs (-)

ESP32-C3 Stromversorgung:  
USB-C / 5V (separat)

---

## 3) ESPHome in Home Assistant einrichten

### 3.1 ESPHome Add-on installieren
Home Assistant:  
- Einstellungen → Add-ons → Add-on Store  
- **ESPHome installieren**  
- starten

### 3.2 Neues Gerät anlegen
- ESPHome öffnen → **New Device**  
- Name: `roborock-abwasser`  
- Board: **esp32-c3-devkitm-1**  
- WLAN eintragen (bei mir: `BrainLess`)

---

## 4) Was macht die Firmware?

Auf dem ESP läuft ESPHome und:  
1. verbindet sich mit WLAN + Reconnect  
2. liest Hall-Signal an GPIO4  
3. liefert:  
   - Binary Sensor: „Abwasserbehälter voll“  
   - Text: „LEER/VOLL“  
4. steuert LEDs:  
   - WLAN suchen: Grün blinkt  
   - WLAN verbunden: Grün 30 Sekunden an, dann aus  
   - WLAN weg: Rot an  
5. OTA Updates

---

## 5) Fertige ESPHome YAML (1:1 kopieren)

In ESPHome bei deinem Gerät → **Edit** → alles ersetzen durch:

```yaml
esphome:
  name: roborock-abwasser
  friendly_name: Roborock Abwasser
  on_boot:
    priority: -100
    then:
      - logger.log: "Roborock Abwasser gestartet"

esp32:
  board: esp32-c3-devkitm-1
  framework:
    type: esp-idf

wifi:
  ssid: "WLAN Name"
  password: "WLAN Passwort"

  fast_connect: true
  power_save_mode: none
  reboot_timeout: 5min

  # WLAN verbunden
  on_connect:
    then:
      - light.turn_on: led_green
      - light.turn_off: led_red
      - delay: 30s
      - if:
          condition:
            wifi.connected
          then:
            - light.turn_off: led_green

  # WLAN weg / getrennt
  on_disconnect:
    then:
      - light.turn_off: led_green
      - light.turn_on: led_red

  # Fallback Hotspot
  ap:
    ssid: "Roborock-Abwasser-Setup"
    password: "Passwort"

captive_portal:

api:
  reboot_timeout: 10min

ota:
  platform: esphome

logger:
  level: DEBUG

# LEDs als Light (damit toggle möglich ist)
output:
  - platform: gpio
    pin: GPIO5
    id: led_green_output

  - platform: gpio
    pin: GPIO6
    id: led_red_output

light:
  - platform: binary
    id: led_green
    output: led_green_output
    internal: true

  - platform: binary
    id: led_red
    output: led_red_output
    internal: true

# Grün blinkt, solange WLAN gesucht wird
interval:
  - interval: 1s
    then:
      - if:
          condition:
            wifi.connected
          then:
            - light.turn_off: led_red
          else:
            - light.toggle: led_green

binary_sensor:
  # ESP online/offline (für HA)
  - platform: status
    name: "Roborock Abwasser ESP Status"

  # Rohsignal vom Hall-Sensor (intern)
  - platform: gpio
    id: abwasser_raw
    internal: true
    pin:
      number: GPIO4
      mode: INPUT_PULLUP
      inverted: true
    filters:
      - delayed_on: 500ms
      - delayed_off: 2s

  # Sensor für Home Assistant: rot = VOLL
  - platform: template
    name: "Roborock Abwasserbehälter voll"
    device_class: problem
    lambda: |-
      // Magnet dran -> RAW true -> LEER
      // Magnet weg  -> RAW false -> VOLL
      return !id(abwasser_raw).state;

text_sensor:
  - platform: template
    name: "Roborock Abwasser Status"
    icon: mdi:water
    lambda: |-
      if (id(abwasser_raw).state) {
        return std::string("LEER");
      } else {
        return std::string("VOLL");
      }

  - platform: wifi_info
    ip_address:
      name: "Roborock Abwasser IP"
    ssid:
      name: "Roborock Abwasser WLAN"

sensor:
  - platform: wifi_signal
    name: "Roborock Abwasser WLAN Signal"
    update_interval: 30s

  - platform: uptime
    name: "Roborock Abwasser Laufzeit"

6) Flashen (ESP programmieren)

Erstes Mal: per USB
	1.	ESP per USB an PC oder HA-Server
	2.	ESPHome → Install
	3.	Plug into this computer → COM-Port wählen
	4. falls nötig: BOOT-Taste gedrückt halten bis „Writing…“ startet
	5. warten bis „Successfully uploaded"

Danach: OTA (ohne Kabel)

ESPHome → Install → Wirelessly

⸻

7) In Home Assistant hinzufügen

Meist automatisch:
	• Einstellungen → Geräte & Dienste → ESPHome
	• Gerät erscheint: Roborock Abwasser

Entitäten:
	• „Roborock Abwasserbehälter voll“
	• „Roborock Abwasser Status“ (LEER/VOLL)
	• WLAN-Signal, IP, Laufzeit

⸻

8) Test (so testet man richtig)

Schwimmer / Magnet bewegen:
	• Magnet dran (Schwimmer unten)
→ Text: LEER
→ Binary „voll“: OFF
	• Magnet weg (Schwimmer oben)
→ Text: VOLL
→ Binary „voll“: ON

⸻

9) Optional: Pumpe per Home Assistant (Beispiel: 2 Minuten)
Wenn du eine Pumpe schalten willst (z. B. Zigbee Relais / Sonoff / etc.):


alias: Roborock Abwasser – Pumpe 2 Minuten  
mode: single  
trigger:  
  - platform: state  
    entity_id: binary_sensor.roborock_abwasserbehalter_voll  
action:  
  - choose:  
      - conditions:  
          - condition: state  
            entity_id: binary_sensor.roborock_abwasserbehalter_voll  
            state: "off"   # LEER
        sequence:  
          - service: switch.turn_on  
            target:  
              entity_id: switch.DEINE_PUMPE  
          - delay: "00:02:00"  
          - service: switch.turn_off  
            target:  
              entity_id: switch.DEINE_PUMPE  
      - conditions:  
          - condition: state  
            entity_id: binary_sensor.roborock_abwasserbehalter_voll  
            state: "on"    # VOLL
        sequence:  
          - service: switch.turn_off  
            target:  
              entity_id: switch.DEINE_PUMPE  


