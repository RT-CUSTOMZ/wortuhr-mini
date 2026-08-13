# Einleitung

Kleinere Version der Original Campuswochen Wortuhr. 
Das Buchstabenlayout dieser WortUhr ist angelehnt an das Layout, welches von Frank Meyer verwendet wird.

Der Aufbau ist nun kompatibel mit 60 LED/m WS2812B/SK6812 LED Streifen. Um das zu erreichen ist die Rahmengröße reduziert auf 35cm Innenmaß. Hierzu eignet sich zum Beispiel der IKEA SANNAHED 35x35cm Rahmen. Die Steuerplatine ist auf das wesentliche beschränkt, die Stromversorgung übernimmt ein einziges generisches DCDC XL4015 Netzteil. Es wird weiterhin über einen Rundbuchsenanschluss mit 19-21V eingespeist. Um die Bestellung möglichst einfach zu gestalten werden die Power und Steuerplatine auf eine gemeinsame Platine gepaneled. (Siehe Kombiplatine THT)

[Kombilatine](Bauanleitung/FullView.png)

Die Software und das Layout der Matrix von Frank Meyer wurden minimal angepasst.

Originaler Artikel: https://www.mikrocontroller.net/articles/WordClock_mit_WS2812

Originaler Aufbau: https://www.mikrocontroller.net/articles/Tutorial_-_Aufbau_WordClock_mit_WS2812


