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
| Runden-Uhr | 20 Sekunden Echtzeit zu Dienstbeginn, per Fortbildung bis 40 s |
| Aufwärmphase | 2 s — erst schauen, dann schlägt der Stempel zu |
| Stempeltakt | 2,0 s zu Dienstbeginn, per Fortbildung bis 1,0 s — **automatisch**, der Spieler steuert nur die Position |
| Takte pro Tag | 10 zu Dienstbeginn, 39 bei vollem Ausbau von Takt und Dienstzeit |
| Tisch | 12 Plätze, Formulare rücken alle 1,6 s vom Stapel nach |
| Posteingang | wächst um 1 Vorgang pro Sekunde — schneller, als die Takte des Tages schaffen |
| Stempel | GENEHMIGT, NACHFORDERUNG, WEITERLEITEN, ABGELEHNT |
| Stempelwechsel | auf das fremde Kissen stempeln — kostet einen Takt und die Kombo |
| Nachtinten | auf das eigene Kissen stempeln — kostet einen Takt, **hält** die Kombo |
| Kissenkapazität | **1 Vorgang**. Ein Balken unter dem Stempel zeigt den Füllstand und leert sich mit jedem Abdruck |
| Leerer Stempel | komplett grau, Aufschrift „TINTE LEER", rotes Warnabzeichen — er stempelt **gar nicht** mehr |
| Hand gehoben | Maus außerhalb des Tisches — keine Tinte, Takt trotzdem weg |
| Tinte | Kostenposten, 0,50 € pro Druck, Abrechnung bei Dienstschluss |
| Kombo | ×1,1 pro Stufe, Deckel ×3 |
| Formulartypen | 4, jeweils mit eigenem sichtbarem Merkmal |
| Abrechnungsbogen | Ertrag, Tinte, Fehldrucke, Beschwerden, Rückstand, Punkte |
| Homescreen | Fortbildungsplan: 34 Icon-Knoten, zoom- und verschiebbar |
| Voraussetzungen | Ring 2 ist gesperrt, bis der Knoten davor freigeschaltet ist |
| Währung | **Euro** — Upgrades werden vom eigenen Kontostand bezahlt |

Vier Formulartypen statt der drei aus der Spezifikation, damit jeder der vier
Stempel eine Aufgabe hat.

## Tuning

Sämtliche Stellschrauben stehen im `CONFIG`-Block ganz oben im `<script>`.
Nichts anderes muss angefasst werden. Die interessantesten Werte:

- `beatMs` — der Takt. Das Spiel fühlt sich bei 1200 völlig anders an als bei 1800.
  Startwert 2000; `routi` und `blind` ziehen ihn in 0,10-s-Stufen bis 1000 herunter.
- `roundSeconds` — der Arbeitstag. Startwert 20; `ueber` und `gleit` hängen in
  2-Sekunden-Stufen bis zu 20 s an.
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
6. Reichen 10 Takte am ersten Tag, oder ist er vorbei, bevor er angefangen hat?
7. Machen die Namen im Fortbildungsplan Lust auf den nächsten Tag?
8. Fühlt sich ein Leerschlag nach eigenem Fehler an oder nach Gemeinheit?

## Der Fortbildungsplan

Alle 40 Knoten sind freischaltbar und wirken sofort. Jeder Knoten trägt eine
`apply`-Funktion, die in `STATS` schreibt; `resetStats()` baut die Werte bei
jedem Rundenstart aus `CONFIG` plus allen gekauften Knoten neu auf. Neue
Upgrades brauchen daher nur einen Eintrag in `TREE` — keine Änderung an der
Spiellogik.

Sechs Speichen, an fünf davon gabelt sich der Weg — man muss wählen.

