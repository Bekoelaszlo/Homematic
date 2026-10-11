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
| SYS Bestätigen, SYS war gestört | ✅ gelöscht |
| @HomeNotificationSender (unsichtbar, alte API) | ✅ gelöscht (10.10.2026) |
| @HomeNotificationSenderV2 | behalten (@Home-App wird genutzt) |
| urlaub ein / urlaub aus | ✅ SONST-Aktionen gelöscht |
| Urlaub gerate | ✅ gelöscht, ersetzt durch neuen Urlaubsmodus |

### Neuer Urlaubsmodus (`skripte/urlaub/`)
Schalter bleibt die Systemvariable `urlaub modus ein` (Favorit, @Home-App, virtuelle Taste 1). Alle Räume → 16 °C.
- [x] `urlaub_test.hms` → „Fehlend: 0“
- `urlaub_anzeige.hms` zeigt jederzeit den Zustand (schaltet nichts)
- [x] `alte_programme_loeschen.hms` (Probelauf, dann `loeschen = true`) – löscht Urlaub gerate, SYS Bestätigen, SYS war gestört
- [x] In „urlaub ein“ und „urlaub aus“ die Aktion im SONST-Zweig löschen
- [x] Ein Programm **Urlaubsmodus**: WENN `urlaub modus ein` bei **an** *bei Änderung auslösen* ODER Zeitsteuerung Zeitintervall 1 Std *bei Zeitsteuerung auslösen*; DANN **und** SONST: Skript `urlaubsmodus.hms`
- [x] Test: Urlaub an → Heizkörperthermostate „Manu“ 16 °C; Urlaub aus → „Auto“ (10.10.2026, bestanden)

Das Skript entscheidet selbst anhand der Systemvariable. Wandthermostate (HM-CC-TC) bekommen im Urlaub stündlich 16 °C
(manuell gesetzte Werte gelten bei ihnen nur bis zum nächsten Schaltzeitpunkt) und folgen danach wieder dem Wochenprogramm.
Heizkörperthermostate (HM-CC-RT-DN) werden nur bei Bedarf geschaltet (Batterie): Urlaub → manuell 16 °C, danach → Automatik.

## 5. Systemvariablen (Nutzung geprüft)
Skript: [`skripte/systemvariablen_loeschen.hms`](skripte/systemvariablen_loeschen.hms) (mit Probelauf)
- [x] `Systemvariable` – nirgends genutzt → gelöscht 10.10.2026
- [x] `Systemvariable 1` (Text „???“) – nirgends genutzt → gelöscht 10.10.2026
- [x] `Heizungsautomati` – nirgends genutzt → gelöscht 10.10.2026
- [x] `schloss` – an einen Kanal gebunden, den es nicht mehr gibt → gelöscht 10.10.2026
- [x] `svEnergyCounter_11479_CUX3700001:2`, `svEnergyCounterOldVal_11479` – unsichtbare Reste eines gelöschten Energiezählers → gelöscht 10.10.2026
- [x] `myHomeToken`, `myHomeLastPushMessage`, `myHomeLastPushMessage 1` (Alarmvariable) – HomeControl/myHome-App → gelöscht 10.10.2026
- `@HomeNotification` – @Home-App → behalten
- [x] `Alarmzone 1`, `Anwesenheit` – CCU-Standard, nirgends genutzt → gelöscht 10.10.2026
- behalten: `urlaub modus ein`, `Systemfall`; `-ActiveState`-Variablen gehören intern zu den Programmen

## 6. Firmware
- [x] Firmware von 2.41.5 auf **2.61.7** aktualisiert (11.10.2026)
- [ ] Nach dem Update prüfen: `status_gesamt.hms`, @Home-Push, CloudMatic
- Installiertes Add-on: **CloudMatic** (Fernzugriff/Push) – nach dem Update prüfen, ob es weiter läuft.

## 7. Servicemeldungen (Stand 10.10.2026, 21:30)
UNREACH seit 20:52 (alle drei innerhalb von 2 Sekunden), alles Ventilantriebe HM-CC-VD:
- [ ] Zimmer 6 Ventil (JEQ0377802)
- [ ] Girls Bad Ventil (KEQ0012342)
- [ ] Zimmer 8 Ventil 1 (IEQ0171850)

Ein HM-CC-VD spricht nur mit seinem Wandthermostat; die CCU hört mit. UNREACH heißt zuerst nur: die CCU hat ihn
eine Weile nicht gehört. Prüfen: Display am Ventil (Fehlercode?), Heizkörper warm/regelt?, Batterien.
`skripte/status_gesamt.hms` bzw. `skripte/servicemeldungen.hms` morgen erneut ausführen.

