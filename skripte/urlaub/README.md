# Urlaubsmodus – Aufbau und Funktionsweise

CCU2, Firmware 2.61.7, BidCos-RF über zwei RF-LAN-Gateways. Heizung gemischt aus
HM-CC-TC + HM-CC-VD (alt) und HM-CC-RT-DN. Eingerichtet und getestet am 10.10.2026.

## Ausgangslage (vorher)

- Programm „Urlaub gerate“: bei `urlaub modus ein` = an → `SETPOINT` 19 °C auf alle HM-CC-TC.
  - In Zimmer 3, 4, 7 regelt aber ein HM-CC-RT-DN mit eigenem Sollwert. Der HM-CC-TC liefert dort nur
    über die Direktverknüpfung `WEATHER → WEATHER_RECEIVER` (TC:1 → RT-DN:1) die Raumtemperatur.
    Der Sollwert wird nicht übertragen → Urlaub hatte dort keine Wirkung.
  - Kein Zurücksetzen nach dem Urlaub.
- Programme „urlaub ein“ / „urlaub aus“ (virtuelle Taste BidCoS-RF:1 kurz bzw. lang) hatten SONST-Zweige,
  die die Systemvariable jeweils gegenteilig setzten. Beim CCU-Start werden alle Programme ausgewertet →
  beide SONST-Zweige liefen → Urlaubsmodus konnte nach einem Neustart von selbst auf „an“ gehen.

## Aufbau (jetzt)

**Schalter:** Systemvariable `urlaub modus ein` (Logikwert). Bedienung über @Home-App, Favoriten
oder virtuelle Taste 1 (kurz = an, lang = aus). Die SONST-Zweige der Tastenprogramme wurden entfernt.

**Ein Programm „Urlaubsmodus“:**

```
WENN  Systemzustand  urlaub modus ein  bei an   bei Änderung auslösen
ODER  Zeitsteuerung  ganztägig, Zeitintervall 1 Std   zu Zeitpunkten auslösen
DANN  Skript urlaubsmodus.hms   sofort
SONST Skript urlaubsmodus.hms   sofort
```

DANN und SONST enthalten **dasselbe Skript**. Das Skript entscheidet selbst anhand der Systemvariable,
was zu tun ist – damit ist es egal, über welchen Zweig es ausgeführt wird.

## Skript-Logik (`urlaubsmodus.hms`)

```
urlaub = Wert von "urlaub modus ein"
neu    = Systemvariable in den letzten 180 s geändert   (Umschalten vs. stündlicher Lauf)

wenn urlaub:
    11 × HM-CC-TC (nur die mit Ventil):  BidCos-RF.<SN>:2.SETPOINT = 16      ← jeder Lauf
    5  × HM-CC-RT-DN:  wenn neu ODER CONTROL_MODE ≠ 1 (Manu):
                       BidCos-RF.<SN>:4.MANU_MODE = 16
sonst:
    5  × HM-CC-RT-DN:  wenn neu UND CONTROL_MODE ≠ 0 (Auto):
                       BidCos-RF.<SN>:4.AUTO_MODE = true
```

### Warum so

| Entscheidung | Grund |
|---|---|
| HM-CC-TC stündlich neu beschreiben | Ein gesetzter `SETPOINT` gilt im Automatikbetrieb nur bis zum nächsten Schaltzeitpunkt des Wochenprogramms. Die Betriebsart (`MODE_TEMPERATUR_REGULATOR`) ist ein MASTER-Parameter und per HM-Script nicht setzbar. |
| HM-CC-RT-DN über `MANU_MODE` / `AUTO_MODE` | Echte Betriebsart statt nur Sollwert → hält ohne Wiederholung, und „Urlaub aus“ kann sauber zurück auf Automatik. |
| RT-DN nur bei Bedarf ansprechen | RT-DN werden per Burst geweckt → kostet Batterie und Duty Cycle. Stündlich wird nur nachgesteuert, falls jemand die Betriebsart geändert hat. |
| Umschalten über Zeitstempel erkennen (`neu`) | Ein Skript für beide Zweige und den Zeittrigger. Erste Version verglich die Solltemperatur – das schlug fehl, wenn bei offenem Fenster 12 °C (Fenster-auf-Temperatur) gemeldet wurden; die Thermostate blieben dann in „Manu“ hängen. |
| Adressierung über `BidCos-RF.<Seriennummer>:<Kanal>.<Datenpunkt>` | Unabhängig von Geräte- und Kanalnamen (die wurden im Zuge des Aufräumens umbenannt). |
| HM-CC-TC ohne Ventil (Zimmer 3, 5, 7) ausgelassen | Die liefern nur Temperatur an den RT-DN; ein Sollwert dort hätte keine Wirkung. |

### Abgedeckte Geräte

| Typ | Räume |
|---|---|
| HM-CC-TC (SETPOINT) | Bad Gäste, Flur, Girls Bad, Reci, Solarium, Waschraum, Zimmer 1, 2, 4, 6, 8 |
| HM-CC-RT-DN (MANU/AUTO) | Küche, Zimmer 3, 4, 5, 7 |

Die Fenster-Absenkung (Direktverknüpfung Fensterkontakt → TC:3 bzw. RT-DN:3, 12 °C) läuft unabhängig
davon weiter und hat Vorrang.

## Bekannte Einschränkungen

- Nach dem Urlaub kehren die HM-CC-TC erst beim nächsten Schaltzeitpunkt zum Wochenprogramm zurück.
- Wer innerhalb von 3 Minuten nach dem Umschalten von Hand etwas ändert, kann vom Skript überschrieben werden.
- Beim CCU-Neustart wird das Programm ausgewertet. Ist der Urlaub aus, gilt das als „neu“ → RT-DN, die
  von Hand auf Manu standen, gehen zurück auf Auto.
- Nach einem CCU-Neustart kennt die CCU die `SETPOINT`-Werte der HM-CC-TC erst, wenn diese sich melden
  (Anzeige 0.0) – reine Anzeige, die Geräte regeln normal.

## Test (10.10.2026)

- Urlaub an → alle 11 HM-CC-TC 16 °C, RT-DN „Manu 16 °C“ (Zimmer 4/5 „Manu 12 °C“ wegen offenem Fenster).
- Urlaub aus → alle RT-DN zurück auf „Auto“.

## Dateien

| Datei | Zweck |
|---|---|
| `urlaubsmodus.hms` | das Skript für DANN und SONST |
| `urlaub_test.hms` | prüft, ob alle Datenpunkte existieren (schaltet nichts) |
| `urlaub_anzeige.hms` | zeigt Urlaubsmodus, Sollwerte und Betriebsarten (schaltet nichts) |
| `heizkoerper_auf_auto.hms` | Notfall: alle RT-DN sofort auf Auto |
| `alte_programme_loeschen.hms` | hat „Urlaub gerate“, „SYS Bestätigen“, „SYS war gestört“ entfernt |
