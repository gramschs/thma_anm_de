---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 2.7 Scatter-Plots, Histogramme und Fehlerbalken

In den letzten beiden Kapiteln haben wir Zeitverläufe als Linienplots
dargestellt. Nicht alle Daten lassen sich aber sinnvoll als Linie verbinden.
Wenn wir zum Beispiel einzelne Messwerte aus verschiedenen Versuchen
vergleichen, interessiert uns die Verteilung der Punkte, nicht ihre
Reihenfolge. Und wenn wir die statistische Verteilung unseres Messrauschens
aus Kapitel 2.3 untersuchen wollen, brauchen wir ein ganz anderes Diagramm:
das **Histogramm**. In diesem Kapitel lernen wir drei weitere Diagrammtypen
kennen, die in der Ingenieurpraxis unverzichtbar sind.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können mit `ax.scatter()` einen Scatter-Plot erstellen und Punkte
  nach Farbe und Größe unterscheiden.
* [ ] Sie können mit `ax.hist()` ein Histogramm erstellen und die Anzahl der
  Klassen mit `bins` festlegen.
* [ ] Sie können mit `ax.errorbar()` Messwerte mit Fehlerbalken darstellen.
* [ ] Sie können Scatter-Plot und Linienplot in einem gemeinsamen Diagramm
  kombinieren.
```

## Scatter-Plots

Ein **Scatter-Plot** stellt einzelne Datenpunkte als Punkte dar, ohne sie
zu verbinden. Das ist sinnvoll, wenn kein kontinuierlicher Zusammenhang
zwischen den Punkten besteht.

Wir stellen uns vor, dass wir an unserer Maschine 30 Einzelmessungen der
maximalen Beschleunigung unter verschiedenen Betriebsbedingungen aufgenommen
haben. Jede Messung liefert eine Betriebstemperatur und eine
Spitzenbeschleunigung:

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

np.random.seed(7)

# Simulierte Messdaten: Temperatur und Spitzenbeschleunigung
temperatur     = np.random.uniform(20, 80, 30)  # in °C
beschleunigung = 0.05 * temperatur + np.random.normal(0, 0.5, 30)  # in m/s²

# Visualisierung
fig, ax = plt.subplots(figsize=(7, 5))

ax.scatter(temperatur, beschleunigung, color='#4C72B0', marker='o')

ax.set_xlabel('Temperatur in °C')
ax.set_ylabel('Spitzenbeschleunigung in m/s²')
ax.set_title('Beschleunigung vs. Temperatur')
ax.grid(True)

plt.show()
```

Der Scatter-Plot zeigt sofort, ob ein Zusammenhang zwischen Temperatur und
Beschleunigung besteht. Wir sehen einen leichten Aufwärtstrend: Bei höheren
Temperaturen tendiert die Beschleunigung zu größeren Werten.

Wir können die Punkte zusätzlich nach einer dritten Größe einfärben, zum
Beispiel nach der Drehzahl der Maschine zum Messzeitpunkt. Dazu übergeben wir
dem `c`-Argument ein Array und wählen eine Farbskala mit `cmap`:

```{code-cell} python
drehzahl = np.random.uniform(500, 3000, 30)   # in U/min

# Visualisierung
fig, ax = plt.subplots(figsize=(7, 5))

scatter = ax.scatter(temperatur, beschleunigung,
                     c=drehzahl, cmap='plasma',
                     s=60, marker='o')

fig.colorbar(scatter, ax=ax, label='Drehzahl in U/min')

ax.set_xlabel('Temperatur in °C')
ax.set_ylabel('Spitzenbeschleunigung in m/s²')
ax.set_title('Beschleunigung vs. Temperatur (eingefärbt nach Drehzahl)')
ax.grid(True)

plt.show()
```

`s=60` legt die Punktgröße in Quadratpunkten fest. `fig.colorbar()` fügt
eine Farbskala als Legende für die dritte Dimension hinzu. So lassen sich
drei Variablen gleichzeitig in einem einzigen Diagramm darstellen.

