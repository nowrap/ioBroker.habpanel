# Das Raster verstehen: Spalten, Zeilenhöhe, Anfasser

Drei Eigenheiten von HABPanel, die immer wieder für „das geht nicht" sorgen. Keine davon
ist ein Fehler, alle drei sind an einem Nachmittag nachgemessen worden.

## 1. Die Spaltenzahl entscheidet über Höhe *und* Breite

**Dashboard-Einstellungen → Erweitert → „Nb. of columns (default 12)"**

Ist das Feld leer, gilt **12**. Und weil `Row height (%)` leer bedeutet „quadratische
Zellen", ist eine Zeile dann **genauso hoch wie eine Spalte breit**.

Auf einem 2530 px breiten Bildschirm heißt das:

| Spalten | Schrittweite | kleinste Kachel |
|---|---|---|
| 12 (Standard) | 210 px | 210 × 210 px |
| 36 | 70 px | 70 × 70 px |
| 60 | 42 px | 42 × 42 px |

Daraus folgen die beiden häufigsten Beschwerden, und beide haben dieselbe Ursache:

- *„Ich kann die Kachel nicht flacher ziehen."* Die kleinste Höhe ist **eine Zeile**.
  Bei 12 Spalten sind das 210 px.
- *„Drittelbreiten gehen nicht."* Das Raster kennt nur Zwölftel. Für Drittel, Fünftel
  oder feinere Aufteilungen braucht es mehr Spalten.

**Abhilfe:** Spaltenzahl hochsetzen. 60 ist für Übersichtsseiten mit vielen Werten
bewährt, 36 für Seiten mit größeren Elementen. `Row height (%)` dabei leer lassen — die
quadratischen Zellen sind genau das, was man will, sobald die Spalten fein genug sind.

Der Warnkasten „Diese Optionen sind derzeit experimentell, instabil" steht dort seit
2017. Die Spaltenzahl ist davon der harmloseste Teil.

## 2. Der Größen-Anfasser ist nur beim Überfahren sichtbar

Das Dreieck unten rechts existiert immer, ist aber im Ruhezustand **12 × 12 px groß und
vollständig durchsichtig**:

```css
.handle-se                                  { border-color: transparent }
.gridster .gridster-item:hover .handle-se   { border-width: 0 0 3rem 3rem !important;
                                              border-color: … var(--widget-text-color) }
```

Erst der Mauszeiger auf der Kachel macht daraus ein großes, helles Dreieck. Wer die
Kachel nur ansieht, hält sie für nicht veränderbar — ein Bildschirmfoto ohne Mauszeiger
zeigt nie einen Anfasser.

## 3. `.dash-card` ist toter Code

Im Stylesheet steht:

```css
.dash-card { min-height: 100px !important; }
.dash-card .handle-se { … }
```

**Diese Klasse kommt im DOM nicht vor.** Kacheln sind `.box` innerhalb eines
`li.gridster-item`, und dort gilt `min-height: 0`. Wer die Mindesthöhe für die Ursache
zu großer Kacheln hält und sie überschreibt, ändert nichts — die Regel greift auf kein
Element.

Die tatsächliche Kachelhöhe setzt Gridster als Inline-Stil aus `sizeY × Zeilenhöhe`.

## Beispiel: eine Zeile mit Chart und Wertkacheln

Zum Ausprobieren, mit erfundenen Datenpunkten. **60 Spalten**, `Row height` leer.
Rechnung: 60 ÷ 3 = 20 Spalten je Block, davon 15 fürs Chart und 5 für die Kachelspalte
daneben. Das Chart ist 6 Zeilen hoch, die Kachel darunter genau 1 — das geht nur, weil
eine Zeile bei 60 Spalten rund 42 px misst.

In den Dashboard-Einstellungen unter *Benutzerdefinierte Widgets* lässt sich ein
Dashboard als JSON einfügen; alternativ die Werte von Hand in die Widgets eintragen.

