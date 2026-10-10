# CCU2 aufräumen – Checkliste

Stand: 10.10.2026 · 54 Funkgeräte + 3 Systemgeräte · alle erreichbar, keine leere Batterie, nichts mit CONFIG_PENDING.

## 0. Vorher
- [ ] Systemsicherung (.sbk) erstellen: *Einstellungen → Systemsteuerung → Sicherheit*

## 1. Geräte & Kanäle umbenennen
Plan: [`bestand/umbenennung.csv`](bestand/umbenennung.csv) · Skript: [`skripte/umbenennen.hms`](skripte/umbenennen.hms)

Schema: `<Raum> Wandthermostat | Heizkörperthermostat | Ventil | Fensterkontakt [n] | Rollo Fenster/Tür`,
Kanäle der Thermostate mit Zusatz (`Temperatur`, `Regler`, `Fensterempfang`, …), Fernbedienung `FB <Raum> Rollo … hoch/runter`.
Damit sind auch erledigt: Tippfehler `Washraum 1.3`, doppeltes Leerzeichen `Waschraum  3.2`, Kanäle mit Werksnamen.

- [ ] Prüfen, ob Programme **HM-Skripte** enthalten, die Geräte über den Namen ansprechen (`dom.GetObject("…")`) – die würden nach dem Umbenennen nicht mehr funktionieren. Programme mit normalen Bedingungen/Aktionen sind nicht betroffen (Verknüpfung über ID).
- [x] Probelauf (`aendern = false`), Ausgabe prüfen – 172/172 gefunden
- [x] Echtlauf (`aendern = true`) – „Umbenannt: 172“ (10.10.2026)
- [x] Kontrolle mit `skripte/export_kanaele.hms` – alle Namen übernommen

Nicht umbenannt: Systemgeräte und virtuelle Tasten (`urlaub:ein`, `urlaub:aus`) – evtl. von der myHome-App oder Programmen per Name genutzt.

## 2. Räume
- [ ] Zimmer 5: „Zimmer 5 Wandthermostat Regler“ und „… Fensterempfang“ dem Raum Zimmer 5 zuordnen
- [ ] Optional: Heizkörperthermostat-Kanäle 1, 2, 3, 5, 6 in Zimmer 3 und 5 dem Raum zuordnen (wie in Küche/Zimmer 4/7) – rein kosmetisch
- [ ] Fernbedienungs-Tasten 11/12 unbelegt (ok)
- [ ] Raum „URLAUB“ → besser Gewerk „Szenen/Zentrale“, Raum löschen (optional)

## 3. Heizung: Wandthermostat + Heizkörperthermostat ✅ erledigt
Zimmer 3, 4, 5, 7: HM-CC-TC liefert per Direktverknüpfung „Wettersensor → Heizungsthermostat“ (TC:1 → RT-DN:1)
die Raumtemperatur an den HM-CC-RT-DN; Fensterkontakt → RT-DN:3 überall vorhanden. In Zimmer 4 regelt der TC zusätzlich das Ventil
(Sollwert/Zeiten an TC und RT-DN gleich halten).

| Raum | Temperatur TC → RT-DN | Fenster → RT-DN |
|---|---|---|
| Zimmer 3 | JEQ0235880:1 → OEQ1257212:1 ✅ | OEQ2246622:1 → OEQ1257212:3 ✅ |
| Zimmer 4 | JEQ0121520:1 → REQ0381230:1 ✅ | OEQ2245291:1 → REQ0381230:3 ✅ |
| Zimmer 5 | JEQ0457714:1 → LEQ1287076:1 ✅ | NEQ1156915:1 → LEQ1287076:3 ✅ |
| Zimmer 7 | JEQ0044237:1 → REQ0381224:1 ✅ | MEQ1831605:1 → REQ0381224:3 ✅ |

- [ ] Optional: Verknüpfungsnamen nach dem Umbenennen anpassen (z. B. „Zimmer 3 2 mit HM-CC-RT-DN OEQ1257212:1“)

## 4. Programme (ausgewertet aus Backup, Details: `bestand/programme_2026-10-10.txt`)
| Programm | Was es tut | Empfehlung |
|---|---|---|
| service | Servicemeldungen > 0 → `Systemfall` = true, sonst false | behalten |
| SYS Bestätigen | Skript: bestätigt STICKY_UNREACH automatisch (deaktiviert) | [ ] löschen |
| SYS war gestört | Skript: fragt nicht erreichbare Geräte erneut ab (deaktiviert) | [ ] löschen |
| urlaub ein | virt. Taste 1 kurz → `urlaub modus ein` = an; **SONST → aus** | [ ] SONST-Zweig entfernen |
| urlaub aus | virt. Taste 1 **lang** → `urlaub modus ein` = aus; **SONST → an** | [ ] SONST-Zweig entfernen |
| Urlaub gerate | `urlaub modus ein` = an → alle Regler 19 °C | [ ] Heizkörperthermostate Zimmer 3, 4, 7 ergänzen |
| @HomeNotificationSender (unsichtbar) | Push über @Home-App, alte API | [ ] löschen (Duplikat von V2) |
| @HomeNotificationSenderV2 (unsichtbar) | Push über @Home-App, API v3 | behalten, falls @Home-App genutzt |

Problem SONST-Zweige: Beim CCU-Start werden beide Programme geprüft, beide Bedingungen sind falsch →
„urlaub ein“ setzt aus, „urlaub aus“ setzt an. Je nach Reihenfolge steht der Urlaubsmodus nach einem Neustart auf **an**.

Problem Urlaub gerate: In Zimmer 3, 4, 7 regelt der Heizkörperthermostat mit eigenem Sollwert, das Programm setzt dort aber nur den Wandthermostat.
Außerdem kein Zurücksetzen nach dem Urlaub (im Automatikbetrieb gilt der gesetzte Wert nur bis zum nächsten Schaltzeitpunkt).

## 5. Systemvariablen (Nutzung geprüft)
Skript: [`skripte/systemvariablen_loeschen.hms`](skripte/systemvariablen_loeschen.hms) (mit Probelauf)
- [ ] `Systemvariable` – nirgends genutzt → löschen
- [ ] `Systemvariable 1` (Text „???“) – nirgends genutzt → löschen
- [ ] `Heizungsautomati` – nirgends genutzt → löschen
- [ ] `schloss` – an einen Kanal gebunden, den es nicht mehr gibt → löschen
- [ ] `svEnergyCounter_11479_CUX3700001:2`, `svEnergyCounterOldVal_11479` – unsichtbare Reste eines gelöschten Energiezählers → löschen
- [ ] `myHomeToken`, `myHomeLastPushMessage`, `myHomeLastPushMessage 1` (Alarmvariable) – HomeControl/myHome-App; nur behalten, wenn genutzt
- [ ] `@HomeNotification` – @Home-App; nur behalten, wenn genutzt
- [ ] `Alarmzone 1`, `Anwesenheit` – CCU-Standard, nirgends genutzt; optional löschen
- behalten: `urlaub modus ein`, `Systemfall`; `-ActiveState`-Variablen gehören intern zu den Programmen

## 6. Firmware
- [ ] CCU2 läuft mit 2.41.5 (2018). Auf die letzte CCU2-Firmware aktualisieren (Sicherheitsupdates) – vorher Backup.
