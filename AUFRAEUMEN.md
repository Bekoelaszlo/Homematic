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
- [ ] Probelauf (`aendern = false`), Ausgabe prüfen
- [ ] Echtlauf (`aendern = true`)

Nicht umbenannt: Systemgeräte und virtuelle Tasten (`urlaub:ein`, `urlaub:aus`) – evtl. von der myHome-App oder Programmen per Name genutzt.

## 2. Räume
- [ ] Zimmer 5: Wandthermostat-Kanäle `:2` (Regler) und `:3` sind keinem Raum zugeordnet
- [ ] Fernbedienungs-Tasten 11/12 unbelegt (ok)
- [ ] Raum „URLAUB“ → besser Gewerk „Szenen/Zentrale“, Raum löschen (optional)

## 3. Heizung: alte Wandthermostate ohne Ventil (vor Ort prüfen)
| Raum | HM-CC-TC | Ventil | HM-CC-RT-DN |
|---|---|---|---|
| Zimmer 3 | JEQ0235880 | – | OEQ1257212 |
| Zimmer 5 | JEQ0457714 | – | LEQ1287076 |
| Zimmer 7 | JEQ0044237 | – | REQ0381224 |

HM-CC-TC steuert nur HM-CC-VD; ohne Ventil regelt er nichts. Wenn er nicht mehr gebraucht wird: ablernen (Batterien raus, in der CCU löschen).

## 4. Programme (Inhalt prüfen – braucht .sbk oder Screenshots)
- [ ] `SYS Bestätigen` – deaktiviert, nie gelaufen → löschen?
- [ ] `SYS war gestört` – deaktiviert, nie gelaufen → löschen?
- [ ] `urlaub ein` / `urlaub aus` – laufen beide beim CCU-Start → Auslöser „bei Aktualisierung“ → „bei Änderung“?

## 5. Systemvariablen (erst nach Prüfung der Programme löschen)
- [ ] `Systemvariable` (0.0) – vermutlich Test
- [ ] `Systemvariable 1` (Wert `???`) – kaputte Werteliste
- [ ] `myHomeLastPushMessage 1` – Duplikat, nie beschrieben
- [ ] `Heizungsautomati` – genutzt? Name abgeschnitten
- [ ] `schloss` – wofür?
- [ ] `Alarmzone 1`, `Anwesenheit` – CCU-Standard, nur behalten wenn genutzt
- [ ] `@HomeNotification`, `myHomeToken`, `myHomeLastPushMessage` – nur bei Nutzung der myHome/HomeControl-App behalten