````{admonition} Mini-Übung
:class: tip
Wir untersuchen den Zusammenhang zwischen Schwingungsamplitude und
Dämpfungskoeffizient $\delta$. Berechnen Sie für 20 verschiedene
Dämpfungskoeffizienten die maximale Beschleunigung des Schwingungssignals:

```python
# Parameter für die Schwingungen
delta_werte = np.linspace(0.2, 4.0, 20)
f           = 10.0
t           = np.linspace(0, 2, 1000)

# Erzeugung der Signale und Berechnung der maximalen Beschleunigung
max_beschleunigung = np.zeros(20)
for i, d in enumerate(delta_werte):
    signal = 5.0 * np.exp(-d * t) * np.sin(2 * np.pi * f * t)
    max_beschleunigung[i] = np.max(signal)
```

1. Erstellen Sie einen Scatter-Plot mit `delta_werte` auf der x-Achse und
   `max_beschleunigung` auf der y-Achse.
2. Verwenden Sie Dreiecke als Marker (`marker='^'`).
3. Beschriften Sie die Achsen mit Einheiten und fügen Sie einen Titel hinzu.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

# Parameter für die Schwingungen
delta_werte = np.linspace(0.2, 4.0, 20)
f           = 10.0
t           = np.linspace(0, 2, 1000)

# Erzeugung der Signale und Berechnung der maximalen Beschleunigung
max_beschleunigung = np.zeros(20)
for i, d in enumerate(delta_werte):
    signal = 5.0 * np.exp(-d * t) * np.sin(2 * np.pi * f * t)
    max_beschleunigung[i] = np.max(signal)

# Visualisierung
fig, ax = plt.subplots(figsize=(7, 5))
ax.scatter(delta_werte, max_beschleunigung, color='#DD8452', marker='^', s=60)
ax.set_xlabel('Dämpfungskoeffizient in 1/s')
ax.set_ylabel('Maximale Beschleunigung in m/s²')
ax.set_title('Einfluss der Dämpfung auf die Spitzenbeschleunigung')
ax.grid(True)
plt.show()
```

Der Scatter-Plot zeigt deutlich, dass die maximale Beschleunigung mit
zunehmendem Dämpfungskoeffizient monoton abnimmt. Bei starker Dämpfung
(δ > 3) ist die Spitzenamplitude bereits auf unter 4.7 m/s² reduziert.
````

## Histogramme

Ein **Histogramm** zeigt, wie häufig Werte in bestimmten Intervallen
(Klassen) auftreten. Das ist das richtige Werkzeug, um die Verteilung
von Messwerten oder Rauschen zu untersuchen.

Wir analysieren das Messrauschen unseres Sensors. In Kapitel 2.3 haben wir
es als normalverteiltes Rauschen mit Standardabweichung 0.5 modelliert.
Das Histogramm zeigt, ob die Rauschverteilung tatsächlich dieser Annahme
entspricht:

```{code-cell} python
np.random.seed(42)
rauschen = np.random.normal(0.0, 0.5, 1000)

fig, ax = plt.subplots(figsize=(7, 5))

ax.hist(rauschen, bins=30, color='#4C72B0', edgecolor='white')

ax.set_xlabel('Rauschwert in m/s²')
ax.set_ylabel('Häufigkeit')
ax.set_title('Verteilung des Messrauschens (N=1000)')
ax.grid(True, axis='y')

plt.show()
```

`bins=30` legt die Anzahl der Klassen fest. Mehr Klassen geben ein
feineres Bild der Verteilung, aber zu viele Klassen machen das Histogramm
unruhig. Als Faustregel gilt: Für N Messwerte sind $\sqrt{N}$ bis $2\sqrt{N}$
Klassen ein guter Ausgangspunkt. `edgecolor='white'` zeichnet weiße Ränder
zwischen den Balken, was die Lesbarkeit verbessert.

Wir können dem Histogramm eine theoretische Normalverteilung überlagern, um
die Übereinstimmung zu prüfen. Dazu kombinieren wir Histogramm und Linienplot:

```{code-cell} python
fig, ax = plt.subplots(figsize=(7, 5))

