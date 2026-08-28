# Schreibstil-Anweisungen für Vorlesungsskripte

## Kontext

Dieses Projekt enthält das Vorlesungsmaterial für das Modul **Angewandte Numerik
im Maschinenbau** an der Technischen Hochschule Mannheim (Prof. Dr. Simone
Gramsch) für Studierende des Bachelor-Studiengangs Maschinenbau im 5. Semester.
Das Modul umfasst 4 SWS und 5 ECTS. Das Material wird mit **MyST Markdown**
erstellt und ist für **Jupyter Book 2** konzipiert. Die HTML-Version wird auf
GitHub Pages veröffentlicht.

Ziel des Moduls ist es, den Studierenden die wichtigsten Numerik-Konzepte an
Beispielen aus dem Maschinenbau zu demonstrieren. Die Darstellung ist bewusst
pragmatisch, nicht lehrbuchartig.

## Format

- Jupyter Book 2 Format = MyST Markdown mit `{code-cell}` und
  `{admonition}`-Blöcken
- Sprache: Deutsch

## Struktur eines Parts

Ein **Part** umfasst den Stoff einer Woche und entspricht 2× 90 min Präsenzzeit
im Computerraum. Ein Part entspricht einem `chapterXX`-Ordner und besteht aus
fünf Kapiteldateien. Jede Datei ist ein **Kapitel** in der Nummerierung `X.Y`
(zum Beispiel ist `chapter02_sec01.md` das Kapitel 2.1). Die H2-Überschriften
innerhalb einer Datei heißen **Abschnitte**. Das Wort "Sektion" wird nicht
verwendet.

| Datei | Kontext | Dauer | Beschreibung |
| ----- | ------- | ----- | ------------ |
| `chapterXX_sec01.md` | 1. Vorlesung, erste Hälfte | 45 min | Code-Along als Jupyter Notebook, von der Lehrperson präsentiert |
| `chapterXX_sec02.md` | 1. Vorlesung, zweite Hälfte | 45 min | Vertiefungsprojekt, paarweise bearbeitet |
| `chapterXX_sec03.md` | 2. Vorlesung, erste Hälfte | 45 min | Code-Along als Jupyter Notebook, von der Lehrperson präsentiert |
| `chapterXX_sec04.md` | 2. Vorlesung, zweite Hälfte | 45 min | Vertiefungsprojekt, paarweise bearbeitet |
| `chapterXX_sec05.md` | Selbststudium zuhause | ca. 90 min | Übungsaufgaben ✩/✩✩/✩✩✩ mit Musterlösungen |

### Code-Along-Kapitel (sec01, sec03)

Aufbau:

1. **Einführung**: kurze Motivation, die an Bekanntes anknüpft ("Bisher haben
   wir… In diesem Kapitel…")
2. **Lernziele** als Checkliste mit `* [ ]`, direkt nach der Einführung,
   H2-Überschrift "Lernziele"
3. **Möglichst drei inhaltliche H2-Abschnitte**. Jeder Abschnitt:
   - ist so bemessen, dass seine Präsentation im Code-Along ca. 10 min dauert
   - folgt dem roten Faden Code-Beispiel → Erklärung und Verallgemeinerung
   - erhält später ein eigenes Erklärvideo von ca. 10 min
   - **endet mit einer Mini-Übung (✩)** (siehe Abschnitt "Mini-Übungen")
4. **Zusammenfassung und Ausblick** mit explizitem Vorgriff auf das nächste
   Kapitel

### Vertiefungskapitel (sec02, sec04)

Die Studierenden erweitern und vertiefen das Gelernte, möglichst paarweise.
Aufbau:

1. **Kurze Einführung** (zwei bis drei Sätze): Bezug zum Code-Along, Hinweis auf
   Partnerarbeit
2. **Ein zusammenhängendes Projekt (✩✩)** als Pflichtteil: eine durchgehende
   ingenieurnahe Problemstellung mit benannten Teilschritten (`**Teil 1:**`,
   `**Teil 2:**`, …), die aufeinander aufbauen. Endet mit einer Abschlussfrage
   zur Reflexion.
3. **Ein bis zwei optionale Zusatzaufgaben (✩✩✩)** für Paare, die schneller
   fertig sind.

Jeder Teilschritt und jede Zusatzaufgabe hat eine Lösung als `dropdown` mit Code
und kurzer fachlicher Erklärung.

### Selbststudiumskapitel (sec05)

Siehe Abschnitt "Struktur des Selbststudiumskapitels (sec05)". Enthält
ausschließlich Übungsaufgaben mit Musterlösungen, keine erklärenden Texte, keine
Motivation, keine Lernziele, keine Zusammenfassung.

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

## Mini-Übungen

Jeder inhaltliche H2-Abschnitt in sec01 und sec03 endet mit genau einer
Mini-Übung, unmittelbar nach dem zugehörigen Code-Beispiel. Sie trägt den
Schwierigkeitsgrad ✩ im Titel.

**Zweck:** Die Mini-Übungen richten sich an Studierende, die asynchron
teilnehmen und daher kein begleitetes Code-Along haben. Für die
Präsenzpräsentation werden die Mini-Übungen durch ein Skript automatisiert aus
dem Notebook entfernt. Jede Mini-Übung muss deshalb allein aus dem
vorangehenden Text lösbar sein.

**Viele kleine Mini-Übungen statt einer großen:** Pro H2-Abschnitt eine kurze
Mini-Übung einbauen, unmittelbar nach dem zugehörigen Code-Beispiel. Nicht alle
Mini-Übungen am Ende des Kapitels sammeln. So werden Studierende gezwungen, sich
mit jedem Teilkonzept aktiv auseinanderzusetzen.

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

## Selbstgesteuertes Lernen (Flipped-Classroom-Modus)

Das Material muss auch ohne Präsenzbegleitung funktionieren: Studierende, die
asynchron teilnehmen, arbeiten die Code-Along-Kapitel sec01 und sec03
eigenständig durch und lösen dabei die Mini-Übungen, die in der Präsenzversion
entfernt sind. Daher gelten folgende zusätzliche Anforderungen:

**Stolperstellen explizit ansprechen:** Physikalisch oder mathematisch
überraschende Ergebnisse (z.B. negatives Vorzeichen eines Wärmestroms,
`det_singular` nicht exakt null) werden im Fließtext direkt nach der
Code-Zelle in ein bis zwei Sätzen erklärt. Studierende, die ohne Begleitung
arbeiten, haben keine Möglichkeit, spontan nachzufragen.

## Storytelling

- Jedes Kapitel (sec01, sec02, sec03, sec04) hat nach Möglichkeit ein eigenes
  durchgehendes Beispiel, das sich innerhalb des Kapitels wie ein roter Faden
  zieht. In den Code-Along-Kapiteln verbindet es die drei H2-Abschnitte, in den
  Vertiefungskapiteln ist es das Projekt selbst.
- Die Kapitel eines Parts müssen thematisch zusammenhängen, aber kein
  gemeinsames Beispiel teilen. sec02 knüpft an sec01 an, sec04 an sec03.
- Ein gelungenes Beispiel wird idealerweise in einem späteren Part wieder
  aufgegriffen.
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

### Lernziele (Beginn jedes Code-Along-Kapitels)

````markdown
## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Lernziel 1
* [ ] Lernziel 2
```
````

Beispiel:

````markdown
```{admonition} Lernziele
:class: attention
* [ ] Sie wissen, was ein **NumPy-Array** ist.
* [ ] Sie können ein Array mit `np.array()` erzeugen.
```
````

### Ausführbare Code-Zellen

````markdown
```{code-cell} python
# Python-Code hier
print("Beispiel")
```
````

### Mini-Übungen (in den Code-Along-Kapiteln)

```{admonition} Mini-Übung (✩)
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
Kurze Erklärung des Ergebnisses.
````

### Vertiefungsprojekt (sec02, sec04)

Das Kapitel beginnt mit zwei bis drei einleitenden Sätzen (Bezug zum Code-Along,
Hinweis auf Partnerarbeit). Dann folgt ein Projektkopf mit dem Szenario, danach
die Teilschritte. Jeder Teilschritt hat eine eigene Code-Zelle und eine eigene
Lösung als `dropdown`, damit die Paare sich während der Bearbeitung schrittweise
selbst kontrollieren können.

```{admonition} Projekt: <sprechender Titel> (✩✩)
:class: tip
Kurze Beschreibung des ingenieurnahen Szenarios mit den gegebenen Größen.
```

```{admonition} Teil 1: <kurzer Titel>
:class: tip
Aufgabentext des ersten Teilschritts.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
```python
# lauffähige Musterlösung mit Kommentaren
```
Kurze fachliche Erklärung des Ergebnisses.
````

```{admonition} Teil 2: <kurzer Titel>
:class: tip
Zweiter Teilschritt, baut auf Teil 1 auf.
```

... (weitere Teilschritte analog)

```{admonition} Abschlussfrage
:class: tip
Reflexions- oder Erklärfrage, die ohne Code beantwortet wird.
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
Ausführliche Antwort in mehreren Sätzen.
````

Danach ein bis zwei optionale Zusatzaufgaben, jeweils mit eigener Lösung:

```{admonition} Zusatzaufgabe: <Titel> (✩✩✩)
:class: tip
Aufgabentext ...
```

### Videos

**Eigene Erklärvideos:** Zu jedem inhaltlichen H2-Abschnitt in sec01 und sec03
entsteht ein Erklärvideo von ca. 10 min. Es wird als aufklappbarer Block
eingebunden, damit es den Lesefluss nicht stört, und zwar direkt nach der
H2-Überschrift, vor dem ersten Code-Beispiel:

````markdown
```{dropdown} Erklärvideo: <Titel des Abschnitts>
<iframe width="560" height="315" src="https://www.youtube.com/embed/<ID>"
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay;
clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
```
````

Die Videos sind noch nicht produziert. Bis dahin wird kein Platzhalter
eingefügt, die Stelle nach der H2-Überschrift bleibt frei.

**Kuratierte Fremdvideos:** Ergänzende Videos Dritter sind erlaubt. Sie stehen
als aufklappbarer Block am Ende des zugehörigen H2-Abschnitts, nach der
Mini-Übung, niemals am Ende des Kapitels gesammelt:

````markdown
```{dropdown} Video "<Titel>" von <Kanal>
<iframe width="560" height="315" src="https://www.youtube.com/embed/<ID>"
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay;
clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
```
````

## Variablennamen im Code

- Variablennamen und Funktionsnamen ausschließlich **ASCII-Zeichen**: keine
  Umlaute (ä, ö, ü) und kein ß
- Erlaubt: `messwerte`, `groesse`, `standardabweichung`, `daempfung`
- Nicht erlaubt: `meßwerte`, `größe`, `dämpfung`
- **Kommentare und Strings dürfen und sollen Umlaute enthalten**, damit sie
  natürliches Deutsch bleiben und gut lesbar sind. Beispiel:
  `# Temperaturdifferenz über jede Schicht` ist korrekt,
  `# Temperaturdifferenz ueber jede Schicht` ist zu vermeiden.

## Admonition-Blöcke

- `attention` für Lernziele (als Checkliste mit `* [ ]`)
- `note` für Definitionen und Erläuterungen von Fachbegriffen
- `tip` für Mini-Übungen, Vertiefungsprojekte, Teilschritte, Zusatzaufgaben,
  Übungsaufgaben und alle zugehörigen Lösungen
- `warning` für Hinweise und Stolperfallen
- `dropdown` für Videos und für alle Lösungen; Lösungen enthalten immer Code
  **und** eine kurze Erklärung des fachlichen Ergebnisses

## Struktur des Selbststudiumskapitels (sec05)

Die Datei `chapterXX_sec05.md` (Kapitel X.5) enthält die Übungsaufgaben für das
vertiefende Selbststudium zuhause (Richtwert 90 min). Sie ist unabhängig von den
Vertiefungsprojekten aus sec02 und sec04 und wiederholt deren Aufgaben nicht.
Abgesehen von einem kurzen Orientierungshinweis am Anfang enthält sie
ausschließlich Übungsaufgaben mit Musterlösungen: keine erklärenden Texte, keine
Motivation, keine Lernziele, keine Zusammenfassung.

### Orientierungshinweis am Anfang

Direkt nach der H1-Überschrift stehen wenige Zeilen Fließtext (kein
Admonition-Block): Sie nennen den wiederholten Stoff (Kapitel X.1 bis X.4) und
den Zeitrichtwert von rund 90 Minuten. Danach folgt eine Aufzählung der drei
Schwierigkeitsgrade mit je einer kurzen Beschreibung und einem Zeitrichtwert pro
Aufgabe:

```markdown
* ✩ Verständnis: Code und Ausgaben vorhersagen und erklären (ca. 5 min)
* ✩✩ Anwendung: eigenen Code schreiben und Ergebnisse interpretieren (ca. 10 min)
* ✩✩✩ Mini-Projekt: mehrere Konzepte des Parts kombinieren (ca. 30 min)
```

### Schwierigkeitsgrade

Jede Aufgabe trägt einen Schwierigkeitsgrad im Titel:

- `✩` Verständnisaufgaben: Code vorhersagen, erklären, Ausgaben benennen
- `✩✩` Anwendungsaufgaben: eigenen Code schreiben, Funktionen definieren,
  Ergebnisse interpretieren
- `✩✩✩` Mini-Projekte: mehrere Konzepte des Parts kombinieren, komplexere
  Problemstellungen

### Aufgabenformat

```{admonition} Aufgabe X.Y (✩)
:class: tip
Aufgabentext, bei Mini-Projekten mit benannten Teilen (`**Teil 1:**`, ...)
und einer Abschlussfrage.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# vollständiger, lauffähiger Code mit Kommentaren
```
Erwartete Ausgabe als Codeblock, dann Erklärung des fachlichen Ergebnisses in
ein bis drei Sätzen. Bei Mini-Projekten wird die Abschlussfrage ausführlich
beantwortet.
````

### Inhaltliche Merkmale

- ✩-Aufgaben: vorgegebener Code, Fragen zu Ausgabe und Verhalten, am Ende
  "Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen"
- ✩✩-Aufgaben: eigener Code, mit Kommentaren strukturiert, oft mit Funktionen
  und Docstrings
- ✩✩✩-Aufgaben: mehrere benannte Teile (`**Teil 1:**`, `**Teil 2:**` usw.),
  Abschlussfrage zur Reflexion oder Erklärung

### Lösungen

- Zeigen vollständigen, lauffähigen Code mit Kommentaren
- Geben die erwartete Ausgabe als Codeblock an
- Erklären das **fachliche Ergebnis** in ein bis drei Sätzen ("Nach 10 Sekunden
  fallen Körper auf dem Mond 81 m…")
- Bei Mini-Projekten wird die Abschlussfrage ausführlich beantwortet

## Qualitätscheckliste vor dem Abliefern

### Für alle Kapitel

- [ ] YAML-Header vorhanden?
- [ ] Variablennamen im Code nur ASCII, Kommentare und Strings mit Umlauten?
- [ ] Kein Gedankenstrich im Fließtext, kein `d.h.` oder `z.B.` mitten im Satz?
- [ ] Alle Lösungen als `dropdown` mit Code und fachlicher Erklärung?

### Zusätzlich für Code-Along-Kapitel (sec01, sec03)

- [ ] Einführung beginnt mit einem konkreten Szenario, nicht mit einer
  Definition?
- [ ] Lernziele mit `* [ ]`, unmittelbar nach der Einführung, Anrede "Sie"?
- [ ] Wir-Perspektive im übrigen Fließtext?
- [ ] Möglichst drei inhaltliche H2-Abschnitte, Überschriften als Fragen oder
  natürliche Aussagen?
- [ ] Prinzip "Erst Beispiel, dann abstrakt" in jedem H2-Abschnitt eingehalten?
- [ ] Mindestens eine kursiv gesetzte rhetorische Frage im Fließtext?
- [ ] Wo inhaltlich sinnvoll ein Rückverweis auf ein früheres Kapitel, immer
  ein konkreter Vorwärtsverweis?
- [ ] Wichtige Konzepte stehen als Kommentare im Code, nicht nur im Fließtext?
- [ ] Pro H2-Abschnitt genau eine Mini-Übung (✩), direkt nach dem Code-Beispiel?
- [ ] Jede Mini-Übung allein aus dem vorangehenden Text lösbar (asynchrone
  Teilnahme)?
- [ ] Mindestens eine Teilaufgabe pro Mini-Übung ist eine Verständnisfrage, die
  sich nicht durch bloßes Ausführen beantworten lässt?
- [ ] Fremdvideos als Dropdown am Ende des jeweiligen H2-Abschnitts, nicht
  gesammelt?
- [ ] Zusammenfassung mit konkretem Ausblick auf das nächste Kapitel?

### Zusätzlich für Vertiefungskapitel (sec02, sec04)

- [ ] Kurze Einführung mit Bezug zum Code-Along und Hinweis auf Partnerarbeit?
- [ ] Ein zusammenhängendes Projekt (✩✩) mit aufeinander aufbauenden
  Teilschritten?
- [ ] Jeder Teilschritt mit eigener Code-Zelle und eigener Lösung?
- [ ] Abschlussfrage zur Reflexion vorhanden?
- [ ] Ein bis zwei optionale Zusatzaufgaben (✩✩✩)?

### Zusätzlich für das Selbststudiumskapitel (sec05)

- [ ] Orientierungshinweis am Anfang (Stoffbezug, 90-min-Richtwert,
  Schwierigkeitsgrade mit Zeitrichtwert pro Aufgabe)?
- [ ] Ansonsten keine erklärenden Texte, keine Lernziele, keine Zusammenfassung?
- [ ] Jede Aufgabe mit Schwierigkeitsgrad im Titel?
- [ ] Mischung aus ✩, ✩✩ und ✩✩✩?
- [ ] ✩-Aufgaben mit "Führen Sie den Code aus und überprüfen Sie Ihre
  Vorhersagen"?
- [ ] Lösungen mit erwarteter Ausgabe als Codeblock?

## TikZ-Abbildungen

Alle Abbildungen werden als eigenständige `standalone`-Dokumente in LaTeX
erstellt, nach SVG exportiert und über eine `{figure}`-Direktive in die
MyST-Markdown-Quelldatei eingebunden. Die Kompilierung erfolgt mit `lualatex`.

### Präambel-Vorlage

Jede TikZ-Datei verwendet mindestens die folgende Präambel:

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

`pgfplots` wird in jeder Datei eingebunden, auch wenn die Abbildung nur reines
TikZ verwendet, damit die Präambel über alle Dateien identisch bleibt. Die
Präamble darf erweitert werden.

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

SVG-Dateien werden im Unterordner `pics/` abgelegt und mit der
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
Kapitel- oder Abschnittsnummer. Das Sprachkürzel `_DE` oder `_EN` nur
anhängen, wenn zwei Sprachversionen derselben Abbildung existieren.
