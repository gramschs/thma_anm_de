---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 2.1 NumPy-Grundlagen

In der Messtechnik fallen schnell tausende von Messwerten an. Ein
Beschleunigungssensor, der eine vibrierende Maschine überwacht, liefert
beispielsweise 10.000 Messwerte pro Sekunde. Wollen wir diese Daten mit
Python-Listen verarbeiten, brauchen wir Schleifen über tausende von Elementen:
mühsam zu schreiben und langsam in der Ausführung. In diesem Kapitel lernen wir
NumPy kennen, eine Bibliothek, die genau für solche Aufgaben gebaut wurde. Ihr
zentraler Datentyp, das **Array**, erlaubt es, mathematische Operationen und
statistische Kenngrößen direkt auf ganze Zahlenreihen anzuwenden, ohne eine
einzige Schleife zu schreiben.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie wissen, was ein **NumPy-Array** ist und wie es sich von einer
  Python-Liste unterscheidet.
* [ ] Sie können ein Array mit `np.array()`, `np.linspace()` und `np.zeros()`
  erzeugen.
* [ ] Sie können **Vektoroperationen** (elementweise Addition, Multiplikation,
  Skalierung) auf Arrays anwenden.
* [ ] Sie können mathematische Funktionen wie `np.sin()` und `np.exp()` auf
  Arrays anwenden.
* [ ] Sie können mit `np.mean()`, `np.std()`, `np.min()` und `np.max()`
  statistische Kenngrößen eines Arrays berechnen.
```

## Was ist ein NumPy-Array?

NumPy (kurz für *Numerical Python*) ist eine Bibliothek, also eine Sammlung
fertiger Funktionen, die wir in eigenem Code nutzen können, ohne sie selbst
zu schreiben. Bevor wir eine Bibliothek verwenden können, müssen wir sie mit
`import` laden. Für NumPy hat sich eine feste Abkürzung etabliert, unter der
wir die Bibliothek im restlichen Code ansprechen:

```{code-cell} python
import numpy as np
```

Diese Zeile lädt die Bibliothek `numpy` und macht sie im Code über den Namen
`np` verfügbar. Ab jetzt rufen wir alle Funktionen aus NumPy mit diesem
Kürzel auf, zum Beispiel `np.array()`. Die Abkürzung `np` ist reine
Konvention, ein anderer Name würde technisch genauso funktionieren, aber
`np` ist in der Python-Welt so verbreitet, dass praktisch jeder NumPy-Code
sie verwendet.

Der zentrale Datentyp von NumPy ist das **Array**. Es erlaubt uns,
mathematische Operationen direkt auf ganze Zahlenreihen anzuwenden, ohne
eine einzige Schleife zu schreiben, wie wir im Folgenden sehen.

### Array versus Liste

Den Unterschied zwischen Liste und Array sehen wir am schnellsten an einem
Beispiel. Ein Sensor liefert fünf Beschleunigungswerte in m/s²:

```{code-cell} python
# Als Python-Liste
messwerte_liste = [0.3, 1.2, 2.5, 1.8, 0.7]

# Als NumPy-Array
messwerte_array = np.array([0.3, 1.2, 2.5, 1.8, 0.7])