| Speiche | Ring 1 | Gabelung A | Gabelung B |
|---|---|---|---|
| Kissen & Tinte | Volles Kissen (2 Vorgänge) | XXL (4) → Fasspumpe (6) | Sparsames Kissen (0,35 €) → Großbestellung (0,22 €) |
| *(Volltreffer und Kombo bilden einen eigenen Zweig, siehe unten)* | | | |
| Fläche & Nachschub | Breiter Stempel (2 Vorgänge) | Amtsstempel XXL (4) → **Sammelakte ×4** | **Flinker Bote ×4** → **Zweiter Bote ×4** → **Ablagekorb ×3** |
| Ergonomie | Handgelenkdrehung (Wechsel ½ Takt) | Trockenwechsel (Wechsel ohne Tinte) | Nachtinten im Vorbeigehen (½ Takt) |
| Takt & Dienstzeit | **Routine ×5** (2,00 → 1,50 s) | **Blindstempeln ×5** (→ 1,00 s) | **Überstunden ×5** (→ 30 s) → **Gleitzeit ×5** (→ 40 s) → **Nachspielzeit ×3** |
| Amtsautorität | Dienst nach Vorschrift (Beschwerde 1,00 €) | Verwaltungsgebühr → Säumniszuschlag | **Ablehnungsbescheid ×5** → **Nachforderungsgebühr ×4**, **Weiterleitungspauschale ×4** |

### Der Serien-Zweig

Kombo und Volltreffer hängen in **einem** zusammenhängenden Zweig rechts vom
Zentrum. Seine Wurzel schaltet die Kombo überhaupt erst frei:

**Sammelbearbeitung** (70 €) — *„Gleichartige Vorgänge nacheinander bearbeitet
bauen ab jetzt eine Kombo auf. Ohne diese Fortbildung zerfällt jede Serie
sofort."*

Der Name benennt die Bedingung: Die Kombo wächst, solange **gleichartige**
Vorgänge aufeinander folgen — ein Stempelwechsel setzt sie zurück, bis „Der
kurze Dienstweg" das aufhebt.

Ohne diesen Knoten bleibt `STATS.comboOn` auf 0: `bumpCombo()` zählt gar nicht
hoch und `multFor()` gibt konstant ×1,0 zurück. Die Kopfzeile zeigt dem Spieler
also von Beginn an eine Kombo-Anzeige, die sich nie bewegt — das ist der Köder.

Von der Wurzel gehen vier Äste ab:

| Ast | Knoten |
|---|---|
| Volltreffer | Geübter Blick → **Routiniertes Auge ×5** → Glückliche Hand · **Sechster Sinn ×5** → **Durchschlagpapier ×5** |
| Kombo-Deckel | **Aktenzeichen-Gedächtnis ×4** → Beharrlichkeit ⟶ |
| Kombo-Zuwachs | **Schwung ×5** → **Warmgelaufen ×5** → Ordnungsliebe, und ⟶ |
| Auszahlung | **Leistungsprämie ×5** — hängt direkt an der Wurzel |
| Abschluss | **Der kurze Dienstweg** — hängt an **beiden** Kombo-Ästen |
| Kreuzung | **Doppelter Durchschlag** — hängt an Sechster Sinn **und** Durchschlagpapier |

**Zwei Knoten mit doppeltem Vorgänger.** *Der kurze Dienstweg* schließt die zwei Kombo-Äste zusammen, *Doppelter Durchschlag* (900 €) die beiden Volltreffer-Äste: ein Volltreffer löst dort immer einen Durchschlag aus. Beide funktionieren gleich: Er wird erst
kaufbar, wenn Beharrlichkeit *und* Warmgelaufen **vollständig** ausgebaut sind.
Dafür kann ein Knoten mehrere Vorgänger haben (`PARENTS` statt eines einzelnen
Elternteils); `missingParents()` liefert die noch fehlenden, und die Detailkarte
nennt sie beim Namen.

Dieser Zweig ist zu groß für das Radialschema und wird deshalb **von Hand
gesetzt** — über `hx`/`hy` im **selben Raster** wie die radialen Knoten, nicht
in fertigen Bildpunkten. Erst zum Schluss zieht `SPREAD_X`/`SPREAD_Y` alles
elliptisch in die Breite. Dadurch wirken Abstände in beiden Systemen gleich,
und eine Änderung am Spreizfaktor zieht den ganzen Baum mit.

### Die `hidden`-Falle