ax.hist(rauschen, bins=30, color='#4C72B0', edgecolor='white',
        density=True, label='Messdaten')

# Theoretische Normalverteilung mit NumPy berechnen
sigma   = 0.5
x_kurve = np.linspace(-2, 2, 200)
y_kurve = (1 / (sigma * np.sqrt(2 * np.pi))) * np.exp(-0.5 * (x_kurve / sigma)**2)
ax.plot(x_kurve, y_kurve,
        color='red', linewidth=2, label='Normalverteilung (σ=0.5)')

ax.set_xlabel('Rauschwert in m/s²')
ax.set_ylabel('Wahrscheinlichkeitsdichte')
ax.set_title('Messrauschen vs. Normalverteilung')
ax.legend()
ax.grid(True, axis='y')

plt.show()
```

`density=True` normiert das Histogramm zur Wahrscheinlichkeitsdichte, sodass
die Gesamtfläche unter allen Balken gleich 1 ist. Nur dann lässt sich das
Histogramm mit der theoretischen Dichtefunktion vergleichen. Die
Dichtefunktion der Normalverteilung lautet:

$$f(x) = \frac{1}{\sigma\sqrt{2\pi}} \cdot e^{-\frac{1}{2}\left(\frac{x}{\sigma}\right)^2}$$

Wir berechnen sie direkt mit `np.exp()` als Vektoroperation über alle
Punkte von `x_kurve`. Die gute Übereinstimmung bestätigt unsere
Rauschmodellierung aus Kapitel 2.3.

````{admonition} Mini-Übung
:class: tip
Vergleichen Sie in einem gemeinsamen Diagramm mit zwei Subplots nebeneinander
die Verteilungen von zwei verschiedenen Rauschquellen:

- Links: normalverteiltes Rauschen `np.random.normal(0, 0.5, 2000)`
- Rechts: gleichverteiltes Rauschen `np.random.uniform(-1, 1, 2000)`

Verwenden Sie jeweils `bins=40` und `density=True`. Beschriften Sie die
Achsen, vergeben Sie Titel und fügen Sie ein Gitter hinzu.

Verwenden Sie `np.random.seed(0)`.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

# Erzeugung der verrauschten Signale
np.random.seed(0)
rauschen_normal = np.random.normal(0, 0.5, 2000)
rauschen_gleich = np.random.uniform(-1, 1, 2000)

# Visualisierung
fig, ax = plt.subplots(nrows=1, ncols=2, figsize=(12, 5))

# linkes Histogramm mit normalverteiltem Rauschen
ax[0].hist(rauschen_normal, bins=40, color='#4C72B0',
           edgecolor='white', density=True)
ax[0].set_xlabel('Rauschwert in m/s²')
ax[0].set_ylabel('Wahrscheinlichkeitsdichte')
ax[0].set_title('Normalverteiltes Rauschen')
ax[0].grid(True)

# rechtes Histogramm mit gleichverteiltem Rauschen
ax[1].hist(rauschen_gleich, bins=40, color='#55A868',
           edgecolor='white', density=True)
ax[1].set_xlabel('Rauschwert in m/s²')
ax[1].set_ylabel('Wahrscheinlichkeitsdichte')
ax[1].set_title('Gleichverteiltes Rauschen')
ax[1].grid(True)

plt.tight_layout()
plt.show()
```

Das normalverteilte Rauschen zeigt die typische Glockenform mit dem Maximum
bei null. Das gleichverteilte Rauschen ergibt näherungsweise ein flaches
Rechteck zwischen -1 und 1. In der Messtechnik ist normalverteiltes Rauschen
häufiger anzutreffen, weil es durch das Zusammenwirken vieler unabhängiger
Störquellen entsteht.
````

## Fehlerbalken mit `ax.errorbar()`

In der Messtechnik ist jeder Messwert mit einer Unsicherheit behaftet. Diese
Unsicherheit lässt sich mit **Fehlerbalken** darstellen. `ax.errorbar()` ist
die Matplotlib-Funktion dafür.

