# Schreibstil-Anweisungen für Vorlesungsskripte

## Format
- Jupyter Book Format mit `{code-cell}` und `{admonition}`-Blöcken
- Sprache: Deutsch

## Struktur eines Kapitels
- Kurze **Motivation** am Anfang, die an Bekanntes anknüpft ("Bisher haben wir… In diesem Kapitel…")
- **Lernziele** als Checkliste mit Checkboxen, direkt nach der Motivation
- Roter Faden: Code-Beispiel → Erklärung & Verallgemeinerung → Mini-Übung → Lösung
- Kurze **Zusammenfassung** am Ende mit explizitem Ausblick auf das nächste Kapitel

## Sprache
- In den **Lernzielen**: "Sie können…", "Sie wissen…" (handlungsorientiert)
- Im **restlichen Text**: "Wir" (kein "man", kein "Sie")
- Kurze, klare Sätze
- Keine Gedankenstriche
- Fachbegriffe werden beim ersten Auftreten **fett** markiert und sofort erklärt
- Kein Lehrbuch-Jargon, sondern pragmatische Erklärungen

## Didaktik: Code zuerst
- Erst ein konkretes, lauffähiges Code-Beispiel zeigen
- Dann das Konzept dahinter erklären und verallgemeinern ("Wir sehen, dass… Das nennt man…")
- Niemals erst die Theorie, dann den Code

## Storytelling
- Ein durchgehendes Beispiel pro Kapitel, das sich wie ein roter Faden zieht
- Idealerweise wird das Beispiel im nächsten Kapitel wieder aufgegriffen
- Beispiele sind konsequent **ingenieurnah**: physikalische Größen mit Einheiten (Geschwindigkeit, Kraft, Schwingung etc.)
- Keine abstrakten Variablennamen wie `foo`, `bar`, `x`, `y`

## Variablennamen im Code
- Ausschließlich **ASCII-Zeichen**: keine Umlaute (ä, ö, ü) und kein ß in Variablennamen
- Erlaubt: `messwerte`, `groesse`, `standardabweichung`, `daempfung`
- Nicht erlaubt: `messwärte`, `größe`, `dämpfung`
- Kommentare und Strings im Code dürfen Umlaute enthalten

## Admonition-Blöcke
- `attention` für Lernziele (als Checkliste mit `* [ ]`)
- `note` für konzeptuelle Hinweise und Unterschiede (z.B. Liste vs. Array)
- `tip` für Mini-Übungen und deren Lösungen
- Lösungen immer als `dropdown`, mit Code **und** kurzer Erklärung des Ergebnisses

## Beispiel für einen Lernziel-Block
```
{admonition} Lernziele
:class: attention
* [ ] Sie wissen, was ein **NumPy-Array** ist.
* [ ] Sie können ein Array mit `np.array()` erzeugen.
```

## Beispiel für eine Mini-Übung mit Lösung
```
{admonition} Mini-Übung
:class: tip
Aufgabentext...

{admonition} Lösung
:class: tip
:class: dropdown
Code + kurze Erklärung des Ergebnisses
```

---

## Struktur des Übungskapitels (sec04)

Das vierte Kapitel enthält ausschließlich Übungsaufgaben für das Selbststudium.
Es hat keine erklärenden Texte, keine Motivation und keine Zusammenfassung.

### Schwierigkeitsgrade
Jede Aufgabe trägt einen Schwierigkeitsgrad im Titel:
- `✩` Verständnisaufgaben: Code vorhersagen, erklären, Ausgaben benennen
- `✩✩` Anwendungsaufgaben: eigenen Code schreiben, Funktionen definieren, Ergebnisse interpretieren
- `✩✩✩` Mini-Projekte: mehrere Konzepte des Kapitels kombinieren, komplexere Problemstellungen

### Aufgabenformat
```
{admonition} Übung X.Y (✩)
:class: tip
Aufgabentext mit nummerierten Teilaufgaben...

{admonition} Lösung
:class: tip
:class: dropdown
Code + Erklärung des fachlichen Ergebnisses
```

### Inhaltliche Merkmale
- ✩-Aufgaben: vorgegebener Code, Fragen zu Ausgabe und Verhalten, am Ende "Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen"
- ✩✩-Aufgaben: eigener Code mit **EVA-Kommentaren** (`# Eingabe`, `# Verarbeitung`, `# Ausgabe`) strukturieren, oft mit Funktionen und Docstrings
- ✩✩✩-Aufgaben (Mini-Projekte): mehrere benannte Teile (`**Teil 1:**`, `**Teil 2:**` usw.), Abschlussfrage zur Reflexion oder Erklärung

### Lösungen im sec04
- Zeigen vollständigen, lauffähigen Code mit EVA-Kommentaren
- Geben die erwartete Ausgabe als Codeblock an
- Erklären das **fachliche Ergebnis** in ein bis drei Sätzen ("Nach 10 Sekunden fallen Körper auf dem Mond 81 m…")
- Bei Mini-Projekten wird die Abschlussfrage ausführlich beantwortet