Protokoll vom 11.10. 03:35: Die Wandthermostate senden an diese drei Ventile, die Ventile antworten nicht → Batterien tauschen.
Nach dem Firmware-Update stehen die Servicemeldungen zunächst auf 0 – die Meldungen kommen zurück, solange die Ventile nicht antworten.

Neu seit 22:15:
- [ ] Girls Bad Wandthermostat (KEQ0010690): **Batterie leer** (LOWBAT) → Batterien tauschen. Kann auch die Funkstörung zum Girls Bad Ventil erklären.

## 8. Funk: zwei LAN-Gateways (keine Repeater)
Die CCU funkt über zwei **HomeMatic RF-LAN-Gateways** (HM-LGW-O-TW-W-EU); das eingebaute Funkmodul der CCU ist keinem Gerät zugeordnet.

| Gateway | Seriennummer | IP | Geräte |
|---|---|---|---|
| RF-LAN Gateway | KEQ1064927 | 192.168.2.61 | 32 – Bad Gäste, Zimmer 1–8 (inkl. Heizkörperthermostate 3/4/5/7), Zimmer 2 Rollo |
| RF-LAN Gateway 2 | MEQ0461078 | 192.168.2.62 | 23 – Flur, Girls Bad, Reci, Solarium, Waschraum, Küche, Rollos Reci/Solarium, Fernbedienung |

Die 3 stummen Ventile: Zimmer 6 Ventil + Zimmer 8 Ventil 1 → Gateway 1; Girls Bad Ventil → Gateway 2.
- [ ] Status beider Gateways prüfen (Einstellungen → Systemsteuerung → LAN Gateway Konfiguration)
- [ ] Ggf. die 3 Ventile dem anderen Gateway zuordnen bzw. Roaming aktivieren

## 9. Girls Bad Ventil tauschen
Altes Ventil KEQ0012342: Notstellung 15 % mit Antennensymbol – keine Verbindung zum Wandthermostat trotz neuer Batterien und neuer Verknüpfung.
Ersatz: **IEQ0078470** (HM-CC-VD). Prüfen mit `skripte/girlsbad_tausch_check.hms`.
- [x] Neues Ventil angelernt, heißt „Girls Bad Ventil“
- [x] Konfiguration übertragen (CONFIG_PENDING false)
- [x] Montiert
- [x] Erste Ventilmeldung 20:24 – Girls Bad läuft wieder ✅
- [x] Altes Gerät KEQ0012342 aus der CCU entfernt
- Danach ggf. Zimmer 6 Ventil (JEQ0377802) und Zimmer 8 Ventil 1 (IEQ0171850) genauso (beide zeigen 15 + Antennensymbol?)

## 10. Zimmer 8 Ventil 1 getauscht
- [x] Altes Ventil IEQ0171850 (Notstellung 15 %) durch HEQ0238742 ersetzt
- [x] Erste Meldung 21:02 ✅
- [x] Fenster-Test Zimmer 8 (11.10. 21:20): Ventile 1 und 2 schließen auf 0 % – Solltemperatur 12 °C noch nicht per `zimmer8_check.hms` bestätigt
- [x] Zimmer 6 Ventil JEQ0377802 durch **JEQ0045982** ersetzt
- [x] Zimmer 8 Ventil 1 meldet sich (21:02, 13 %) ✅
- [x] Zimmer 6 Ventil JEQ0045982 meldet sich (21:10, 47 %) ✅ – alle 13 Ventile laufen, Servicemeldungen 0

## 11. Fenster-Absenkung (Test 11.10.2026, 22:35, `skripte/fenster_check.hms`)
- [x] Zimmer 8: Fenster offen → Wandthermostat 12 °C ✅ (Problem von gestern behoben)
- [x] Zimmer 5: Fenster offen → Heizkörperthermostat 12 °C, zurück nach 11 s ✅
- [x] **Zimmer 6:** Fenstersignal kam beim Wandthermostat nicht an → Verknüpfung per Skript neu angelegt (`zimmer6_fenster_neu_verknuepfen.hms`)
- [ ] Zimmer 6: Taste am Fensterkontakt, dann Fenster-Test wiederholen
- [ ] Bad Gäste, Girls Bad: kein Fensterkontakt vorhanden (optional nachrüsten)
