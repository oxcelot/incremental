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

Alle 43 Knoten sind freischaltbar und wirken sofort. Jeder Knoten trägt eine
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
| Takt & Dienstzeit | **Routine ×5** (2,00 → 1,50 s) | **Blindstempeln ×5** (→ 1,00 s) | **Überstunden ×5** (→ 30 s) → **Gleitzeit ×5** (→ 40 s) · **Nachspielzeit ×3** → **Nachtschicht ×3** (→ 6 Takte) |
| Amtsautorität | **Verwaltungsgebühr** (+0,50 € je Vorgang, 35 €) | Dienst nach Vorschrift → Säumniszuschlag | **Ablehnungsbescheid ×5** → **Nachforderungsgebühr ×4**, **Weiterleitungspauschale ×4** |

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
| Seltene Vorgänge | Geübter Blick → **Eilvermerk ×4** → **Sammelverfügung ×4** |
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

### Seltene Vorgänge

Zwei Knoten geben eingehenden Vorgängen eine Seltenheit, die **beim Auflegen**
gewürfelt wird — nicht beim Stempeln. Der Spieler soll sie liegen sehen und
einplanen können; das ist der ganze Punkt.

| | Papier | Randvermerk | Wirkung | Wahrscheinlichkeit |
|---|---|---|---|---|
| **Eilvermerk** | gold | `EILT ×3` | dreifacher **Grundwert**, 7 s lang | 2 / 4 / 6 / 8 % |
| **Sammelverfügung** | diamant | `◆ SAMMEL` | erledigt die 8 Nachbarfelder mit | 2 / 3 / 4 / 5 % |

#### Die Frist des Eilvermerks

Ein Eilvermerk gilt nur `CONFIG.goldMs` lang — 7 Sekunden. Oben auf der Karte
läuft ein Balken ab, in der letzten Sekunde wird er rot und der Randvermerk
pulst. Läuft er aus, wird der Vorgang ein **ganz normaler Vorgang derselben
Art**: gleiche Sorte, gleicher Sollstempel, nur zum Grundwert. Er verschwindet
nicht — man verliert die Prämie, nicht die Arbeit.

Bei 2,0 s Takt sind sieben Sekunden **drei bis vier Schläge**. Der Eilvermerk
ist damit keine Belohnung, die man einsammelt, sondern eine Unterbrechung:
lohnt sich der Stempelwechsel für den einen Vorgang, oder läuft die Serie
weiter? Sieben Sekunden lassen Platz für den Wechsel, den Abdruck **und** den
Weg zurück — auch bei voll ausgebautem Tempo-Ast.

Zwei Dinge, die dabei feststehen:

- **Die Frist läuft über dieselbe Uhr wie der Takt** (`dt` aus `tick()`), nicht
  über eine CSS-Animation. Sonst laufen Balken und Ablauf auseinander, sobald
  der Browser bremst oder der Tab in den Hintergrund geht.
- **Während der Aufwärmphase steht sie still.** In diesen 2 Sekunden kann der
  Spieler gar nicht stempeln — eine Frist, die dort abläuft, wäre nicht zu
  halten. Ein Eilvermerk, der zu Dienstbeginn schon liegt, bekommt seine vollen
  7 Sekunden erst ab dem ersten Schlag.

Die Sammelverfügung sticht den Eilvermerk: erst wird auf sie gewürfelt, nur
wenn sie nicht fällt, auf Gold. Ein Vorgang trägt also nie beides.

Drei Entscheidungen, die die Sammelverfügung erst zu einer Entscheidung machen:

1. **Nur der richtige Stempel zündet sie.** Ein Fehlgriff gibt eine Beschwerde
   wie sonst auch und räumt nichts ab — man muss den Typ erkennen.
2. **Die acht Nachbarn werden immer richtig bearbeitet**, wie beim
   Durchschlagpapier. Daraus kann keine Beschwerde entstehen.
3. **Nachbarn zünden nicht weiter.** Eine Sammelverfügung, die von einer
   anderen mitgerissen wird, löst keine eigene Welle aus — sonst räumt ein
   Treffer den halben Tisch ab.

Am Rand sind es entsprechend weniger Felder: 6 an der Kante, 4 in der Ecke.
Die Nachbarschaft rechnet über `deskCols` und bricht **nicht** in die
Nachbarzeile um — auf dem schmalen Layout (3 Spalten × 4 Zeilen) stimmt sie
dadurch genauso.

Der dreifache Wert gilt für den **Grundwert** des Vorgangs. Verwaltungsgebühr,
Stempelzuschlag und Kombo kommen danach obendrauf, nicht mit ×3 multipliziert —
sonst würde Gold jedes späte Ertrags-Upgrade doppelt verstärken. Die Karte
zeigt den verdreifachten Betrag direkt an, damit die Rechnung sichtbar ist.

