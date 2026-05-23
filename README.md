# ACE Pro Native UI Patch für Anycubic Kobra S1 Combo / KS1C

Diese README beschreibt die **Kobra-S1-Combo-/KS1C-Anpassung** des nativen ACE-Pro-Dashboard-Patches für **Mainsail** und **Fluidd**.

<img width="910" height="745" alt="Mainsail-1" src="https://github.com/user-attachments/assets/9132e5de-f980-41ea-809f-b6846a442a56" />
<img width="961" height="842" alt="Mainsail-2" src="https://github.com/user-attachments/assets/041583d4-bfe7-498d-a3c1-8e972eb6ed0c" />

<img width="960" height="747" alt="Fluidd-1" src="https://github.com/user-attachments/assets/3ca543ac-521a-4e59-9831-f244c5d3be54" />
<img width="960" height="747" alt="Fluidd-2" src="https://github.com/user-attachments/assets/24370c1a-c800-438e-ba75-22f3e42f06c7" />

## Grundlage und Credits

Diese Version wird im Einsatz mit dem Projekt:

- **Kobra-S1/ACEPRO**
https://github.com/Kobra-S1/ACEPRO

verwendet und wurde für die hier verwendete KS1C-Konfiguration erweitert.

Das **ursprüngliche Dashboard-Layout bzw. die UI-Vorlage**, auf der dieser native Mainsail-/Fluidd-Patch aufbaut, stammt jedoch aus:

- **swilsonnc/ACEPROK1Max**
https://github.com/swilsonnc/ACEPROK1Max


Diese Vorlage war die gestalterische und strukturelle Ausgangsbasis für das ACE-Panel. Sie wird hier ausdrücklich genannt und gewürdigt.

Darauf aufbauend enthält diese KS1C-Fassung:

- natives ACE-Panel in **Mainsail**
- natives ACE-Panel in **Fluidd**
- automatische Erkennung beliebig vieler ACE-Instanzen über `ace_inventory_0`, `ace_inventory_1`, `ace_inventory_2`, …
- KS1C-Makro-Mapping
- Dryer-Synchronisierung zwischen Mainsail und Fluidd
- Spoolman Active-Spool-Sync über die RFID-/Inventory-`sku`
- update-/restore-fester Patch-Mechanismus

Offiziell getestet:
✅ Mainsail v2.17.0
✅ Fluidd v1.37.0

Notfall-Fallback vorhanden:
⚠️ Fluidd v1.36.2
---

# 1. Was diese Version kann

## Native Mainsail-/Fluidd-Integration

Der ACE-Bereich ist direkt in das jeweilige Dashboard eingebaut:

- Mainsail: natives ACE-Panel
- Fluidd: natives ACE-Panel
- verschiebbar wie ein normales Dashboard-Panel
- **kein iframe**, kein extern eingebettetes `ace.html`

---

## Dynamische ACE-Erkennung

Der Patch ist **nicht auf 2 ACE Pro begrenzt**.

Er erkennt alle ACE-Instanzen automatisch anhand der vorhandenen Variablen:

```ini
ace_inventory_0
ace_inventory_1
ace_inventory_2
ace_inventory_3
...
```

Daraus wird das Dropdown dynamisch aufgebaut:

```text
ACE 1
ACE 2
ACE 3
ACE 4
...
```

### Beispiel

Wenn in `saved_variables.cfg` vorhanden ist:

```ini
ace_inventory_0 = [...]
ace_inventory_1 = [...]
ace_inventory_2 = [...]
```

zeigt das Panel automatisch:

```text
ACE 1
ACE 2
ACE 3
```

---

## Tool-/Slot-Mapping

Das Mapping läuft generisch über:

```text
global_tool = ace_instance * 4 + local_slot
```

### Beispiele

| ACE | Instance | Local Slot | Global Tool |
|---|---:|---:|---:|
| ACE 1 | 0 | 0 | T0 |
| ACE 1 | 0 | 1 | T1 |
| ACE 1 | 0 | 2 | T2 |
| ACE 1 | 0 | 3 | T3 |
| ACE 2 | 1 | 0 | T4 |
| ACE 2 | 1 | 1 | T5 |
| ACE 2 | 1 | 2 | T6 |
| ACE 2 | 1 | 3 | T7 |
| ACE 3 | 2 | 0 | T8 |
| ACE 3 | 2 | 1 | T9 |
| ACE 3 | 2 | 2 | T10 |
| ACE 3 | 2 | 3 | T11 |

Das Panel selbst bleibt damit auch für weitere ACE-Einheiten nutzbar, sofern die zugrunde liegende ACEPRO-/Klipper-Konfiguration diese Instanzen bereitstellt.

---

# 2. Voraussetzungen

## Moonraker

In `moonraker.cfg` muss die ACE-Status-Komponente vorhanden sein:

```ini
[ace_status]
```

## Spoolman

Spoolman muss in Moonraker eingebunden sein, z. B.:

```ini
[spoolman]
server: http://192.168.x.xxx:7912
sync_rate: 5
```

## ACEPRO-Kobra-S1-Basis

Diese Anpassung setzt die Kobra-S1-ACEPRO-Struktur voraus, bei der ACE-Daten und Status über das Kobra-S1/ACEPRO-Projekt bereitgestellt werden.

---

# 3. Datenbasis in `saved_variables.cfg`

Das Panel liest die Slot-/Inventardaten aus:

```ini
ace_inventory_0
ace_inventory_1
ace_inventory_2
...
```

Beispiel:

```ini
[Variables]
ace_current_index = 1
ace_filament_pos = 'nozzle'

ace_inventory_0 = [
  {'status': 'ready', 'sku': '15', 'material': 'PETG', 'temp': 205, 'color': [255,255,255]},
  {'status': 'ready', 'sku': '11', 'material': 'PETG', 'temp': 250, 'color': [1,1,1]}
]

ace_inventory_1 = [
  {'status': 'ready', 'sku': '2', 'material': 'PLA+', 'temp': 210, 'color': [250,124,12]},
  {'status': 'ready', 'sku': '17', 'material': 'PLA', 'temp': 200, 'color': [177,190,198]}
]
```

---

# 4. ACE-Temperaturanzeige

Damit die ACE-/Chamber-Temperaturen im UI angezeigt werden, werden die ACE-Temperatursensoren verwendet:

```ini
[temperature_ace]

[temperature_sensor ace_temp_0]
sensor_type: temperature_ace
ace_instance: 0
min_temp: 0
max_temp: 70

[temperature_sensor ace_temp_1]
sensor_type: temperature_ace
ace_instance: 1
min_temp: 0
max_temp: 70
```

Für weitere ACEs entsprechend:

```ini
[temperature_sensor ace_temp_2]
sensor_type: temperature_ace
ace_instance: 2
min_temp: 0
max_temp: 70
```

---

# 5. Dryer-Funktionen

Der Dryer-Bereich liest den echten ACE-Status pro Instanz über:

```text
/server/ace/status?instance=<N>
```

Dadurch werden korrekt synchronisiert:

- Chamber-/ACE-Temperatur
- Dryer-Status
- Target-Temperatur
- Duration
- Remaining Time

## Remaining Time

Die Restzeit wird korrekt lesbar dargestellt, z. B.:

```text
3h 15m
```

---

## Cross-UI-Sync Mainsail ↔ Fluidd

Wenn in Mainsail ein Dryer gestartet wird, zeigt Fluidd denselben laufenden Zustand an.  
Wenn in Fluidd gestartet wird, zeigt Mainsail denselben Zustand an.

Synchronisiert werden bei laufendem Dryer:

- Target
- Duration
- Remaining
- Chamber-/ACE-Temperatur
- Drying-/Idle-Status

---

# 6. KS1C-Makro-Mapping

Die nativen UI-Buttons verwenden die in der KS1C-Konfiguration vorhandenen ACEPRO-Befehle.

## Tool laden

```gcode
ACE_CHANGE_TOOL TOOL=<global_tool>
```

## Park / Smart Unload

```gcode
ACE_SMART_UNLOAD TOOL=<global_tool>
```

## Spule vollständig entladen

```gcode
ACE_FULL_UNLOAD TOOL=<global_tool>
```

## Feed Assist aktivieren

```gcode
ACE_ENABLE_FEED_ASSIST INSTANCE=<ace_instance> INDEX=<local_slot>
```

## Feed Assist deaktivieren

```gcode
ACE_DISABLE_FEED_ASSIST INSTANCE=<ace_instance> INDEX=<local_slot>
```

## Feed

```gcode
ACE_FEED INSTANCE=<ace_instance> INDEX=<local_slot> LENGTH=<mm> SPEED=<mm_s>
```

## Retract

```gcode
ACE_RETRACT INSTANCE=<ace_instance> INDEX=<local_slot> LENGTH=<mm> SPEED=<mm_s>
```

## Slot-Daten setzen

```gcode
ACE_SET_SLOT INSTANCE=<ace_instance> INDEX=<local_slot> MATERIAL=<material> TEMP=<°C> COLOR=<R,G,B>
```

## Dryer starten

```gcode
ACE_START_DRYING INSTANCE=<ace_instance> TEMP=<°C> DURATION=<minutes>
```

## Dryer stoppen

```gcode
ACE_STOP_DRYING INSTANCE=<ace_instance>
```

---

# 7. Spoolman Active-Spool-Sync

Diese Version synchronisiert die aktive Spule in Spoolman automatisch mit dem aktuell geladenen ACE-Tool.

## Datenquelle

Der Patch beobachtet:

```ini
ace_current_index
```

und liest dazu die passende `sku` aus dem entsprechenden `ace_inventory_N`.

## Beispiel 1

```ini
ace_current_index = 1
ace_inventory_0[1].sku = '11'
```

Dann wird gesendet:

```gcode
SET_ACTIVE_SPOOL ID=11
```

## Beispiel 2

```ini
ace_current_index = 6
ace_inventory_1[2].sku = '20'
```

Dann wird gesendet:

```gcode
SET_ACTIVE_SPOOL ID=20
```

## Bei Unload

Wenn kein Tool aktiv ist:

```ini
ace_current_index = -1
```

wird:

```gcode
CLEAR_ACTIVE_SPOOL
```

gesendet.

## Wichtige Hinweise

- Die `sku` muss numerisch sein, weil sie direkt als Spoolman-Spulen-ID verwendet wird.
- Der Sync wird nur bei einer erkannten Änderung des aktiven Tools ausgelöst, nicht bei jedem Status-Poll erneut.
- Da der Sync im nativen UI-Patch läuft, muss mindestens **Mainsail oder Fluidd mit sichtbarem ACE-Panel geöffnet** sein.

---

# 8. Fluidd Dropdown Dark Fix

In Fluidd waren die ACE-Dropdown-Optionen zeitweise hell hinterlegt und schlecht lesbar.  
Diese Version enthält nur für Fluidd einen kleinen Style-Fix:

- dunkler Dropdown-Hintergrund
- gut lesbare helle Schrift
- optisch passend zum Dark Theme

---

# 9. Installation

## ZIP entpacken

```bash
mkdir -p ~/ACE-Mainsail-Fluidd-Patch
cd ~/ACE-Mainsail-Fluidd-Patch
unzip ace_ui_native_safe_update_kit_v2_spoolman_sync_restore_safe_v4_fluidd_dropdown_dark_fix.zip
```

## Patch anwenden

```bash
python3 apply_ace_ui_native_patch.py --ui both
```

Oder einzeln:

```bash
python3 apply_ace_ui_native_patch.py --ui mainsail
python3 apply_ace_ui_native_patch.py --ui fluidd
```

## Browser hart neu laden

```text
Strg + Shift + R
```

---

# 10. Verhalten nach Mainsail-/Fluidd-Update oder Neuinstallation

Der Patch ist so aufgebaut, dass er auch nach Update oder frischer Neuinstallation erneut angewendet werden kann.

```bash
python3 apply_ace_ui_native_patch.py --ui both
```

macht intern:

1. Prüfen, ob Mainsail/Fluidd bereits gepatcht sind
2. Patch anwenden, wenn der aktuelle Frontend-Build passt
3. Falls der Build nicht direkt patchbar ist:
   - Restore der mitgelieferten, bekannten funktionierenden nativen UI-Version
   - dabei bleibt `config.json` erhalten

---

# 11. Sicheres UI-Update

Empfohlen:

```bash
./ace_ui_safe_update.sh --ui both
```

Das Script:

1. aktualisiert Mainsail und Fluidd über Moonraker
2. versucht danach erneut den nativen Patch
3. stellt nur die betroffene UI zurück, falls ein neuer Build nicht patchbar ist

Einzeln:

```bash
./ace_ui_safe_update.sh --ui mainsail
./ace_ui_safe_update.sh --ui fluidd
```

---

# 12. Troubleshooting

## ACE-Panel fehlt nach Update

```bash
python3 apply_ace_ui_native_patch.py --ui both
```

## Nur Fluidd fehlt

```bash
python3 apply_ace_ui_native_patch.py --ui fluidd
```

## Nur Mainsail fehlt

```bash
python3 apply_ace_ui_native_patch.py --ui mainsail
```

## Keine sichtbare Änderung

Browser hart neu laden:

```text
Strg + Shift + R
```

## Spoolman schaltet nicht um

Prüfen:

1. `ace_current_index` ändert sich wirklich
2. passende `sku` ist numerisch vorhanden
3. mindestens ein Dashboard mit ACE-Panel ist geöffnet
4. `SET_ACTIVE_SPOOL ID=<ID>` existiert im System

---

# 13. Bezug zu den Projekten / Credits

Diese KS1C-Version wird zusammen mit:

- **Kobra-S1/ACEPRO**

verwendet und ist auf dessen Kobra-S1-/KS1C-Umgebung abgestimmt.

Für das **Panel-Layout und die ursprüngliche Dashboard-Vorlage** wurde:

- **swilsonnc/ACEPROK1Max**

als Ausgangsbasis genutzt. Das dortige UI-Konzept war die Vorlage, aus der dieser native Mainsail-/Fluidd-Patch weiterentwickelt wurde.

Diese Fassung erweitert und verändert die Vorlage unter anderem um:

- KS1C-spezifisches Makro-Mapping
- dynamische Erkennung mehrerer ACE-Instanzen
- nativen Mainsail-/Fluidd-Patch ohne iframe
- Dryer-Cross-UI-Sync
- Remaining-Time-/Dryer-State-Fixes
- Spoolman Active-Spool-Sync per SKU
- Restore-/Update-Mechanismus
- Fluidd Dropdown Dark Fix
