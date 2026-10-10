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
| Programm | Aktion |
|---|---|
| service | behalten |
| SYS Bestätigen, SYS war gestört | [ ] löschen (deaktivierte Hilfsskripte) |
| @HomeNotificationSender (unsichtbar, alte API) | ✅ gelöscht (10.10.2026) |
| @HomeNotificationSenderV2 | behalten (@Home-App wird genutzt) |
| urlaub ein / urlaub aus | [ ] Aktion im SONST-Zweig löschen (sonst kann der Urlaubsmodus nach CCU-Neustart „an“ gehen) |
| Urlaub gerate | [ ] löschen, ersetzt durch neuen Urlaubsmodus |

### Neuer Urlaubsmodus (`skripte/urlaub/`)
Schalter bleibt die Systemvariable `urlaub modus ein` (Favorit, @Home-App, virtuelle Taste 1).
- [ ] `urlaub_test.hms` in „Skript testen“ → „Fehlend: 0“
- [ ] Programm **Urlaubsmodus**: WENN `urlaub modus ein` bei **an** – *bei Änderung auslösen*; DANN Skript `urlaub_an.hms`; SONST Skript `urlaub_aus.hms`
- [ ] Programm **Urlaub halten**: WENN `urlaub modus ein` bei **an** – *nur prüfen* UND Zeitsteuerung Zeitintervall alle 1 Std – *bei Zeitsteuerung auslösen*; DANN Skript `urlaub_halten.hms`
- [ ] Test: Urlaub an → Heizkörperthermostate zeigen „Manu“ 19 °C; Urlaub aus → wieder „Auto“

Wandthermostate (HM-CC-TC) mit Ventil bekommen 19 °C als Solltemperatur und kehren nach dem Urlaub beim nächsten
Schaltzeitpunkt zu ihrem Wochenprogramm zurück. Heizkörperthermostate (HM-CC-RT-DN) gehen in den manuellen Betrieb
und per „Urlaub aus“ zurück in den Automatikbetrieb. Wandthermostate ohne Ventil (Zimmer 3, 5, 7) werden nicht geschaltet.

## 5. Systemvariablen (Nutzung geprüft)
Skript: [`skripte/systemvariablen_loeschen.hms`](skripte/systemvariablen_loeschen.hms) (mit Probelauf)
- [ ] `Systemvariable` – nirgends genutzt → löschen
- [ ] `Systemvariable 1` (Text „???“) – nirgends genutzt → löschen
- [ ] `Heizungsautomati` – nirgends genutzt → löschen
- [ ] `schloss` – an einen Kanal gebunden, den es nicht mehr gibt → löschen
- [ ] `svEnergyCounter_11479_CUX3700001:2`, `svEnergyCounterOldVal_11479` – unsichtbare Reste eines gelöschten Energiezählers → löschen
- [ ] `myHomeToken`, `myHomeLastPushMessage`, `myHomeLastPushMessage 1` (Alarmvariable) – HomeControl/myHome-App; im Löschskript enthalten
- `@HomeNotification` – @Home-App → behalten
- [ ] `Alarmzone 1`, `Anwesenheit` – CCU-Standard, nirgends genutzt; optional löschen
- behalten: `urlaub modus ein`, `Systemfall`; `-ActiveState`-Variablen gehören intern zu den Programmen

## 6. Firmware
- [ ] CCU2 läuft mit 2.41.5 (2018). Auf die letzte CCU2-Firmware aktualisieren (Sicherheitsupdates) – vorher Backup.