`.veil`, `#home` und `#stamp` setzen selbst ein `display`, und **jede
Autoren-Regel schlägt das `display:none`, das der Browser an `[hidden]`
hängt**. Ohne eigene Regel bleiben die Overlays also sichtbar, egal was der
Code setzt. Im Artifact-Rahmen fällt das nicht auf, weil der eine eigene
`!important`-Regel mitliefert — die Datei direkt von der Platte geöffnet war
dadurch kaputt, ohne dass es je jemandem aufgefallen wäre. Das Spiel-CSS
bringt die Regel jetzt selbst mit:

```css
[hidden]{display:none !important;}
```

Gefunden nicht im Spiel, sondern beim Screenshot: der Fortbildungsplan lag
laut DOM offen, auf dem Bild war aber die Dienstanweisung zu sehen.

### Layout-Prüfungen

Nach jeder Verschiebung laufen drei Tests über den gesamten Baum:

| Prüfung | Grenzwert | Stand |
|---|---|---|
| Kästchen überlappen sich | 80 px waagerecht **oder** 106 px senkrecht | engster Abstand 47 px Rand zu Rand |
| Verbindungen kreuzen sich | keine | keine |
| Verbindung läuft durch fremdes Kästchen | keine, Ziel ≥ 30 px Abstand | engste 32 px |

Alle drei sind nötig: Beim letzten Umbau streifte die Linie *Dienst nach
Vorschrift → Weiterleitungspauschale* das Kästchen *Blindstempeln* auf **2 px**,
ohne dass es im Überblick auffiel.

### Ertrag nach Stempelart

`STATS.stampBonus` hält je Stempel einen Aufschlag, der zum Grundwert des
Vorgangs addiert wird, bevor die Kombo multipliziert. Das belohnt
Spezialisierung: Wer auf Ablehnungen setzt, spielt einen anderen Tisch als wer
weiterleitet.

| Knoten | Stufen | je Stufe | gemaxt |
|---|---|---|---|
| Ablehnungsbescheid | 5 | +0,50 € | +2,50 € auf ABGELEHNT |
| Weiterleitungspauschale | 4 | +0,60 € | +2,40 € auf WEITERLEITEN |
| Nachforderungsgebühr | 4 | +0,50 € | +2,00 € auf NACHFORDERUNG |

Die Weiterleitung ist mit 3,00 € die billigste Aktion und kostet keinen vollen
Takt — aufgewertet wird sie zur echten Alternative statt zur Notlösung.

### Der Kombo-Zweig

Die Kombo lief vorher, wurde aber von keinem einzigen Upgrade berührt. Sechs
neue Knoten hängen an der Ergonomie-Speiche, weil Stempelwechsel und Kombo
dieselbe Frage betreffen.

| Knoten | Stufen | Wirkung |
|---|---|---|
| Aktenzeichen-Gedächtnis | 4 | Kombo-Deckel ×3 → ×5 |
| Schwung | 5 | Kombo wächst um 0,15 statt 0,10 je Vorgang |
| Warmgelaufen | 5 | Kombo startet morgens auf Stufe 1–5 |
| Beharrlichkeit | 1 | Eine Beschwerde halbiert die Kombo, statt sie zu brechen |
| Der kurze Dienstweg | 1 | Ein Stempelwechsel bricht die Kombo nicht mehr |
| **Getrennte Registratur** | 1 | Jede Aktenart führt ihre **eigene** Kombo, die bis Dienstschluss hält |

**Getrennte Registratur** ist der Endgame-Knoten: Statt einer Kette laufen vier
parallel, eine je Aktenart. Der Stempelwechsel verliert damit jeden Nachteil,
und die Anzeige im Kopf zeigt die Kombo der Art, deren Stempel gerade in der
Hand liegt. Mit 900 € ist er der teuerste Knoten im Baum — er entwertet sonst
zu früh die gesamte Ergonomie-Speiche, auf der er sitzt.

### Mehrstufige Upgrades

Fünf Knoten lassen sich mehrfach ausbauen. `meta.owned[id]` hält die Stufenzahl
statt eines Ja/Nein; Knoten ohne `ranks` haben genau eine Stufe und verhalten
sich wie vorher. Jede weitere Stufe kostet das **1,6-fache** der vorigen.

