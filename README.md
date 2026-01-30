diff --git a/README.md b/README.md
index 988591b493b00e1f7dafeacfce6bb7553520072a..aba27dac201a5cf542e33b9b993727cee39e601e 100644
--- a/README.md
+++ b/README.md
@@ -1,385 +1,203 @@
-# Roborock Abwasser-Tankstation → Home Assistant (ESP32-C3 + ESPHome)
+# Roborock Abwasser Tankstation → Home Assistant  
+ESP32-C3 + ESPHome + Zigbee Relais + 12V Pumpe
 
-Dieses Projekt zeigt, wie man bei einer Roborock-Abwasserstation das **Original-Signal “VOLL/LEER”** (Hall-Sensor / Schwimmer+Magnet) **parallel abgreift** und über einen **ESP32-C3** in **Home Assistant** sichtbar macht.
+Dieses Projekt zeigt, wie man bei einer Roborock-Abwasserstation:
 
-✅ **Wichtig:** Die Roborock bleibt vollständig funktionsfähig – der ESP „lauscht“ nur mit.  
-✅ OTA Updates (später ohne USB)  
-✅ Optional: WLAN-Status LEDs (grün/rot)
+✅ das originale **VOLL / LEER Signal** abgreift (ohne Roborock zu verändern)  
+✅ in **Home Assistant** sichtbar macht  
+✅ optional automatisch **Abwasser abpumpt**  
+✅ optional Frischwasser sicher zuführt  
+
+Der ESP32 liest nur den Sensor – geschaltet wird über ein Zigbee-Relais.
+
+Kein offizielles Roborock-Projekt.
 
 ---
 
-## Was wir bauen
+## 🧠 Funktionsprinzip (einfach erklärt)
 
-Die Roborock-Station erkennt über ihren internen Sensor nur **zwei Zustände**:
+Die Roborock-Station kennt nur zwei Zustände:
 
-- **LEER**
-- **VOLL**
+- LEER (Magnet am Sensor)
+- VOLL (Magnet weg)
 
-Dieses Digitalsignal (an/aus) wird parallel abgegriffen und als Binary-Sensor + Textsensor in Home Assistant angezeigt.
+Dieses Signal wird parallel abgegriffen:
 
-Zusätzlich (optional):
-- 🟢 **Grün:** WLAN verbunden (kurz an), bei WLAN-Suche blinkt
-- 🔴 **Rot:** WLAN getrennt
+Roborock → ESP32 → Home Assistant
+
+Wenn „VOLL“ erkannt wird:
+- Home Assistant schaltet per Zigbee:
+  - Magnetventil AUF
+  - Pumpe EIN
+- nach Zeit oder bei „LEER“ wieder AUS
 
 ---
 
-## 0) Du brauchst
+# ⚙️ Komponenten (genau aus diesem Build)
 
-### Hardware
-- 1× **ESP32-C3 DevKitM-1** (oder kompatibel)
-- 1× USB-C Kabel (Flashen + Strom)
-- Kabel / Wago / Lötzeug
-- Optional:
-  - 1× grüne LED
-  - 1× rote LED  
-  *(Hinweis: normalerweise mit Vorwiderstand betreiben – siehe Sicherheitshinweis weiter unten)*
+Keine Werbung – reine Referenzlinks.
 
-### Software
-- Home Assistant + **ESPHome Add-on**
-- (optional) USB Treiber (meist automatisch)
+## ESP32
+ESP32-C3 SuperMini  
+https://de.aliexpress.com/item/1005007663345442.html
 
 ---
 
-## 1) WICHTIG: Sensor-Kabel richtig verstehen (3-polig) ⚠️
+## Abwasser
+
+### DC Aquariumpumpe 12V
+https://de.aliexpress.com/item/1005007587818206.html
 
-Am Roborock-Hall-Sensor sind **3 Adern** (bei vielen Geräten z. B. Schwarz / Weiß / Rot oder Orange):
+### Zigbee 1-Kanal Relais (Tuya / eWeLink / Smart Life)
+https://de.aliexpress.com/item/1005006848429036.html
+
+(bei Ventil + Pumpe besser 2-Kanal oder 2× Relais)
+
+---
 
-- **Schwarz = GND (Minus)**
-- **Weiß = Signal**
-- **Rot/Orange = 3.3V Versorgung (nur für Roborock intern)**
+## Strom
 
-✅ Wir greifen **nur Schwarz + Weiß** ab  
-❌ **Rot/Orange niemals** an den ESP anschließen
+### 230V → 12V Netzteil
+https://de.aliexpress.com/item/1005010168079946.html
 
