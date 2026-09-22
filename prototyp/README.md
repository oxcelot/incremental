# Stamp Duty — Prototyp P1

Zweck dieses Prototyps ist eine einzige Frage:
**Trägt die Kern-Handlung 60 Sekunden lang, ohne dass irgendetwas belohnt wird?**

Deshalb gibt es hier bewusst **keine** Upgrades, **keinen** Skill Tree, **keine**
Grafik, **keine** Meta-Progression. Wenn der Loop ohne all das keinen Spaß macht,
rettet ihn später auch kein Skill Tree.

## Starten

`prototyp/index.html` im Browser öffnen. Kein Build, keine Abhängigkeiten.
Unter Windows reicht ein Doppelklick auf die Datei.

## Was drin ist

| System | Umsetzung |
|---|---|
| Runden-Uhr | 40 Sekunden Echtzeit |
| Aufwärmphase | 2 s — erst schauen, dann schlägt der Stempel zu |
| Stempeltakt | 1,5 s, **automatisch** — der Spieler steuert nur die Position |
| Takte pro Tag | 26 |
| Tisch | 12 Plätze, Formulare rücken alle 1,6 s vom Stapel nach |
| Posteingang | wächst um 1 Vorgang pro Sekunde — schneller als 40 Takte schaffen |
| Stempel | GENEHMIGT, NACHFORDERUNG, WEITERLEITEN, ABGELEHNT |
| Stempelwechsel | auf das fremde Kissen stempeln — kostet einen Takt und die Kombo |
| Nachtinten | auf das eigene Kissen stempeln — kostet einen Takt, **hält** die Kombo |
| Hand gehoben | Maus außerhalb des Tisches — keine Tinte, Takt trotzdem weg |
| Tinte | Kostenposten, 0,50 € pro Druck, Abrechnung bei Dienstschluss |
| Kombo | ×1,1 pro Stufe, Deckel ×3 |
| Formulartypen | 4, jeweils mit eigenem sichtbarem Merkmal |
| Abrechnungsbogen | Ertrag, Tinte, Fehldrucke, Beschwerden, Rückstand, Punkte |
| Homescreen | Fortbildungsplan mit 13 Knoten, Kontostand, Punktestand |

Vier Formulartypen statt der drei aus der Spezifikation, damit jeder der vier
Stempel eine Aufgabe hat.

## Tuning

Sämtliche Stellschrauben stehen im `CONFIG`-Block ganz oben im `<script>`.
Nichts anderes muss angefasst werden. Die interessantesten Werte:

- `beatMs` — der Takt. Das Spiel fühlt sich bei 1200 völlig anders an als bei 1800.
- `inflowMs` — der Zulauf. Bestimmt, wie aussichtslos sich der Rückstand anfühlt.
- `inkCost` — aktuell 12,5 % eines Standardvorgangs.
- `startDelayMs` — die Aufwärmphase. Reicht sie, um den Tisch zu erfassen?
- `missBreaksCombo` — steht auf `false`. Bei Auto-Feuer ist ein Fehldruck
  teilweise unfreiwillig; ihn zusätzlich mit dem Komboverlust zu bestrafen,
  könnte zu hart sein. Beides ausprobieren.

## Worauf beim Testen zu achten ist

1. Will man nach dem Abrechnungsbogen sofort den nächsten Tag?
2. Ertappt man sich beim Planen der Reihenfolge — oder klickt man nur ab?
3. Ist der Stempelwechsel eine echte Abwägung oder immer offensichtlich?
4. Funktioniert Nachtinten als Denkpause, oder fühlt es sich wie Strafe an?
5. Ist der automatische Takt angenehm oder hetzt er?
6. Reichen 26 Takte, oder ist der Tag vorbei, bevor er angefangen hat?
7. Machen die Namen im Fortbildungsplan Lust auf den nächsten Tag?

## Was bewusst fehlt

Das **Freischalten** im Fortbildungsplan. Der Baum zeigt, was kommt — kaufen
kann man noch nichts. Sonst misst der Test nur noch die Belohnung statt der
Handlung. Alle Effekte sind in `TREE` bereits als Text hinterlegt und schreiben
später in `STATS`, das schon von der Spiellogik gelesen wird.