| Knoten | Stufen | Wert | Preise |
|---|---|---|---|
| Sechster Sinn | 5 | 20 % → 45 % *(+5 pp)* | 130 / 208 / 333 / 532 / 852 € |
| Routiniertes Auge | 5 | ×2 → ×4,5 *(+0,5)* | 140 / 224 / 358 / 573 / 917 € |
| Durchschlagpapier | 5 | 5 % → 15 % *(+2,5 pp)* | 260 / 416 / 666 / 1065 / 1704 € |
| Flinker Bote | 4 | 1,60 s → 1,20 s *(−0,10 s)* | 40 / 64 / 102 / 164 € |
| Zweiter Bote | 4 | 1,20 s → 0,80 s *(−0,10 s)* | 110 / 176 / 282 / 451 € |

Die Kästchen zeigen die Stufen als Punktleiste am unteren Rand, die Detailkarte
den aktuellen und den nächsten Wert.

**`needsMaxParent`** steht beim Zweiten Boten: Er wird erst kaufbar, wenn der
Flinke Bote **voll** ausgebaut ist. Sonst ließe sich die Kette überspringen und
die Nachrückrate auf Umwegen unter 0,80 s drücken. Alle anderen Knoten öffnen
sich schon bei Stufe 1 des Vorgängers.

`resetStats()` klemmt zur Sicherheit ab: Takt und Nachrücken nie unter 0,80 s,
Tinte nie unter 0,05 €. Das fängt Kombinationen ab, die beim Balancing entstehen.

### Der Flächenstempel muss gezielt werden

Getroffen wird, was der **sichtbare Stempel tatsächlich überdeckt**. Sein
Quadrat (74 px) wird gegen die Vorgänge geschnitten; wer am meisten darunter
liegt, kommt zuerst dran. Mehr als `hitCount` Vorgänge gehen nie mit.

| Stand des Stempels | mit „Breiter Stempel" | mit „Amtsstempel XXL" |
|---|---|---|
| mitten auf einer Karte | 1 Vorgang | 1 Vorgang |
| über der Lücke zwischen zwei Karten | **2 Vorgänge** | 2 Vorgänge |
| über dem Kreuzungspunkt von vier Karten | 2 (gedeckelt) | **4 Vorgänge** |

`CONFIG.areaMinOverlap` (0,10) legt fest, wie viel des Stempels über einer
Karte liegen muss, damit sie zählt — ein Pixel Berührung reicht nicht.
**Leere Plätze belegen keinen Treffer:** Liegt neben dem Kreuzungspunkt nur
eine leere Ablage, werden trotzdem die drei vorhandenen Vorgänge bearbeitet.

Der Stempel behält immer seine Größe, die Reichweite zeigen die **amber
umrandeten Plätze**. Da Trefferprüfung und sichtbarer Stempel jetzt dieselbe
Fläche benutzen, stimmt die Markierung genau mit dem überein, was man sieht.

### Platzmaße müssen nachgemessen werden

Der Tisch ändert seine Größe noch, **nachdem** er gebaut wurde: Der
Stempelhalter darunter entsteht später und staucht ihn, Schriften laden nach,
das Fenster wird verändert. Wer die Platzmaße einmal beim Aufbau nimmt, rechnet
den Rest der Runde mit einem Tisch, den es nicht mehr gibt.

Konkret gemessen: 177 px gespeicherte Platzhöhe gegen 143 px echte — die Zeilen
lagen um bis zu **70 px** daneben, und eine veraltete Mittelzeile reichte in die
echte Unterzeile hinein. Deshalb prüft `ensureMeasured()` vor jedem Bild, ob der
Tisch noch dort liegt, wo gemessen wurde, und `startRound()` misst zusätzlich
am Ende des Aufbaus, wenn der Stempelhalter schon steht.

Ein breiter Abdruck trifft **alle** Vorgänge unter sich, auch die, die einen
anderen Stempel verlangen — die zählen als Beschwerde. Damit wird die Fläche
zur Abwägung statt zum Freifahrtschein: Man sucht Nester gleichartiger
Vorgänge, statt blind draufzuhalten.

### Demo-Modus