-**Warum?**  
-Der ESP darf **NICHT** aus dem Roborock-Sensor-Kabel versorgt werden.  
-Der ESP bekommt seinen Strom **separat per USB**.
+### DC-DC Stepdown (optional)
+https://de.aliexpress.com/item/1005008844540337.html
 
 ---
 
-## 2) Verdrahtung (Übersicht)
+## Frischwasser (Unterdruckventil)
+https://de.aliexpress.com/item/1005005848051466.html
 
-### Sensor → ESP32-C3
-- **Schwarz (GND)** → **GND** am ESP
-- **Weiß (Signal)** → **GPIO4** am ESP
+---
+
+# 🚰 Schlauch-Aufbau Abwasser (sehr wichtig)
 
-**Optional (empfohlen):** 1k–4.7k Widerstand in Reihe zwischen Signal und GPIO4 (Schutz).
+Von Roborock Richtung Abfluss:
 
-### LEDs (optional)
-Damit es keine Bootprobleme gibt, **nicht GPIO8/9**, sondern:
-- 🟢 grün → **GPIO5**
-- 🔴 rot → **GPIO6**
+1. Roborock Ausgang  
+2. Rückschlagventil  
+3. Pumpe  
+4. (optional zweites Rückschlagventil)  
+5. Magnetventil (NC)  
+6. Abfluss / Kanister  
 
-**Einfacher Aufbau (ohne Vorwiderstand NICHT empfohlen):**
-- LED Minus (−) an **GND**
-- LED Plus (+) an **GPIO5 / GPIO6**
+Warum:
 
-> ⚠️ **Sicherheit:** In der Praxis sollten LEDs immer mit Vorwiderstand (z. B. 220–470Ω) betrieben werden, sonst kann die LED oder der GPIO beschädigt werden.  
-> Wenn du es “direkt” machst, ist das auf eigenes Risiko.
+- Rückschlagventil = kein Rücklauf
+- Magnetventil NC = im stromlosen Zustand ZU
+- verhindert Siphon & Nachziehen
 
 ---
 
-## 2.1 ASCII-Skizze (Schaltbild)
+# ⚡ Elektrik (12V)
 
-Roborock Hall Sensor (ONYX3)  
-Schwarz (GND)  ———————––>  ESP32-C3 GND  
-Weiß (SIGNAL)  ———————––>  ESP32-C3 GPIO4  
-Rot/Orange (3.3V)  – NICHT anfassen / bleibt an Roborock –
+Grundregel:
 
-LEDs (optional)  
-GPIO5  —————–>  Grün LED (+)  
-GPIO6  —————–>  Rot  LED (+)  
-GND   —————–>  Beide LEDs (-)
+Minus bleibt immer verbunden.  
+Plus wird über Relais geschaltet.
 
-ESP32-C3 Stromversorgung:  
-USB-C / 5V (separat)
+## Pumpe
 
----
++12V Netzteil → COM Relais  
+NO Relais → Pumpe +  
+Pumpe − → Netzteil −  
 
-## 3) ESPHome in Home Assistant einrichten
+## Magnetventil (falls vorhanden)
 
-### 3.1 ESPHome Add-on installieren
-Home Assistant:  
-- Einstellungen → Add-ons → Add-on Store  
-- **ESPHome installieren**  
-- starten
++12V → COM  
+NO → Ventil +  
+Ventil − → Netzteil −  
 
-### 3.2 Neues Gerät anlegen
-- ESPHome öffnen → **New Device**  
-- Name: `roborock-abwasser`  
-- Board: **esp32-c3-devkitm-1**  
-- WLAN eintragen (bei mir: `BrainLess`)
+NC wird nicht benutzt.
 
 ---
 
-## 4) Was macht die Firmware?
+# 🧠 ESP32 Verdrahtung
+
+Roborock Hall Sensor:
+
+GND → ESP GND  
+Signal → ESP GPIO4  
+
+3.3V vom Sensor NICHT anschließen!
 
-Auf dem ESP läuft ESPHome und:  
-1. verbindet sich mit WLAN + Reconnect  
-2. liest Hall-Signal an GPIO4  
-3. liefert:  
-   - Binary Sensor: „Abwasserbehälter voll“  
-   - Text: „LEER/VOLL“  
-4. steuert LEDs:  
-   - WLAN suchen: Grün blinkt  
-   - WLAN verbunden: Grün 30 Sekunden an, dann aus  
-   - WLAN weg: Rot an  
-5. OTA Updates
+Optional:
+
+GPIO5 → grüne LED (über 220–470Ω nach GND)  
+GPIO6 → rote LED (über 220–470Ω nach GND)
+
+ESP32 bekommt Strom per USB.
 
 ---
 
+# 🏠 Home Assistant Ablauf
+
+ESP32 liefert:
+
+- Binary Sensor: Abwasser voll
+- Text: LEER / VOLL
+
+Zigbee Relais:
+
+- schaltet Pumpe
+- schaltet Magnetventil
 
