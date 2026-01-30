ESP32-C3 + ESPHome + Zigbee Relais + 12V Pumpe

Dieses Projekt zeigt, wie man bei einer Roborock-Abwasserstation:

✅ das originale VOLL / LEER Signal abgreift (ohne Roborock zu verändern)  
✅ in Home Assistant sichtbar macht  
✅ optional automatisch Abwasser abpumpt  
✅ optional Frischwasser sicher zuführt  

Der ESP32 liest nur den Sensor – geschaltet wird über ein Zigbee-Relais.

Kein offizielles Roborock-Projekt.

---

🧠 Funktionsprinzip (einfach erklärt)

Die Roborock-Station kennt nur zwei Zustände:

- LEER (Magnet am Sensor)  
- VOLL (Magnet weg)  

Dieses Signal wird parallel abgegriffen:

Roborock → ESP32 → Home Assistant

Wenn „VOLL“ erkannt wird:

Home Assistant schaltet per Zigbee:

- Magnetventil AUF  
- Pumpe EIN  

nach Zeit oder bei „LEER“ wieder AUS.

---

⚙️ Komponenten (genau aus diesem Build)  
Keine Werbung – reine Referenzlinks.

### ESP32  
ESP32-C3 SuperMini  
https://de.aliexpress.com/item/1005007663345442.html  

---

### Abwasser

DC Aquariumpumpe 12V  
https://de.aliexpress.com/item/1005007587818206.html  

Zigbee 1-Kanal Relais (Tuya / eWeLink / Smart Life)  
https://de.aliexpress.com/item/1005006848429036.html  

(bei Ventil + Pumpe besser 2-Kanal oder 2× Relais)

---

### Strom

230V → 12V Netzteil  
https://de.aliexpress.com/item/1005010168079946.html  

DC-DC Stepdown (optional)  
https://de.aliexpress.com/item/1005008844540337.html  

---

### Frischwasser (Unterdruckventil)

https://de.aliexpress.com/item/1005005848051466.html  

---

🚰 Schlauch-Aufbau Abwasser (sehr wichtig)

Von Roborock Richtung Abfluss:

1. Roborock Ausgang  
2. Rückschlagventil  
3. Pumpe  
4. (optional zweites Rückschlagventil)  
5. Magnetventil (NC)  
6. Abfluss / Kanister  

Warum:

- Rückschlagventil = kein Rücklauf  
- Magnetventil NC = im stromlosen Zustand ZU  
- verhindert Siphon & Nachziehen  

---

⚡ Elektrik (12V)

Grundregel:

Minus bleibt immer verbunden.  
Plus wird über Relais geschaltet.

### Pumpe

+12V Netzteil → COM Relais  
NO Relais → Pumpe +  
Pumpe − → Netzteil −  

### Magnetventil (falls vorhanden)

+12V → COM  
NO → Ventil +  
Ventil − → Netzteil −  

NC wird nicht benutzt.

---

🧠 ESP32 Verdrahtung

Roborock Hall Sensor:

GND → ESP GND  
Signal → ESP GPIO4  

3.3V vom Sensor NICHT anschließen!

Optional:

GPIO5 → grüne LED (über 220–470Ω nach GND)  
GPIO6 → rote LED (über 220–470Ω nach GND)  

ESP32 bekommt Strom per USB.

---

🏠 Home Assistant Ablauf

ESP32 liefert:

- Binary Sensor: Abwasser voll  
- Text: LEER / VOLL  

Zigbee Relais:

- schaltet Pumpe  
- schaltet Magnetventil  

Automation:

VOLL → Ventil AUF → 2s → Pumpe EIN → max 2 Minuten oder bis LEER → Pumpe AUS → 2s → Ventil ZU  

---

⚠️ Sicherheit

- Alle Schlauchverbindungen mit Schellen sichern  
- Netzteil genügend Ampere wählen  
- LEDs immer mit Widerstand  
- 230V sauber isolieren  
- Umbau auf eigene Verantwortung  

---

🧪 Test

Magnet am Sensor:

- dran → LEER  
- weg → VOLL  

Wenn vertauscht: YAML invertieren.

---

Projekt von Ronny / Rinno  
Community Projekt – keine Garantie.

---

## 💬 Feedback & Ideen willkommen

Ich freue mich sehr über Feedback zu diesem Projekt 😊  
Schreibt mir gerne, was ihr davon haltet oder ob euch Verbesserungen auffallen.

Auch für Vorschläge, wie man das Setup noch sauberer, stabiler oder einfacher gestalten kann, bin ich jederzeit offen – egal ob es um:

- Elektrik  
- Schlauchführung  
- Automationen  
- ESPHome  
- Home Assistant  
- Wartung  
- oder Erweiterungen  

geht.

Das Projekt ist als Community-Projekt gedacht, und wenn ihr eigene Ideen oder Optimierungen habt, teilt sie gerne – vielleicht profitieren andere davon genauso wie ich.

Danke fürs Reinschauen und viel Spaß beim Basteln 🚀



<img width="1536" height="1024" alt="ChatGPT Image 30  Jan  2026, 09_39_17" src="https://github.com/user-attachments/assets/30071371-c5b0-459a-913c-519dd42175bc" />



<img width="1536" height="1024" alt="ChatGPT Image 30  Jan  2026, 09_39_51" src="https://github.com/user-attachments/assets/02604dfc-a019-4227-a6ae-cba2613809c7" />