# Inhaltsverzeichnis
- [Bauanleitung](#bauanleitung)
- [Hardware](#hardware)
- [Software](#software)
  - [Hochladen der Icons]

# Bauanleitung
1. Lötübungen (nach Ermessen)
2. Komponenten mit der [BOM](/Platinen/BOM/ibom.html) bestücken. (pro Uhr ca. 5h)
3. Led Streifen (WS2812b) schneiden (18 Leds lang, 19 Streifen pro Uhr) und Stiftleisten anlöten. (pro Uhr ca. 2h)
4. [Software](#software) Flashen. (pro Uhr ca. 30min)
5. [Pappgitter](/Pappgitter/) lasern und zusammenstecken. (pro Uhr ca. 3h)
6. Folie plottern [WortuhrBuchstaben.svg](/Buchstabenmatrix/WortuhrBuchstaben.svg). (pro Uhr ca. 30min)
7. Folie auf Bilderrahmen übertragen. (pro Uhr ca. 3h)
8. Alle Einzelteile zusammenfügen. (pro Uhr ca. 1h)
9. Die Dateien auf den ESP hochladen

# Hardware:
- [IKEA SANNAHED 35x35cm Rahmen](https://www.ikea.com/de/de/p/sannahed-rahmen-schwarz-80459117/)
- STM32F103C8T6 BluePill
- ESP8266-12F
- RTC: DS3231
- DC-DC Stepdown (12V → 5V) Mini 560
- WS2812b Led Streifen (60 LED/m, 342 Leds pro Uhr)

# Software:

### Erforderliche Hardware zum Flashen:
* **WortUhr-Mini** Board
* **ST-Link V2 USB-Adapter** (zum Flashen des STM32)
* **USB-to-UART / Serial Adapter** (z. B. FTDI / CP2102, zum Flashen des ESP8266)

### Erforderliche Software & Dateien:
* **STM32CubeProgrammer:** [ST-Link Flashtool herunterladen](https://www.st.com/en/development-tools/stm32cubeprog.html)
* **Firmware STM32:** [wc24h-stm32f103-ws2812-grb-backwards.hex](/software/wc24h-stm32f103-ws2812-grb-backwards.hex)
* **Web-Flasher für ESP:** [ESPHome Web Flasher](https://web.esphome.io/)
* **Firmware ESP8266:** [ESP-WordClock-4M.bin](/software/ESP-WordClock-4M.bin)

---

## TEIL 1: Flashen des STM32

Der STM32 steuert die LED-Matrix an. Er muss programmiert werden, **bevor** der ESP8266 aufgesteckt und geflasht wird.

### 1.1 Hardware verbinden
1. Nimm den **STM32 außerhalb der WortUhr** (noch nicht auf die Platine stecken).
2. Verbinde den STM32 mit dem **ST-Link V2 Adapter**:

### 1.2 Firmware flashen
1. Verbinde den ST-Link V2 Adapter mit dem PC.
2. Öffne den **STM32CubeProgrammer**.
3. Klicke oben rechts auf **Connect** (Verbindung über ST-LINK).
4. Öffne die Firmware-Datei [wc24h-stm32f103-ws2812-grb-backwards.hex](/Software/wc24h-stm32f103-ws2812-grb-backwards.hex).
5. Starte den Programmiervorgang (**Download** / **Program**).

### 1.3 Funktionstest (Ohne ESP8266)
1. Trenne die ST-Link-Verbindung.
2. Stecke den geflashten **STM32 ohne den ESP8266** in die WortUhr-Mini.
3. Schalte die Stromversorgung der WortUhr-Mini ein.
4. **Ergebnis:** Erscheint auf der Matrix ein **rotes "X"**, war die Programmierung des STM32 erfolgreich!

> **Hinweis zu künftigen Updates:** Um die STM32-Firmware zu einem späteren Zeitpunkt zu aktualisieren, muss eine Verbindung zwischen **J1** und einem **mittleren Pin der BOOT-Pins** auf dem STM32 hergestellt werden.

---

## TEIL 2: Flashen des ESP8266 über die WortUhr

Nachdem der STM32 erfolgreich vorbereitet ist, wird das ESP8266-WLAN-Modul für die Zeit-Synchronisation und Steuerung geflasht.

### 2.1 Hardware vorbereiten & verkabeln
1. Stecke den **ESP8266** in den dafür vorgesehenen Sockel der WortUhr-Mini (der STM32 bleibt installiert).
2. Verbinde den **USB-UART-Adapter** mit der WortUhr-Mini / dem ESP8266:

| USB-UART Adapter | WortUhr-Mini / ESP8266 |
| :---: | :---: |
| **RX** | **RX** |
| **TX** | **TX** |
| **VCC** | **VOUT** |
| **GND** | **GND** |

3. Setze den **Jumper** auf dem Board in die Stellung **`Prog`** (Programmiermodus).

### 2.2 Programmiermodus aktivieren
1. Verbinde den USB-UART-Adapter mit dem PC.
2. Setze **einen** Jumper auf dem STM32 auf BOOT-0 auf der Seite von Pin `B11`
3. Halte den Taster **`SW1` gedrückt**.
4. Drücke währenddessen einmal kurz die **`RESET`**-Taste auf dem STM32.
5. Sobald die **grüne LED auf dem STM32 dauerhaft leuchtet**, kannst du **`SW1` loslassen**. Das Board befindet sich jetzt im Flash-Modus.

### 2.3 ESP8266 Firmware flashen
1. Öffne im Browser die Seite **[web.esphome.io](https://web.esphome.io/)**.
2. Klicke auf **Connect** und wähle den COM-Port deines USB-UART-Adapters aus.
3. Wähle die Datei [ESP-WordClock-4M.bin](/Software/ESP-WordClock-4M.bin) aus und starte den Flash-Vorgang.

---

## TEIL 3: Abschluss & Inbetriebnahme

1. Drücke nach dem erfolgreichen Flashen erneut die **`RESET`**-Taste auf dem STM32 (oder trenne kurz die Stromversorgung).
2. Der ESP8266 startet neu und macht sein WLAN auf.
3. **Erfolgskontrolle:** Sobald die zugewiesene **IP-Adresse** als Laufschrift über das Display der WortUhr-Mini durchläuft, ist die Wortuhr erfolgreich eingerichtet!

> **Wichtig nach dem Flashen:** Stecke den Jumper nach Beendigung des Flashens für den Normalbetrieb wieder auf die Ausgangsposition (`run`) zurück.


## Teil 4: Hochladen der config-Dateien

[Hochladen-Anleitung](/Bilder/Wordclock_config-files.png)

1. Auf der Webseite der Uhr auf den Menüpunktk "SPIFFS" gehen
2. Unten auf der Seite die Dateien wc24h-icon.txt, wc24h-weather.txt & wc24h-tables-local.txt **einzeln** hochladen.




Copyright (c) 2014-2019 Frank Meyer - frank(at)fli4l.de

This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 2 of the License, or (at your option) any later version.