`CONFIG.demoCheapUpgrades` steht auf `true`: **jede Fortbildung kostet 1 €**,
damit sich der ganze Baum in wenigen Runden durchprobieren lässt. Auf dem
Homescreen weist ein rotes Schild darauf hin. Die echten Preise stehen
unverändert in `TREE` — Flag auf `false`, und sie gelten wieder.

### Volltreffer und Durchschlag

Ohne Fortbildung gibt es **keine** Volltreffer (`critChance` startet bei 0).
„Geübter Blick" schaltet sie überhaupt erst frei — danach trennen sich die Wege
zwischen höherem Multiplikator und höherer Wahrscheinlichkeit.

„Durchschlagpapier" erledigt mit 5 % Wahrscheinlichkeit einen zweiten,
zufälligen Vorgang auf dem Tisch — und zwar **immer mit der richtigen Aktion**,
egal was gerade in der Hand liegt. Ein Durchschlag kann also nie eine
Beschwerde auslösen und bringt stets den vollen Ertrag.

Damit die 5 % nicht untergehen, bekommt der betroffene Vorgang eine eigene
Animation: Die Karte springt kurz auf und kippt weg, eine blaue Ringwelle läuft
nach außen, und der Aufdruck erscheint als **blauer Doppeldruck** — der
korrekte Stempelname zweimal, leicht versetzt, wie ein verrutschter
Durchschlag — mit „DURCHSCHLAG" darunter. Dazu ein eigener Ton und die blaue
Zahl am Zeiger. Kein anderer Vorgang im Spiel sieht so aus.

## Das leere Kissen

Ist die Tinte alle, hinterlässt der Stempel **nichts**. Der Takt verpufft, es
wird keine Tinte berechnet, der Vorgang bleibt liegen. Wer das Nachtinten nicht
einplant, setzt schlicht aus. Die Abrechnung zählt diese Leerschläge und rechnet
vor, was sie gekostet haben.

**Effektiver Durchsatz** bei T Takten und Kissenkapazität C: `T·C/(C+1)`
— am ersten Tag (T=10) also 5 Vorgänge bei C=1, 7 bei C=2, 8 bei C=4. Deshalb
ist „Volles Kissen" für 40 € der stärkste erste Kauf im Spiel.

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

**Voraussetzungen.** Ein Knoten wird erst kaufbar, wenn der Knoten davor
freigeschaltet ist. Die Ring-2-Knoten sind also gesperrt, bis ihr Ring-1-Knoten
gekauft wurde. Ein Tipp auf einen gesperrten Knoten kauft nichts, zeigt aber in
der Detailkarte, welcher Knoten ihm im Weg steht.

## Warum der Kauf nicht am `click`-Event hängt

Die Zoom-Steuerung ruft beim Ziehen `setPointerCapture()` auf dem SVG auf.
Sobald ein Element den Zeiger eingefangen hat, wird das daraus abgeleitete
`click`-Event auf dieses Element umgeleitet — es landet also auf dem `<svg>`
statt auf dem angeklickten `<g>`. Ein Klick-Listener am Knoten feuert dann nie.

Deshalb erledigt `endDrag()` den Kauf selbst: Beim `pointerup` wird geprüft, ob
sich der Zeiger seit dem `pointerdown` um mehr als 5 px bewegt hat. Wenn nicht,
war es ein Tippen — dann sucht `document.elementFromPoint()` den Knoten unter
dem Zeiger und kauft ihn. Der Zeiger wird außerdem erst eingefangen, **nachdem**
die 5-px-Schwelle überschritten ist, nicht schon beim Drücken.

## Wo die Upgrades dokumentiert sind

`verbesserungen.txt` im Projektstamm listet **alle 34 eingebauten Upgrades** mit
Stufen, Stufenwerten, Preisstaffel, Stellschraube und Vorgängern — dazu die
sechs vorgeschlagenen, aber nicht gebauten, eine Übersicht aller 25
Stellschrauben in `STATS`, eine Anleitung zum Einbauen und Platz für eigene
Ideen.

Die Liste wird **aus dem laufenden Spiel ausgelesen**, nicht von Hand gepflegt.
Nach größeren Änderungen am Baum gehört sie neu erzeugt, sonst driftet sie.