print(messwerte_liste)
print(messwerte_array)
print(type(messwerte_liste))
print(type(messwerte_array))
```

Auf den ersten Blick sehen beide Ausgaben ähnlich aus, aber `type()` zeigt den
grundlegenden Unterschied: Eine Liste ist ein allgemeiner Container, der
beliebige Objekte in beliebiger Mischung aufnehmen kann. Ein Array ist speziell
für numerische Daten gebaut und besitzt grundsätzlich einen gemeinsamen
Datentyp. Beim Erzeugen wandelt NumPy die Eingabewerte ggf. in einen passenden
gemeinsamen Datentyp um. Der gemeinsame Datentyp macht Rechenoperationen auf
Arrays so schnell: NumPy kann intern effizienten, kompilierten Code verwenden,
statt jedes Element einzeln in Python zu verarbeiten. Wie sich das bei
Rechenoperationen konkret auswirkt, sehen wir im nächsten Abschnitt.

### Ein Array untersuchen: `.shape` und `.dtype`

Zwei Attribute helfen uns, ein Array auf einen Blick zu verstehen:

```{code-cell} python
print(messwerte_array.shape)   # Anzahl der Elemente je Dimension
print(messwerte_array.dtype)   # Datentyp der gespeicherten Werte
```

`.shape` gibt die Abmessungen des Arrays als Tupel zurück. `(5,)` bedeutet: eine
Dimension mit fünf Elementen. `.dtype` gibt den gemeinsamen Datentyp aller
Elemente zurück, hier `float64` für Fließkommazahlen. Diese beiden Attribute
sind der schnellste Weg, ein unbekanntes Array zu prüfen.

### Arrays erzeugen

Neben `np.array()`, das eine bestehende Liste in ein Array umwandelt, stellt
NumPy zwei weitere Funktionen bereit, mit denen wir Arrays direkt erzeugen, ohne
die Werte einzeln aufzuschreiben.

`np.linspace(start, stop, anzahl)` erzeugt `anzahl` gleichmäßig verteilte Werte
zwischen `start` und `stop`. Das eignet sich beispielsweise für Zeitachsen:

```{code-cell} python
t = np.linspace(0, 2, 5)   # 5 Werte zwischen 0 und 2 Sekunden
print(t)
```

`np.zeros(anzahl)` erzeugt ein Array aus lauter Nullen. Das ist nützlich, um
ein Array als Platzhalter anzulegen, das später mit Werten gefüllt wird:

```{code-cell} python
platzhalter = np.zeros(5)
print(platzhalter)
```

Mit diesen drei Funktionen, `np.array()`, `np.linspace()` und `np.zeros()`,
decken wir bereits die meisten Fälle ab, in denen wir ein Array neu anlegen
müssen: aus vorhandenen Werten, als gleichmäßig verteilte Achse oder als
leerer Platzhalter.

## Vektoroperationen und mathematische Funktionen

Im letzten Abschnitt haben wir Arrays erzeugt und uns ihre Struktur mit
`.shape` und `.dtype` angesehen. Jetzt sehen wir, was Arrays wirklich
nützlich macht: Rechenoperationen, die auf ganze Zahlenreihen wirken, ohne
eine einzige Schleife zu schreiben.

### Elementweise Operationen

Angenommen, wir wollen aus den Beschleunigungswerten die wirkende Kraft
berechnen. Es gilt $F = m \cdot a$, wobei die Masse $m = 5\,\mathrm{kg}$
beträgt. Mit einer Python-Liste brauchen wir dafür eine Schleife:

```{code-cell} python
# Mit der Liste: manuelle Schleife notwendig
kraefte_liste = []
for a in messwerte_liste:
    kraefte_liste.append(5.0 * a)

print(kraefte_liste)
```

Mit dem Array genügt eine einzige Zeile:

```{code-cell} python
kraefte_array = 5.0 * messwerte_array
print(kraefte_array)
```

Die Multiplikation wird auf **jedes Element einzeln** angewendet, ohne dass
wir das explizit programmieren müssen. Das nennt man eine
**Vektoroperation**. Dasselbe Prinzip gilt für alle Grundrechenarten.
Addieren wir zwei Arrays, werden die Elemente paarweise addiert:

```{code-cell} python
sensor_a = np.array([0.3, 1.2, 2.5, 1.8, 0.7])
sensor_b = np.array([0.1, 0.2, 0.3, 0.1, 0.2])