Wir kehren zu unserem Beschleunigungssensor zurück. Statt 30 Einzelmessungen
haben wir jetzt 8 Messpunkte, wobei jeder Punkt der Mittelwert aus mehreren
Wiederholungsmessungen ist und die Fehlerbalken die Standardabweichung zeigen:

```{code-cell} python
# Mittelwerte und Standardabweichungen aus Wiederholungsmessungen
temp_punkte   = np.array([25, 30, 35, 40, 45, 50, 60, 70])    # in °C
beschl_mittel = np.array([1.2, 1.4, 1.5, 1.8, 2.0, 2.1, 2.5, 2.9])  # in m/s²
beschl_std    = np.array([0.15, 0.12, 0.18, 0.20, 0.14, 0.22, 0.19, 0.25])

fig, ax = plt.subplots(figsize=(7, 5))

ax.errorbar(temp_punkte, beschl_mittel,
            yerr=beschl_std,
            fmt='o', color='#4C72B0',
            capsize=5, linewidth=1.5,
            label='Messwerte ± Standardabweichung')

ax.set_xlabel('Temperatur in °C')
ax.set_ylabel('Beschleunigung in m/s²')
ax.set_title('Kalibrierungsmessung mit Unsicherheiten')
ax.legend()
ax.grid(True)

plt.show()
```

`yerr` übergibt die Fehler in y-Richtung als Array. `fmt='o'` legt das
Markersymbol fest, `capsize=5` gibt den Querbalken am Ende der Fehlerbalken
eine Breite von 5 Punkten. Ohne `capsize` sind die Fehlerbalken schwerer
abzulesen.

Wir können Fehlerbalken auch mit einer Regressionsgerade kombinieren:

```{code-cell} python
# Lineare Regression
N      = len(temp_punkte)
x      = temp_punkte.astype(float)
y      = beschl_mittel

m = (N * np.sum(x * y) - np.sum(x) * np.sum(y)) / \
    (N * np.sum(x**2) - np.sum(x)**2)
b = (np.sum(x**2) * np.sum(y) - np.sum(x) * np.sum(x * y)) / \
    (N * np.sum(x**2) - np.sum(x)**2)

x_linie   = np.linspace(20, 75, 200)
y_linie   = m * x_linie + b

fig, ax = plt.subplots(figsize=(7, 5))

ax.errorbar(temp_punkte, beschl_mittel, yerr=beschl_std,
            fmt='o', color='#4C72B0', capsize=5,
            label='Messwerte ± Std')
ax.plot(x_linie, y_linie, color='#DD8452', linewidth=1.5,
        label=f'Regression: y = {m:.3f}x + {b:.3f}')

ax.set_xlabel('Temperatur in °C')
ax.set_ylabel('Beschleunigung in m/s²')
ax.set_title('Kalibrierungsmessung mit Regressionsgerade')
ax.legend()
ax.grid(True)

plt.show()
```

Die Kombination aus `errorbar` und Linienplot in einem gemeinsamen Diagramm
ist ein sehr häufiges Muster in der Ingenieurpraxis: Messwerte als Punkte,
Modell oder Regression als Linie.

## Zusammenfassung und Ausblick

Wir haben drei weitere Diagrammtypen kennengelernt. Der Scatter-Plot zeigt
Einzelpunkte ohne Verbindungslinien und eignet sich für den Vergleich von
Messwerten unter verschiedenen Bedingungen. Das Histogramm stellt die
Häufigkeitsverteilung von Messwerten dar und lässt sich mit `density=True`
zur Wahrscheinlichkeitsdichte normieren. Fehlerbalken mit `ax.errorbar()`
machen Messunsicherheiten sichtbar und sind in jedem quantitativen Bericht
unverzichtbar.

Im Übungskapitel wenden wir alle drei Diagrammtypen aus diesem Kapitel
sowie die Linienplots und Subplots aus den vorigen Kapiteln auf neue
Aufgabenstellungen an.
