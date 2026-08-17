---
jupytext:
  formats: ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.15.2
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Verteifung 1.2

Die folgenden Aufgaben bearbeiten wir in der restlichen Zeit dieses
Blocks. Aufgaben mit einem Stern (✩) und zwei Sternen (✩✩) sind
Pflichtaufgaben. Die Aufgabe mit drei Sternen (✩✩✩) ist eine Zusatzaufgabe
für alle, die schneller fertig sind.

```{admonition} Übung A (✩)
:class: tip
Gegeben ist die Liste `messreihe = [72, 88, 95, 60, 110, 130]`. Notieren Sie
zunächst Ihre Vermutung, bevor Sie den Code ausführen.

* `messreihe[0]` -->
* `messreihe[-1]` -->
* `messreihe[2:4]` -->
* `len(messreihe)` -->

Überprüfen Sie anschließend jede Zeile in einer Code-Zelle.
```

```{code-cell} ipython3
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
messreihe = [72, 88, 95, 60, 110, 130]
print(messreihe[0])
print(messreihe[-1])
print(messreihe[2:4])
print(len(messreihe))
```
`messreihe[0]` ist `72`, das erste Element. `messreihe[-1]` ist `130`, das
letzte Element. `messreihe[2:4]` liefert den Ausschnitt `[95, 60]`, also die
Elemente an Index 2 und 3, denn der Endindex 4 gehört nicht mehr zum
Ausschnitt. `len(messreihe)` ist `6`, die Anzahl der Elemente.
````

```{admonition} Übung B (✩✩)
:class: tip
Gegeben ist die Liste `messreihe = [72, 88, 95, 60, 110, 130]` mit
Geschwindigkeiten in km/h. Berechnen Sie die durchschnittliche Geschwindigkeit
mit einer for-Schleife, die jeden Wert zu einer Summe aufaddiert. Verwenden
Sie dabei nicht die eingebaute Funktion `sum()`. Geben Sie das Ergebnis
gerundet auf eine Nachkommastelle aus.
```

```{code-cell} ipython3
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
messreihe = [72, 88, 95, 60, 110, 130]

summe = 0
for geschwindigkeit in messreihe:
    summe = summe + geschwindigkeit

durchschnitt = summe / len(messreihe)
print(f'Durchschnittliche Geschwindigkeit: {durchschnitt:.1f} km/h')
```
Die durchschnittliche Geschwindigkeit beträgt rund 92.5 km/h. Die Variable
`summe` wirkt hier als Akkumulator, der bei jedem Schleifendurchgang um den
aktuellen Messwert erhöht wird.
````

```{admonition} Übung C (✩✩)
:class: tip
Erstellen Sie ein Dictionary `pruefstand` für einen Prüfstand mit folgenden
Informationen:

* bezeichnung: `'Pruefstand_3'`
* max_geschwindigkeit_kmh: 180.0
* baujahr: 2021

Geben Sie dann aus: "Pruefstand_3 (Baujahr 2021) erlaubt bis zu 180.0 km/h."
```

```{code-cell} ipython3
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
pruefstand = {
    'bezeichnung': 'Pruefstand_3',
    'max_geschwindigkeit_kmh': 180.0,
    'baujahr': 2021
}

print(f'{pruefstand["bezeichnung"]} (Baujahr {pruefstand["baujahr"]}) '
      f'erlaubt bis zu {pruefstand["max_geschwindigkeit_kmh"]} km/h.')
```
Der Zugriff auf jeden Wert erfolgt über den passenden Schlüssel. Die
f-Strings lassen sich dabei über mehrere Zeilen verteilen, solange jede
Teilzeichenkette mit einem `f` beginnt.
````

```{admonition} Übung D (✩✩)
:class: tip
Schreiben Sie eine Funktion `kmh_zu_ms(geschwindigkeit_kmh)`, die eine
Geschwindigkeit von km/h in m/s umrechnet und zurückgibt. Rufen Sie die
Funktion anschließend in einer for-Schleife für jeden Wert der Liste
`messreihe = [72, 88, 95, 60, 110, 130]` auf und geben Sie jeweils km/h und
m/s aus.
```

```{code-cell} ipython3
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
def kmh_zu_ms(geschwindigkeit_kmh):
    return geschwindigkeit_kmh / 3.6

messreihe = [72, 88, 95, 60, 110, 130]

for geschwindigkeit_kmh in messreihe:
    geschwindigkeit_ms = kmh_zu_ms(geschwindigkeit_kmh)
    print(f'{geschwindigkeit_kmh} km/h entsprechen {geschwindigkeit_ms:.1f} m/s')
```
Da die Funktion einmal definiert wird, können wir sie beliebig oft mit
unterschiedlichen Argumenten aufrufen, ohne die Umrechnungsformel jedes Mal
neu hinzuschreiben.
````

```{admonition} Übung E (✩✩✩, Mini-Projekt)
:class: tip
Ein Beschleunigungstest liefert die Messreihe
`messreihe = [72, 88, 95, 60, 110, 130]` in km/h. Setzen Sie folgende
Schritte um.

**Teil 1:** Schreiben Sie eine Funktion `kinetische_energie(geschwindigkeit_kmh, masse=1200)`,
die zunächst intern in m/s umrechnet und anschließend die kinetische Energie
in Joule zurückgibt.

**Teil 2:** Durchlaufen Sie `messreihe` mit einer for-Schleife. Berechnen Sie
für jeden Messwert die kinetische Energie und merken Sie sich in einer
Variable `maximale_energie` den bisher größten berechneten Wert (Hinweis:
Starten Sie mit `maximale_energie = 0` vor der Schleife und vergleichen Sie
in jedem Durchgang mit einer `if`-Abfrage).

**Teil 3:** Geben Sie nach der Schleife den größten gefundenen Energiewert
aus. Beantworten Sie abschließend: Bei welcher Geschwindigkeit aus der
Messreihe tritt dieser Maximalwert auf, und warum lässt sich das ohne
Ausführen des Codes bereits vermuten?
```

```{code-cell} ipython3
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
def kinetische_energie(geschwindigkeit_kmh, masse=1200):
    geschwindigkeit_ms = geschwindigkeit_kmh / 3.6
    return 0.5 * masse * geschwindigkeit_ms**2

messreihe = [72, 88, 95, 60, 110, 130]

maximale_energie = 0
for geschwindigkeit_kmh in messreihe:
    energie = kinetische_energie(geschwindigkeit_kmh)
    if energie > maximale_energie:
        maximale_energie = energie

print(f'Maximale kinetische Energie: {maximale_energie:.1f} Joule')
```
Der Maximalwert tritt bei 130 km/h auf, dem größten Wert der Messreihe. Da
die kinetische Energie mit dem Quadrat der Geschwindigkeit wächst und die
Masse über die gesamte Messreihe konstant bleibt, liegt das Maximum der
Energie immer bei der höchsten gemessenen Geschwindigkeit. Diese Vermutung
lässt sich also bereits aus der Formel ableiten, ohne den Code auszuführen.
````
