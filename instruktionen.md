# Schreibstil-Anweisungen für Vorlesungsskripte

## Kontext

Dieses Projekt enthält das Vorlesungsmaterial für das Modul **Angewandte Numerik
im Maschinenbau** an der Technischen Hochschule Mannheim (Prof. Dr. Simone
Gramsch) für Studierende des Bachelor-Studiengangs Maschinenbau. Das Material
wird mit **MyST Markdown** erstellt und ist für **Jupyter Book** konzipiert.

## Format

- Jupyter Book 2 Format = Myst Markdown mit `{code-cell}` und
  `{admonition}`-Blöcken
- Sprache: Deutsch

## Struktur eines Kapitels

- Kurze **Motivation** am Anfang, die an Bekanntes anknüpft ("Bisher haben wir…
  In diesem Kapitel…")
- **Lernziele** als Checkliste mit Checkboxen, direkt nach der Motivation
- Roter Faden: Code-Beispiel → Erklärung & Verallgemeinerung → Mini-Übung →
  Lösung
- Kurze **Zusammenfassung** am Ende mit explizitem Ausblick auf das nächste
  Kapitel

## Sprache

- In den **Lernzielen**: "Sie können…", "Sie wissen…" (handlungsorientiert)
- Im **restlichen Text**: "Wir" (kein "man", kein "Sie")
- Kurze, klare Sätze
- Keine Gedankenstriche
- Fachbegriffe werden beim ersten Auftreten **fett** markiert und sofort erklärt
- Kein Lehrbuch-Jargon, sondern pragmatische Erklärungen

## Didaktik: Code zuerst

- Erst ein konkretes, lauffähiges Code-Beispiel zeigen
- Dann das Konzept dahinter erklären und verallgemeinern ("Wir sehen, dass… Das
  nennt man…")
- Niemals erst die Theorie, dann den Code

## Leseverhalten und Mini-Übungen

### Beobachtung zum Leseverhalten

Studierende im Maschinenbau überspringen erfahrungsgemäß den Fließtext
zwischen Code-Zellen und springen direkt zum Code und zu den Übungen.
Konzeptuelle Erklärungen im Fließtext werden daher häufig nicht gelesen.

### Konsequenzen für die Gestaltung

**Code-Kommentare als primärer Erklärungskanal:** Wichtige Konzepte, die
Studierende verstehen müssen, gehören als Kommentare direkt in die
Code-Zelle, nicht nur in den Fließtext davor. Der Fließtext dient als
vertiefende Erklärung für Studierende, die ihn lesen, aber das Wesentliche
muss auch ohne ihn verständlich sein.

**Viele kleine Mini-Übungen statt einer großen:** Pro Unterabschnitt eine
kurze Mini-Übung einbauen, unmittelbar nach dem zugehörigen Code-Beispiel.
Nicht alle Mini-Übungen am Ende der Sektion sammeln. So werden Studierende
gezwungen, sich mit jedem Teilkonzept aktiv auseinanderzusetzen.

**Verständnisfragen erzwingen das Lesen:** Mindestens eine Teilaufgabe pro
Mini-Übung sollte eine Verständnisfrage sein, die sich nicht durch bloßes
Ausführen von Code beantworten lässt. Geeignete Formate sind:

- Ausgabe vorhersagen, bevor der Code ausgeführt wird ("Was gibt `A[1, 2]`
  zurück? Was bedeutet dieser Wert inhaltlich?")
- Einen Eintrag inhaltlich lokalisieren ("In welcher Zeile von `A` steckt
  die Information über Tag 2?")
- Eine Beobachtung in eigenen Worten formulieren ("Warum ist
  `det_singular` nicht exakt null?")
- Eine Konsequenz ohne Code begründen ("Was würde eine Determinante von
  null in diesem Kontext bedeuten?")

### Selbstgesteuertes Lernen (Flipped-Classroom-Modus)

Wird das Material ohne Präsenzbegleitung eingesetzt (Studierende arbeiten
eigenständig, Lehrperson steht nur für Rückfragen bereit), gelten folgende
zusätzliche Anforderungen:

**Code-Zellen zusammenhalten:** Viele kleine Code-Zellen, die nur einen
Teilschritt zeigen, zerhacken den Lesefluss für Studierende, die direkt zum
Code springen. Zusammengehörige Schritte (Aufstellen, Lösbarkeitsprüfung,
Lösen, Probe) gehören in eine einzige Code-Zelle, sofern sie inhaltlich eine
Einheit bilden. Jeder Block innerhalb der Zelle wird mit einem
`# --- Abschnittskommentar ---` eingeleitet.

**Herleitung im Code, nicht nur im Fließtext:** Wenn Studierende eine
Matrixstruktur oder eine Formel nicht aus dem Fließtext kennen (weil sie ihn
überspringen), müssen sie sie aus den Code-Kommentaren rekonstruieren können.
Jede Zeile von `A` und jeder Eintrag von `b` erhält einen Kommentar, der die
zugrundeliegende umgeformte Gleichung explizit zeigt.

**Fließtext bleibt erhalten:** Er dient als vertiefende Schicht für
Studierende, die lesen. Er darf konzeptuelle Erklärungen und physikalische
Zusammenhänge enthalten, die im Code nur angedeutet sind. Das Wesentliche
muss aber auch ohne ihn verständlich sein.

**Stolperstellen explizit ansprechen:** Physikalisch oder mathematisch
überraschende Ergebnisse (z.B. negatives Vorzeichen eines Wärmestroms,
`det_singular` nicht exakt null) werden im Fließtext direkt nach der
Code-Zelle in ein bis zwei Sätzen erklärt. Studierende, die ohne Begleitung
arbeiten, haben keine Möglichkeit, spontan nachzufragen.

### Zeitschätzung für Sektionen

Als Faustregel gilt: Studierende verbringen kaum Zeit mit dem Lesen des
Fließtexts. Die tatsächliche Bearbeitungszeit setzt sich zusammen aus:

- ca. 3 bis 5 Minuten pro Code-Zelle (nachvollziehen, ausführen,
  verstehen)
- ca. 10 bis 15 Minuten pro Mini-Übung (je nach Schwierigkeit)
- Wiederholungsanteil aus früheren Kapiteln reduziert die Zeit spürbar

Eine Sektion mit 3 Code-Zellen und 2 Mini-Übungen erfordert also etwa
30 bis 40 Minuten. Sektionen mit Wiederholungsanteil aus dem Vorkapitel
können dabei am unteren Ende der Schätzung liegen.

## Storytelling

- Ein durchgehendes Beispiel pro Kapitel, das sich wie ein roter Faden zieht
- Idealerweise wird das Beispiel im nächsten Kapitel wieder aufgegriffen
- Beispiele sind konsequent **ingenieurnah** oder stammen aus dem **Alltag** der
  Studierenden: physikalische Größen mit Einheiten (Geschwindigkeit, Kraft,
  Schwingung etc.)
- Keine abstrakten Variablennamen wie `foo`, `bar`, `x`, `y`

## MyST-Formatkonventionen

### Dateianfang

Dateien beginnen immer mit dem YAML-Header:

```yaml
---
kernelspec:
  name: python3
  display_name: 'Python 3'
---
```

### Lernziele (Beginn jeder Sektion)

````markdown
## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Lernziel 1
* [ ] Lernziel 2
```

Beispiel:

```{admonition} Lernziele
:class: attention
* [ ] Sie wissen, was ein **NumPy-Array** ist.
* [ ] Sie können ein Array mit `np.array()` erzeugen.
```

### Ausführbare Code-Zellen

```{code-cell} python
# Python-Code hier
print("Beispiel")
```

### Mini-Übungen (innerhalb der Vorlesungskapitel)

````markdown
```{admonition} Mini-Übung
:class: tip
Aufgabentext hier.
```

```{code-cell} python
# Code-Zelle

```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Musterlösung hier
```
````

## Beispiel für eine Mini-Übung mit Lösung

```{admonition} Mini-Übung
:class: tip
Aufgabentext...

{admonition} Lösung
:class: tip
:class: dropdown
Code + kurze Erklärung des Ergebnisses
```

## Variablennamen im Code

- Variablennamen und Funktionsnamen ausschließlich **ASCII-Zeichen**: keine
  Umlaute (ä, ö, ü) und kein ß
- Erlaubt: `messwerte`, `groesse`, `standardabweichung`, `daempfung`
- Nicht erlaubt: `messwärte`, `größe`, `dämpfung`
- **Kommentare und Strings dürfen und sollen Umlaute enthalten**, damit sie
  natürliches Deutsch bleiben und gut lesbar sind. Beispiel:
  `# Temperaturdifferenz über jede Schicht` ist korrekt,
  `# Temperaturdifferenz ueber jede Schicht` ist zu vermeiden.

## Admonition-Blöcke

- `attention` für Lernziele (als Checkliste mit `* [ ]`)
- `note` für konzeptuelle Hinweise und Unterschiede (z.B. Liste vs. Array)
- `tip` für Mini-Übungen und deren Lösungen
- Lösungen immer als `dropdown`, mit Code **und** kurzer Erklärung des
  Ergebnisses

## Struktur des Übungskapitels (sec04)

Das vierte Kapitel enthält ausschließlich Übungsaufgaben für das Selbststudium.
Es hat keine erklärenden Texte, keine Motivation und keine Zusammenfassung.

### Schwierigkeitsgrade

Jede Aufgabe trägt einen Schwierigkeitsgrad im Titel:

- `✩` Verständnisaufgaben: Code vorhersagen, erklären, Ausgaben benennen
- `✩✩` Anwendungsaufgaben: eigenen Code schreiben, Funktionen definieren,
  Ergebnisse interpretieren
- `✩✩✩` Mini-Projekte: mehrere Konzepte des Kapitels kombinieren, komplexere
  Problemstellungen

### Aufgabenformat

```{admonition} Übung X.Y (✩)
:class: tip
Aufgabentext mit nummerierten Teilaufgaben...

{admonition} Lösung
:class: tip
:class: dropdown
Code + Erklärung des fachlichen Ergebnisses
```

### Inhaltliche Merkmale

- ✩-Aufgaben: vorgegebener Code, Fragen zu Ausgabe und Verhalten, am Ende
  "Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen"
- ✩✩-Aufgaben: eigener Code mit **Kommentaren** strukturieren, oft mit
  Funktionen und Docstrings
- ✩✩✩-Aufgaben (Mini-Projekte): mehrere benannte Teile (`**Teil 1:**`, `**Teil
  2:**` usw.), Abschlussfrage zur Reflexion oder Erklärung

### Lösungen im sec04

- Zeigen vollständigen, lauffähigen Code mit Kommentaren
- Geben die erwartete Ausgabe als Codeblock an
- Erklären das **fachliche Ergebnis** in ein bis drei Sätzen ("Nach 10 Sekunden
  fallen Körper auf dem Mond 81 m…")
- Bei Mini-Projekten wird die Abschlussfrage ausführlich beantwortet

## Qualitätscheckliste vor dem Abliefern

Vor dem Ausgeben einer Sektion prüfen:

- [ ] YAML-Header vorhanden?
- [ ] Einleitung beginnt mit einem konkreten Szenario oder Problem, nicht mit
  einer Definition?
- [ ] Lernziele mit `* [ ]` formatiert, unmittelbar nach dem H1-Titel?
- [ ] Unterabschnittsüberschriften als Fragen oder natürliche Aussagen formuliert?
- [ ] Prinzip "Erst Beispiel, dann abstrakt" eingehalten?
- [ ] Mindestens eine rhetorische Frage im Fließtext, kursiv gesetzt?
- [ ] Mindestens ein Rückverweis auf eine frühere Sektion?
- [ ] Mindestens ein konkreter Vorwärtsverweis auf ein späteres Thema?
- [ ] Kein Gedankenstrich im Fließtext?
- [ ] Kein `d.h.` oder `z.B.` mitten im Satz?
- [ ] Wir-Perspektive im Fließtext, Sie-Anrede nur in den Lernzielen?
- [ ] YouTube-Videos direkt nach dem jeweiligen Unterabschnitt eingebettet,
  nicht am Ende der Sektion gesammelt?
- [ ] Mini-Übungen mit Lösungs-Dropdown versehen?
- [ ] Pro Unterabschnitt eine Mini-Übung direkt nach dem Code-Beispiel,
  nicht alle am Ende gesammelt?
- [ ] Mindestens eine Teilaufgabe pro Mini-Übung ist eine Verständnisfrage,
  die sich nicht durch bloßes Ausführen von Code beantworten lässt?
- [ ] Wichtige Konzepte stehen als Kommentare im Code, nicht nur im
  Fließtext?
- [ ] Zusammenfassung mit konkretem Ausblick auf die nächste Sektion?

## TikZ-Abbildungen

Alle Abbildungen werden als eigenständige `standalone`-Dokumente in LaTeX
erstellt, nach SVG exportiert und über eine `{figure}`-Direktive in die
MyST-Markdown-Quelldatei eingebunden. Die Kompilierung erfolgt mit `lualatex`.

### Präambel-Vorlage

Jede TikZ-Datei verwendet ausnahmslos die folgende Präambel:

```latex
\documentclass[11pt]{standalone}
\usepackage{amssymb}
\usepackage{amsmath}
\usepackage[no-math]{fontspec}
\usepackage{unicode-math}
\setmainfont{Libertinus Sans}
\setsansfont{Libertinus Sans}
\setmathfont{Libertinus Math}
\usepackage{pgf,xcolor}
\usepackage{tikz}
\usetikzlibrary{arrows.meta, backgrounds}
\usepackage{pgfplots}
\pgfplotsset{compat=newest}
\usepackage{pifont}
```

`pgfplots` wird in jeder Datei eingebunden, auch wenn die Abbildung nur
reines TikZ verwendet, damit die Präambel über alle Dateien identisch bleibt.

### Schrift

Die Abbildungen verwenden **Libertinus Sans** als Textschrift und
**Libertinus Math** als Mathematikschrift. Libertinus Sans passt als
serifenlose Schrift zum Standard-Theme von Jupyter Book; Libertinus Math
stellt sicher, dass Formeln in Abbildungen auch in späteren Kapiteln
korrekt gesetzt werden. Das Paket `\usepackage{libertinus}` wird nicht
mehr verwendet, da die Schriften nun explizit über `fontspec` und
`unicode-math` konfiguriert werden. Keine anderen Schriftfamilien verwenden
(insbesondere nicht TeX Gyre Heros oder Computer Modern).

### Farbpalette

Alle sieben Farben werden in jeder Datei definiert, auch wenn nur eine
Teilmenge verwendet wird:

```latex
\definecolor{my_darkgray}{HTML}{484949}
\definecolor{my_lightgray}{HTML}{F3F4F4}
\definecolor{my_darkblue}{HTML}{005A94}
\definecolor{my_lightblue}{HTML}{CCDEE9}
\definecolor{my_yellow}{HTML}{FFEC7F}
\definecolor{my_red}{HTML}{E60000}
\definecolor{my_orange}{HTML}{E87846}
```

Keine ad-hoc-Farbnamen einführen. Alle Farben kommen ausnahmslos aus
dieser Palette.

**Zweifarbige Abbildungen (Standardfall):** `my_darkblue` für das primäre
Element (Konturlinie, Hauptkurve, dominante Region) und `my_lightblue` für
das sekundäre Element (Füllung, Hintergrundregion, zweite Kurve).

**Dreifarbige Funktionsgraphen:** `my_darkblue` für die erste Kurve,
`my_red` für die zweite, `my_orange` für die dritte. `my_yellow` nicht für
Kurvenlinien verwenden (zu wenig Kontrast). `my_yellow` ist für
hervorgehobene Füllregionen reserviert.

**Text und Annotationen:** `my_darkgray` für sekundäre Labels.
`my_darkblue` für Labels, die mathematische oder technische Objekte
bezeichnen.

### Hintergrundpanel

Jede Abbildung hat ein explizites Hintergrundpanel mit `my_lightgray`.
Dies gewährleistet korrekte Darstellung in hellen und dunklen
Browser-Themes:

```latex
\begin{tikzpicture}[
    show background rectangle,
    background rectangle/.style={fill=my_lightgray, rounded corners=8pt},
    inner frame sep=0.8cm
]
```

`inner frame sep=0.3cm` nur für breite, flache Abbildungen (zum Beispiel
Zahlenstrahlen). In allen anderen Fällen `0.8cm`.

### Achsenstil für Funktionsgraphen (pgfplots)

```latex
\begin{axis}[
    axis lines = center,
    xlabel = {$x$},
    ylabel = {$y$},
    grid = both,
    axis equal,
    axis line style={thick},
]
```

`axis equal` nur weglassen, wenn das natürliche Seitenverhältnis der
Funktion den Graphen unleserlich macht. Die Ausnahme mit einem Kommentar
dokumentieren. Alle Kurven:

```latex
\addplot[draw=my_darkblue, samples=300, ultra thick, domain=a:b]{ ... };
```

### Legenden

Bei mehr als einer Kurve eine Legende einfügen:

```latex
legend pos=north west,
legend style={font=\small},
legend cell align=left,
```

`\addlegendentry{...}` direkt nach dem jeweiligen `\addplot`-Aufruf.

### Schematische Diagramme und technische Abbildungen

Für Diagramme ohne Koordinatenachsen (annotierte Skizzen, Konzeptdiagramme,
Mehrpanel-Vergleiche) gelten folgende Ergänzungen zur allgemeinen Farbpalette.

**Annotationspfeil:** Beschriftungspfeile verwenden einheitlich den
Stealth-Pfeilkopf in `my_darkgray`. Die Stil-Definition gehört in das
optionale Argument der `tikzpicture`-Umgebung:

```latex
annotation/.style={draw=my_darkgray, thick, -{Stealth[length=7pt, width=5pt]}}
```

Labels neben Annotationspfeilen: `font=\small, text=my_darkgray`.

**Mehrpanel-Layout:** Panels werden mit `\begin{scope}[xshift=...]`
nebeneinander gesetzt. Der Abstand zwischen zwei Panels beträgt mindestens
`1cm`. Vertikale Trennlinien zwischen Panels sind nicht erforderlich.

**Panel-Untertitel:** Jedes Panel erhält einen Titel direkt unterhalb des
Inhalts, zentriert:

```latex
\node[font=\small\bfseries, text=my_darkgray, anchor=north] at (...) {...};
```

**Hervorgehobene Elemente:**

- Fläche oder Füllregion: `fill=my_yellow`
- Linienelement (Kante, Achse): `draw=my_orange, ultra thick`
- Knotenpunkt: `fill=my_orange` mit weißem Rand (`draw=my_lightgray, thick`)

**Bewertungssymbole:** ✓ und ✗ werden über das Paket `pifont` gesetzt
(bereits in der Präambel). `\ding{51}` für ✓, `\ding{55}` für ✗.
Unicode-Zeichen dürfen in Kommentaren erscheinen, aber nicht im ausführbaren
LaTeX-Code. `font=\large\bfseries`. ✓ in `my_darkblue`, ✗ in `my_red`.

### Beschriftung von Schichtendiagrammen (Wandquerschnitte, Schichtmodelle)

Schichtendiagramme zeigen physikalische Schichten als nebeneinanderliegende
Rechtecke. Die Breite jeder Schicht ist proportional zur dominanten
physikalischen Größe (thermischer Widerstand, elektrischer Widerstand,
Dicke etc.).

**Schichtfarben:** Zwei alternierende Füllfarben `my_lightblue` und
`my_yellow`. `my_yellow` hebt die physikalisch bedeutsamste Schicht hervor
(größter Widerstand, stärkste Dämmwirkung). Sind alle Schichten gleichwertig,
nur `my_lightblue` verwenden.

**Schichtbeschriftungen:** Text innerhalb der Schicht, rotiert um 90°
(`rotate=90`), damit auch schmale Schichten lesbar bleiben.
`font=\small, text=my_darkblue`.

**Randwerte:** Temperaturen oder andere Randbedingungen als Annotationen
mit Stealth-Pfeil links und rechts außerhalb der Wand.

**Temperatur- oder Zustandsprofil:** Stückweise lineares Profil als
`ultra thick`-Linie in `my_darkblue` oberhalb des Querschnitts. Die
x-Achse des Profils entspricht der kumulierten physikalischen Größe
(z.B. kumulierter thermischer Widerstand). Grenzflächenpositionen durch
gestrichelte `my_darkgray`-Linien markieren, die beide Bereiche verbinden.
Punkte an Grenzflächen: `fill=my_orange, draw=my_lightgray`.

**Fluss-Pfeil:** Physikalischen Fluss (Wärmestrom, Stromfluss) als
`ultra thick`-Pfeil in `my_orange` unterhalb des Querschnitts, Richtung
entsprechend dem tatsächlichen Fluss (nicht Vorzeichenkonvention).

### Einbindung in MyST Markdown

SVG-Dateien werden im Unterordner `bilder/` abgelegt und mit der
`{figure}`-Direktive eingebunden:

````markdown
```{figure} pics/dateiname.svg
:alt: Kurze Beschreibung für Screenreader
:align: center

Darstellung von [was die Abbildung zeigt].
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```
````

Den beschreibenden Teil der Bildunterschrift auf einen Satz begrenzen.
Keine Informationen wiederholen, die bereits im umgebenden Fließtext stehen.

### Dateinamen

Dateinamen beschreibend, in Kleinbuchstaben mit Unterstrichen, ohne
Kapitel- oder Sektionsnummer. Das Sprachkürzel `_DE` oder `_EN` nur
anhängen, wenn zwei Sprachversionen derselben Abbildung existieren.