+Automation:
 
+VOLL →
+Ventil AUF →
+2s →
+Pumpe EIN →
+max 2 Minuten oder bis LEER →
+Pumpe AUS →
+2s →
+Ventil ZU
 
-switch.DEINE_PUMPE ersetzt du durch deine echte Entität.
+---
+
+# ⚠️ Sicherheit
+
+- Alle Schlauchverbindungen mit Schellen sichern
+- Netzteil genügend Ampere wählen
+- LEDs immer mit Widerstand
+- 230V sauber isolieren
+- Umbau auf eigene Verantwortung
 
-⸻
+---
 
-10) Häufige Probleme
-	• LEDs auf GPIO8/9 → Bootprobleme → besser GPIO5/6
-	• Rot/Orange (3.3V) vom Sensor an ESP angeschlossen → nicht machen
-	• VOLL/LEER falsch herum → im YAML ist das zentral:  
-return !id(abwasser_raw).state; → dort ggf. Logik drehen
-	• WLAN instabil → reboot_timeout sorgt dafür, dass der ESP wiederkommt
+# 🧪 Test
 
-⸻
+Magnet am Sensor:
 
-Disclaimer ⚠️
+dran → LEER  
+weg → VOLL  
 
-Umbau/Abgriff erfolgt auf eigenes Risiko.  
-Kein offizielles Roborock-Projekt. Keine Garantie/Haftung.
+Wenn vertauscht: YAML invertieren.
 
 ---
 
-## Weitere Arbeiten & Feedback
+Projekt von Ronny / Rinno  
+Community Projekt – keine Garantie.
+
+---
 
-🚧 **Hinweis:** Dieses Projekt ist noch in Arbeit! Ich werde es überarbeiten und neu aufzeichnen, sowie die Abläufe zur **Abwasser-Entleerung** und **Frischwasser-Zuführung** detailliert erklären.  
-📷 Bald werden auch **Bilder** und ein **Schaltplan** hinzugefügt.
+## 💬 Feedback & Ideen willkommen
 
-💡 Ich freue mich über Feedback und kreative Ideen! Viel Spaß beim Ausprobieren.
+Ich freue mich sehr über Feedback zu diesem Projekt 😊  
+Schreibt mir gerne, was ihr davon haltet oder ob euch Verbesserungen auffallen.
 
+Auch für Vorschläge, wie man das Setup noch sauberer, stabiler oder einfacher gestalten kann, bin ich jederzeit offen – egal ob es um:
 
-<img width="1536" height="1024" alt="ChatGPT Image 30  Jan  2026, 09_39_51" src="https://github.com/user-attachments/assets/a5ae0011-697b-4ae0-9533-c1b837fa571a" />
-<img width="1536" height="1024" alt="ChatGPT Image 30  Jan  2026, 09_39_17" src="https://github.com/user-attachments/assets/ab6e2440-a1b4-4dbf-8801-8d6d3559542b" />
-
+- Elektrik
+- Schlauchführung
+- Automationen
+- ESPHome
+- Home Assistant
+- Wartung
+- oder Erweiterungen
 
+geht.
 
+Das Projekt ist als Community-Projekt gedacht, und wenn ihr eigene Ideen oder Optimierungen habt, teilt sie gerne – vielleicht profitieren andere davon genauso wie ich.
 
