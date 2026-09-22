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
| Kissenkapazität | **1 Vorgang**. Danach wird der Stempel grau, ein rotes `!` erscheint — er stempelt **gar nicht** mehr |
| Hand gehoben | Maus außerhalb des Tisches — keine Tinte, Takt trotzdem weg |
| Tinte | Kostenposten, 0,50 € pro Druck, Abrechnung bei Dienstschluss |
| Kombo | ×1,1 pro Stufe, Deckel ×3 |
| Formulartypen | 4, jeweils mit eigenem sichtbarem Merkmal |
| Abrechnungsbogen | Ertrag, Tinte, Fehldrucke, Beschwerden, Rückstand, Punkte |
| Homescreen | Fortbildungsplan: 13 Icon-Knoten, zoom- und verschiebbar |
| Währung | **Euro** — Upgrades werden vom eigenen Kontostand bezahlt |

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
8. Fühlt sich ein Leerschlag nach eigenem Fehler an oder nach Gemeinheit?

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

## Das leere Kissen

Ist die Tinte alle, hinterlässt der Stempel **nichts**. Der Takt verpufft, es
wird keine Tinte berechnet, der Vorgang bleibt liegen. Wer das Nachtinten nicht
einplant, setzt schlicht aus. Die Abrechnung zählt diese Leerschläge und rechnet
vor, was sie gekostet haben.

**Effektiver Durchsatz** bei 26 Takten und Kissenkapazität C: `26·C/(C+1)`
— also 13 Vorgänge bei C=1, 17 bei C=2, 21 bei C=4. Deshalb ist „Volles
Kissen" für 40 € der stärkste erste Kauf im Spiel.

**Kosten pro Vorgang** bei C=1: ein Stempeldruck plus ein Nachtinten, also
1,00 € Tinte auf einen Vorgang im Wert von rund 4,00 €. Bei C=2 sinkt das auf
0,75 €, bei C=4 auf 0,63 €.

## Geld ist die Währung

Es gibt keine zweite Währung. Was am Tagesende auf dem Konto landet, ist
zugleich Punktestand und Kaufkraft — man bezahlt seine Fortbildungen vom
eigenen Gehalt. Der ganze Baum kostet **1000 €**, ein Tag bringt anfangs rund
30 €. Der erste Kauf fällt damit auf Tag 2.

Das ist der wichtigste Balancing-Hebel im Prototyp: Fühlt sich der Baum zäh an,
gehören die `cost`-Werte in `TREE` heruntergesetzt, nicht die Erträge hoch.
