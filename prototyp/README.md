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
| Kissenkapazität | **1 Vorgang**. Danach wird der Stempel grau, ein rotes `!` erscheint, der Abdruck bringt nur noch 25 % |
| Hand gehoben | Maus außerhalb des Tisches — keine Tinte, Takt trotzdem weg |
| Tinte | Kostenposten, 0,50 € pro Druck, Abrechnung bei Dienstschluss |
| Kombo | ×1,1 pro Stufe, Deckel ×3 |
| Formulartypen | 4, jeweils mit eigenem sichtbarem Merkmal |
| Abrechnungsbogen | Ertrag, Tinte, Fehldrucke, Beschwerden, Rückstand, Punkte |
| Homescreen | Fortbildungsplan: 13 Icon-Knoten, alle freischaltbar |
| Fortbildungspunkte | 1 pro Tag + 1 je 8 bearbeitete Vorgänge |

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

## Der Fortbildungsplan

Alle zwölf Knoten sind freischaltbar und wirken sofort. Jeder Knoten trägt eine
`apply`-Funktion, die in `STATS` schreibt; `resetStats()` baut die Werte bei
jedem Rundenstart aus `CONFIG` plus allen gekauften Knoten neu auf. Neue
Upgrades brauchen daher nur einen Eintrag in `TREE` — keine Änderung an der
Spiellogik.

| Speiche | Ring 1 | Ring 2 |
|---|---|---|
| Dienstzeit | Überstunden (+6 s) | Gleitzeit (+8 s) |
| Takt | Routine (−0,15 s) | Blindstempeln (−0,20 s) |
| Nachschub | Flinker Bote (1,3 s) | Zweiter Bote (1,0 s) |
| Kissen | Volles Kissen (2 Vorgänge) | Stempelkissen XXL (4) |
| Ergonomie | Handgelenkdrehung (Wechsel ½ Takt) | Nachtinten im Vorbeigehen (½ Takt) |
| Beschaffung | Sparsames Kissen (0,35 €) | Dienst nach Vorschrift (Beschwerde 1,00 €) |

## Warum `fadedFactor` bei 0,25 steht und nicht bei 0,5

Bei einem Kissen, das genau einen Vorgang trägt, gilt über drei Takte:

- nie nachtinten: `4,00 + Faktor·4,00 + Faktor·4,00`
- jeden zweiten Takt nachtinten: `4,00 + 0 + 4,00 = 8,00`

Bei `fadedFactor = 0,5` ergibt die erste Zeile ebenfalls genau 8,00 — beide
Strategien sind exakt gleichwertig, die Entscheidung ist wertlos. Erst bei 0,25
(Ergebnis 6,00) lohnt sich Nachtinten, und das Weiterstempeln mit trockenem
Stempel wird zur bewussten Ausnahme, etwa für einen Vorgang, der sonst
verloren geht.

**Effektiver Durchsatz** bei 26 Takten und Kissenkapazität C: `26·C/(C+1)`
— also 13 Vorgänge bei C=1, 17 bei C=2, 21 bei C=4. Genau deshalb ist
„Volles Kissen" mit 2 Punkten der stärkste Kauf im Spiel.