```json
{
  "name": "Rasterbeispiel",
  "columns": 60,
  "widgets": [
    { "type": "chart",  "col": 0,  "row": 0, "sizeX": 15, "sizeY": 6,
      "name": "Temperatur",
      "series": [
        { "item": "0_userdata.0.demo.temp_wohnzimmer", "name": "Wohnzimmer", "color": "#ef5350" },
        { "item": "0_userdata.0.demo.temp_kueche",     "name": "Küche",      "color": "#66bb6a" }
      ] },
    { "type": "dummy",  "col": 15, "row": 0, "sizeX": 5, "sizeY": 6,
      "name": "Wohnzimmer", "item": "0_userdata.0.demo.temp_wohnzimmer", "unit": "°C", "format": "%.1f" },

    { "type": "dummy",  "col": 0,  "row": 6, "sizeX": 7, "sizeY": 1,
      "name": "Wohnzimmer", "item": "0_userdata.0.demo.temp_wohnzimmer", "unit": "°C", "format": "%.1f" },
    { "type": "dummy",  "col": 7,  "row": 6, "sizeX": 8, "sizeY": 1,
      "name": "Küche",      "item": "0_userdata.0.demo.temp_kueche",     "unit": "°C", "format": "%.1f" },
    { "type": "dummy",  "col": 15, "row": 6, "sizeX": 5, "sizeY": 1,
      "name": "Außen",      "item": "0_userdata.0.demo.temp_aussen",     "unit": "°C", "format": "%.1f" }
  ]
}
```

So sieht das fertig aus — Ansichtsmodus, **60 Spalten**, alle Werte erfunden:

![Drei Bloecke in Chartgroesse, darunter eine Kachelzeile von zwei Zeilen Hoehe mit farbigen Balken](bilder/beispiel-ansicht.jpg)

Die obere Reihe ist 6 Zeilen hoch, die Kachelzeile darunter **2** — zusammen 181 und
57 Pixel. Bei 12 Spalten waere allein die Kachelzeile rund 420 px hoch gewesen, und die
Aufteilung in sechs gleich breite Kacheln haette gar nicht ins Raster gepasst.

Derselbe Stand im Bearbeitungsmodus:

![Dasselbe Dashboard im Bearbeitungsmodus, Kacheln mit Kopfzeile und Menuepunkt](bilder/beispiel-bearbeiten.jpg)

Zwei Unterschiede fallen auf, und beide sind normal:

- **Die Werte fehlen.** Im Bearbeitungsmodus zeigt HABPanel nur die Beschriftung.
- **Jede Kachel bekommt eine Kopfzeile** mit dem Verschiebegriff links und dem
  Menuepunkt rechts. Die Kopfzeile liegt *ueber* dem Inhalt, nicht daneben — sie
  verbraucht keine Rasterzeile.

Worauf es dabei ankommt:Worauf es dabei ankommt:

- **Die Kachelzeile füllt die Chartbreite randlos.** 7 + 8 = 15, genau die Breite des
  Charts darüber. Bleibt ein Rest, sieht die Seite gerüttelt aus.
- **Jede Kurve hat ihre Kachel**, mit demselben Wortlaut und derselben Farbe. Eine
  Kachel unter einem Chart, die keine seiner Kurven zeigt, verwirrt mehr, als sie nützt.
- **`sizeY: 1` für die Kachelzeile** — bei 12 Spalten wäre diese Zeile 210 px hoch und
  das Layout unbrauchbar. Genau hier merkt man die Spaltenzahl.

Mit `"columns": 12` statt 60 lässt sich dasselbe Beispiel gegenprobieren: die
Kachelzeile wird so hoch wie das halbe Chart, und die Breiten 7 + 8 passen gar nicht
mehr ins Raster.

## Zum Nachmessen

Alles oben lässt sich in der Entwicklerkonsole in einer Zeile prüfen:

```js
// Spaltenzahl und wirksame Zeilenhöhe des aktuellen Dashboards
let s = angular.element(document.querySelector('.gridster')).scope();
while (s && !s.gridster) s = s.$parent;
({ columns: s.gridster.columns, rowHeight: s.gridster.rowHeight,
   curColWidth: Math.round(s.gridster.curColWidth),
   curRowHeight: Math.round(s.gridster.curRowHeight), minSizeY: s.gridster.minSizeY })
```

`minSizeY: 1` bestätigt: es gibt **keine** künstliche Untergrenze. Was begrenzt, ist
allein die Zeilenhöhe — und die kommt aus der Spaltenzahl.