-## 5) Fertige ESPHome YAML (1:1 kopieren)
-
-In ESPHome bei deinem Gerät → **Edit** → alles ersetzen durch:
-
-```yaml
-esphome:
-  name: roborock-abwasser
-  friendly_name: Roborock Abwasser
-  on_boot:
-    priority: -100
-    then:
-      - logger.log: "Roborock Abwasser gestartet"
-
-esp32:
-  board: esp32-c3-devkitm-1
-  framework:
-    type: esp-idf
-
-wifi:
-  ssid: "WLAN Name"
-  password: "WLAN Passwort"
-
-  fast_connect: true
-  power_save_mode: none
-  reboot_timeout: 5min
-
-  # WLAN verbunden
-  on_connect:
-    then:
-      - light.turn_on: led_green
-      - light.turn_off: led_red
-      - delay: 30s
-      - if:
-          condition:
-            wifi.connected
-          then:
-            - light.turn_off: led_green
-
-  # WLAN weg / getrennt
-  on_disconnect:
-    then:
-      - light.turn_off: led_green
-      - light.turn_on: led_red
-
-  # Fallback Hotspot
-  ap:
-    ssid: "Roborock-Abwasser-Setup"
-    password: "Passwort"
-
-captive_portal:
-
-api:
-  reboot_timeout: 10min
-
-ota:
-  platform: esphome
-
-logger:
-  level: DEBUG
-
-# LEDs als Light (damit toggle möglich ist)
-output:
-  - platform: gpio
-    pin: GPIO5
-    id: led_green_output
-
-  - platform: gpio
-    pin: GPIO6
-    id: led_red_output
-
-light:
-  - platform: binary
-    id: led_green
-    output: led_green_output
-    internal: true
-
-  - platform: binary
-    id: led_red
-    output: led_red_output
-    internal: true
-
-# Grün blinkt, solange WLAN gesucht wird
-interval:
-  - interval: 1s
-    then:
-      - if:
-          condition:
-            wifi.connected
-          then:
-            - light.turn_off: led_red
-          else:
-            - light.toggle: led_green
-
-binary_sensor:
-  # ESP online/offline (für HA)
-  - platform: status
-    name: "Roborock Abwasser ESP Status"
-
-  # Rohsignal vom Hall-Sensor (intern)
-  - platform: gpio
-    id: abwasser_raw
-    internal: true
-    pin:
-      number: GPIO4
-      mode: INPUT_PULLUP
-      inverted: true
-    filters:
-      - delayed_on: 500ms
-      - delayed_off: 2s
-
-  # Sensor für Home Assistant: rot = VOLL
-  - platform: template
-    name: "Roborock Abwasserbehälter voll"
-    device_class: problem
-    lambda: |-
-      // Magnet dran -> RAW true -> LEER
-      // Magnet weg  -> RAW false -> VOLL
-      return !id(abwasser_raw).state;
-
-text_sensor:
-  - platform: template
-    name: "Roborock Abwasser Status"
-    icon: mdi:water
-    lambda: |-
-      if (id(abwasser_raw).state) {
-        return std::string("LEER");
-      } else {
-        return std::string("VOLL");
-      }
-
-  - platform: wifi_info
-    ip_address:
-      name: "Roborock Abwasser IP"
-    ssid:
-      name: "Roborock Abwasser WLAN"
-
-sensor:
-  - platform: wifi_signal
-    name: "Roborock Abwasser WLAN Signal"
-    update_interval: 30s
-
-  - platform: uptime
-    name: "Roborock Abwasser Laufzeit"
-
-6) Flashen (ESP programmieren)
-
-Erstes Mal: per USB
-	1.	ESP per USB an PC oder HA-Server
-	2.	ESPHome → Install
-	3.	Plug into this computer → COM-Port wählen
-	4. falls nötig: BOOT-Taste gedrückt halten bis „Writing…“ startet
-	5. warten bis „Successfully uploaded"
-
-Danach: OTA (ohne Kabel)
-
-ESPHome → Install → Wirelessly
-
-⸻
-
-7) In Home Assistant hinzufügen
-
-Meist automatisch:
-	• Einstellungen → Geräte & Dienste → ESPHome
-	• Gerät erscheint: Roborock Abwasser
-
-Entitäten:
-	• „Roborock Abwasserbehälter voll“
-	• „Roborock Abwasser Status“ (LEER/VOLL)
-	• WLAN-Signal, IP, Laufzeit
-
-⸻
-
-8) Test (so testet man richtig)
-
-Schwimmer / Magnet bewegen:
-	• Magnet dran (Schwimmer unten)
-→ Text: LEER
-→ Binary „voll“: OFF
-	• Magnet weg (Schwimmer oben)
-→ Text: VOLL
-→ Binary „voll“: ON
-
-⸻
-
-9) Optional: Pumpe per Home Assistant (Beispiel: 2 Minuten)
-Wenn du eine Pumpe schalten willst (z. B. Zigbee Relais / Sonoff / etc.):
-
-
-alias: Roborock Abwasser – Pumpe 2 Minuten  
-mode: single  
-trigger:  
-  - platform: state  
-    entity_id: binary_sensor.roborock_abwasserbehalter_voll  
-action:  
-  - choose:  
-      - conditions:  
-          - condition: state  
-            entity_id: binary_sensor.roborock_abwasserbehalter_voll  
-            state: "off"   # LEER
-        sequence:  
-          - service: switch.turn_on  
-            target:  
-              entity_id: switch.DEINE_PUMPE  
-          - delay: "00:02:00"  
-          - service: switch.turn_off  
-            target:  
-              entity_id: switch.DEINE_PUMPE  
-      - conditions:  
-          - condition: state  
-            entity_id: binary_sensor.roborock_abwasserbehalter_voll  
-            state: "on"    # VOLL
-        sequence:  
-          - service: switch.turn_off  
-            target:  
-              entity_id: switch.DEINE_PUMPE  
-
-
-		
+Danke fürs Reinschauen und viel Spaß beim Basteln 🚀