### Dienstschluss in drei Schritten

Das Rundenende läuft nicht mehr in einem Sprung, sondern in mehreren Phasen:

| Phase | Was passiert | Was man sieht |
|---|---|---|
| **Nachlauf** (`CONFIG.roundTailMs`, 0,1 s) | Unsichtbar. Der Schlag, der genau auf die Schlusssekunde fällt, kommt noch durch. | Nichts — die Uhr steht schon auf 00:00 |
| **Nachspielzeit** | Die Uhr steht, der Stempel fällt noch `extraBeats` mal (bis zu 6). Kein Zulauf, kein Nachrücken. | Banner über dem Tisch, geschlossener Posteingang, goldene Uhr |
| **Abspann** (`CONFIG.outroMs`, 2 s) | Nichts mehr. Kein Takt, keine Frist. | Der Tisch, so wie er liegen geblieben ist |
| **Abrechnungsbogen** | — | Der Bogen |

Die Nachspielzeit ändert die **Regeln** — kein Nachschub mehr —, deshalb trägt
sie drei getrennte Zeichen, nicht eines:

1. **Ein Banner quer über dem Tisch**: Titel, der Satz *„Kein Nachschub · es
   zählt, was noch liegt"* und **ein Punkt je verbleibendem Schlag**. Die
   Punkte erlöschen einzeln — man sieht die Zugabe ablaufen, ohne eine Zahl
   lesen zu müssen.
2. **Der Posteingang macht sichtbar zu**: abgeblendet, mit einem schrägen
   *„Schalter geschlossen"* darüber. Das ist die halbe Botschaft — nichts
   kommt mehr nach.
3. **Die Uhr** wird gold und heißt *Nachspielzeit*.

Das Banner liegt **nicht über** den Vorgängen: der Tisch bekommt dafür eine
Kopfzeile (`padding-top` 10 → 44 px). Während einer Phase, in der es darauf
ankommt, was noch liegt, darf kein Vorgang verdeckt sein. Die Kacheln rutschen
dadurch einmalig ~23 px nach unten; `ensureMeasured()` misst den Tisch neu, die
Treffererkennung bleibt also richtig (geprüft: alle 12 Plätze nach dem Umbruch).

Der **Abspann** existiert, weil der Abrechnungsbogen sonst den Tisch in dem
Moment verdeckt, in dem der letzte Abdruck fällt. Zwei Sekunden reichen, um
den Tisch abzusuchen — und erst dann macht der Bogen eine Zahl daraus. Die
Glocke läutet zu Beginn des Abspanns, nicht beim Bogen.

#### Der Nachlauf

Bei 20 s Runde, 2 s Aufwärmphase und 2 s Takt fallen die Schläge auf
2, 4, … 20 s — der letzte **genau** auf die Schlusssekunde. Der wurde
verschluckt: `over` wurde wahr, sobald `S.t >= roundMs`, und der Schlag
im selben Frame fiel weg. Die Dienstanweisung versprach 10 Takte, das Spiel
lieferte 9, und bei aktiver Nachspielzeit fraß der verlorene reguläre Schlag
zusätzlich einen aus der Zugabe.

`CONFIG.roundTailMs` (0,1 s) trennt deshalb **Anzeige** und **Ablauf**:

```js
const left = Math.max(0, STATS.roundMs - S.t);        // treibt Uhr und Balken
const over = S.t >= STATS.roundMs + CONFIG.roundTailMs; // treibt das Spiel
```

Die Uhr steht also 0,1 s lang auf `00:00`, während der letzte Schlag noch
fällt. Danach reicht der Nachlauf für keinen zweiten Schlag, weil `accBeat`
gerade zurückgesetzt wurde. Gemessen, alle Ausbaustufen:

| Ausbau | versprochen | gefallen |
|---|---|---|
| Dienstbeginn | 10 | 10 |
| Takt halb ausgebaut | 11 | 11 |
| Takt voll ausgebaut | 19 | 19 |
| Runde + Takt voll | 39 | 39 |
| dazu Nachtschicht ×3 | 39 + 6 | 45 |

### Die ersten fünf Runden

Der Einstieg war zu leer: nach „Volles Kissen" (25 €) kam lange nichts, was
man haben *wollte* — „Sparsames Kissen" spart Centbeträge, alles andere lag
bei 60–120 € oder tief im Baum. Deshalb wurde der Ring 1 umgebaut:

| | Preis | warum früh |
|---|---|---|
| **Routine** (1. Stufe) | **12 €** | Takt 2,00 → 1,90 s, spürbar mehr Durchsatz; die Leiter steigt danach mit Faktor **2,3** statt 1,6 |
| **Volles Kissen** | 25 € | verdoppelt den Durchsatz |
| **Verwaltungsgebühr** | 35 € | +0,50 € je Vorgang, wirkt sofort auf alles — war vorher 160 € und hinter „Dienst nach Vorschrift" |
| Sammelbearbeitung | 45 € | schaltet die Kombo überhaupt erst frei (war 70 €) |
| Handgelenkdrehung | 80 € | |
| Breiter Stempel | 120 € | |

Das `growth`-Feld im `TREE`-Eintrag (Standard 1,6) macht das möglich: **billig
hineinkommen, teuer ausbauen**, ohne die Gesamtstärke des Astes zu verändern.

**Gemessener Verlauf** (derselbe Bot, greedy einkaufend, billigstes zuerst):

| Tag | Ertrag | gekauft |
|---|---|---|
| 1 | 19,00 € | Routine 1 (12 €) |
| 2 | 18,50 € | Volles Kissen (25 €) |
| 3 | 19,50 € | — (spart) |
| 4 | 21,50 € | Routine 2 (28 €) |
| 5 | 22,00 € | Verwaltungsgebühr (35 €) |

Nach fünf Runden: **3 verschiedene Upgrades, 4 Stufen**, und an vier von fünf
Tagen gibt es etwas zu kaufen. Der Ertrag wächst dabei von 19 auf 30,50 € am
sechsten Tag.

### Die Detailkarte als Sprechblase

Die Einzelheiten zum anvisierten Knoten standen früher unten im Bild. Der Blick
musste damit zwischen Kästchen und Bildrand hin und her — bei 43 Knoten der
halbe Bildschirm. Jetzt erscheint die Karte als **Sprechblase direkt über dem
Kästchen**, das der Zeiger trifft, und verschwindet, sobald er es verlässt.

Gemessen wird am **gezeichneten** Knoten (`g.getBoundingClientRect()`), nicht
an seinen `TREE`-Koordinaten — dazwischen liegen `viewBox`, Zoom und
Verschiebung. Drei Regeln halten die Blase im Bild:

- Passt sie über dem Kästchen nicht mehr hin, **klappt sie darunter**
  (`.oben` / `.unten` steuern, auf welcher Seite der Zipfel sitzt).
- Seitlich wird sie in den Bildschirm hineingeschoben; der **Zipfel bleibt am
  Kästchen** (`--px`), auch wenn die Blase verschoben ist.
- Beim Zoomen und beim Fenstergrößenwechsel wird sie nachgeführt, beim
  Schieben des Baums ausgeblendet.

Sie hat `pointer-events:none`, blockiert also nie einen Klick. Die Fußzeile
trägt jetzt nur noch den Hinweis „Zeiger auf ein Kästchen" und den Startknopf —
der Baum hat dadurch spürbar mehr Platz.

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

Es gelten die **echten Preise** aus `TREE`. Oben links im Fortbildungsplan
sitzt ein **Schalter**: eingeschaltet kostet jede Fortbildung **0 €**, damit
sich der ganze Baum in einer Runde durchprobieren lässt, ausgeschaltet gelten
wieder die Preise.

Der Stand liegt in `meta.demo` und wird mitgespeichert; `CONFIG.demoStart`
bestimmt nur, wie der Schalter beim allerersten Start steht (`false`). Beim
Umschalten wird der Baum neu gezeichnet und die Detailkarte aktualisiert —
sonst nennt sie den Preis von vorhin.

`costOf()` liefert im Demo-Modus 0 und rechnet sonst
`cost × 1,6^gekaufte Stufen`. Die Preise in `TREE` werden dabei **nie**
angefasst, der Schalter ist also gefahrlos hin und her zu bewegen.

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
ist „Volles Kissen" der stärkste erste Kauf im Spiel. Es kostet **25 €** und
ist damit der einzige Knoten, der früh überhaupt erreichbar ist — alle anderen
Ring-1-Knoten liegen bei 60–120 €. Der Einstieg führt also über das Kissen,
ohne dass ihn eine Regel dorthin zwingt.

**Gemessen** mit einem Bot, der spielt wie ein vernünftiger Mensch — passenden
Vorgang stempeln, Stempel wechseln wenn eine andere Sorte häufiger liegt, nur
sonst nachtinten: ein erster Tag bringt **rund 19 €**. Volles Kissen ist damit
nach dem zweiten Tag drin.

(Ein früherer Bot, der statt zu wechseln nachtintete, kam nur auf 13,90 € —
die Zahl steht noch in älteren Commit-Texten und ist zu niedrig.)

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