summe = sensor_a + sensor_b
print(summe)
```

Damit eine elementweise Operation funktioniert, müssen die beteiligten Arrays
dieselbe Anzahl an Elementen haben. `sensor_a` und `sensor_b` haben beide fünf
Elemente, deshalb lässt sich jedes Element eindeutig einem Partner zuordnen.
Besteht das Array dahingehend nur aus einem Skalar, dann wird der Skalar einzeln
auf jedes Element angewendet wie bei der Multiplikation `5.0 * messwerte_array`.

Mehrere Operationen lassen sich in einer Zeile kombinieren, genau wie bei
gewöhnlichen Zahlen:

```{code-cell} python
kalibriert = (sensor_a + sensor_b) * 2.0 - 0.1
print(kalibriert)
```

### Mathematische Funktionen auf Arrays

Neben den Grundrechenarten stellt NumPy auch mathematische Funktionen
bereit, die elementweise auf Arrays wirken. Zwei davon brauchen wir häufig:
`np.sin()` für trigonometrische Berechnungen und `np.exp()` für
Exponentialfunktionen.

```{code-cell} python
winkel = np.linspace(0, 2 * np.pi, 5)
print(np.sin(winkel))
```

`np.sin()` wendet den Sinus auf jedes Element von `winkel` einzeln an und
gibt ein neues Array derselben Länge zurück. `np.exp()` funktioniert nach
demselben Prinzip:

```{code-cell} python
werte = np.array([0.0, 1.0, 2.0, 3.0])
print(np.exp(werte))
```

Python bringt mit dem Modul `math` bereits `math.sin()` und `math.exp()` mit,
diese akzeptieren aber nur einzelne Zahlen, keine Arrays. Um sie auf mehrere
Werte anzuwenden, bräuchten wir wieder eine Schleife. Die NumPy-Varianten
`np.sin()` und `np.exp()` sind für Arrays gebaut und damit in unserem Kontext
die richtige Wahl.

## Statistische Kenngrößen

Bisher haben wir einzelne Werte eines Arrays betrachtet oder das ganze Array
auf einmal transformiert. Oft interessiert uns aber nicht jeder einzelne
Wert, sondern eine zusammenfassende Kennzahl: Wie groß ist ein Messwert im
Mittel? Wie stark schwanken die Werte? NumPy stellt dafür Funktionen
bereit, die aus einem Array eine einzelne Zahl berechnen.

Als Datengrundlage nehmen wir eine Messreihe: die Spitzenbeschleunigung, die
ein Sensor bei zwölf aufeinanderfolgenden Testläufen derselben Maschine
aufgezeichnet hat.

```{code-cell} python
spitzenwerte = np.array([4.8, 5.1, 4.6, 5.3, 4.9, 5.0,
                          4.7, 5.2, 4.9, 5.4, 4.8, 5.0])
print(spitzenwerte)
```

### Mittelwert, Minimum und Maximum

```{code-cell} python
print(f"Mittelwert: {np.mean(spitzenwerte):.2f} m/s^2")
print(f"Minimum:    {np.min(spitzenwerte):.2f} m/s^2")
print(f"Maximum:    {np.max(spitzenwerte):.2f} m/s^2")
```

`np.mean()` addiert alle Werte und teilt durch die Anzahl der Elemente,
genau wie eine Mittelwertberechnung von Hand, nur ohne Schleife. `np.min()`
und `np.max()` liefern den kleinsten beziehungsweise größten Wert im Array.

### Standardabweichung

```{code-cell} python
print(f"Standardabweichung: {np.std(spitzenwerte):.3f} m/s^2")
```

Die Standardabweichung beschreibt, wie stark die einzelnen Werte im Mittel
vom Mittelwert abweichen. Eine kleine Standardabweichung bedeutet, dass die
Testläufe sehr ähnliche Spitzenwerte lieferten. Eine große Standardabweichung
zeigt, dass die Maschine von Lauf zu Lauf deutlich unterschiedlich reagiert.

`np.mean()`, `np.std()`, `np.min()` und `np.max()` lassen sich auch direkt
als Methode des Arrays aufrufen: `spitzenwerte.mean()` liefert dasselbe
Ergebnis wie `np.mean(spitzenwerte)`. Beide Schreibweisen sind gebräuchlich.
In diesem Skript verwenden wir durchgehend die Funktionsschreibweise
`np.funktion(array)`, weil sie unabhängig davon funktioniert, ob wir mit
einem Array oder einer gewöhnlichen Liste arbeiten.

Mit diesen vier Funktionen lässt sich jede Messreihe auf einen Blick
charakterisieren: ein typischer Wert durch den Mittelwert, die
Schwankungsbreite durch die Standardabweichung und die Extremwerte durch
Minimum und Maximum. Das sind die ersten Werkzeuge, um aus reinen
Zahlenreihen belastbare Aussagen über ein gemessenes System abzuleiten.

## Zusammenfassung und Ausblick

Wir haben NumPy-Arrays als Alternative zu Python-Listen kennengelernt, mit
`np.array()`, `np.linspace()` und `np.zeros()` erzeugt und mit `.shape` und
`.dtype` untersucht. Vektoroperationen und Funktionen wie `np.sin()` und
`np.exp()` wenden wir direkt auf ganze Arrays an, ohne Schleifen zu
schreiben. Mit `np.mean()`, `np.std()`, `np.min()` und `np.max()` fassen wir
eine Messreihe in wenigen Kennzahlen zusammen.

Im nächsten Kapitel visualisieren wir Daten mit Plotly Express. Zweidimensionale
Arrays und lineare Gleichungssysteme folgen in Kapitel 3.
