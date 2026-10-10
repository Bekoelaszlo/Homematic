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
- [ ] Kontrolle mit `skripte/export_kanaele.hms`

Nicht umbenannt: Systemgeräte und virtuelle Tasten (`urlaub:ein`, `urlaub:aus`) – evtl. von der myHome-App oder Programmen per Name genutzt.

## 2. Räume
- [ ] Zimmer 5: Wandthermostat-Kanäle `:2` (Regler) und `:3` sind keinem Raum zugeordnet
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
