---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# Übungen

````{admonition} Übung 2.6 (✩)
:class: tip
Gegeben ist folgender Code:

```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

t = np.linspace(0, 2 * np.pi, 100)
y = np.sin(t)

fig, ax = plt.subplots(figsize=(6, 3))
ax.plot(t, y, color='#4C72B0', linewidth=2, linestyle='dashed', label='sin(t)')
ax.set_xlabel('t in rad')
ax.set_ylabel('y')
ax.set_title('Sinusfunktion')
ax.legend()
ax.grid(True)
plt.show()
```

1. Sagen Sie vorher: Welchen Linienstil hat die Kurve?
2. Was bewirkt `label='sin(t)'` allein, ohne den Aufruf von `ax.legend()`?
3. Was passiert, wenn man `figsize=(6, 3)` durch `figsize=(3, 6)` ersetzt?
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
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

t = np.linspace(0, 2 * np.pi, 100)
y = np.sin(t)

fig, ax = plt.subplots(figsize=(6, 3))
ax.plot(t, y, color='#4C72B0', linewidth=2, linestyle='dashed', label='sin(t)')
ax.set_xlabel('t in rad')
ax.set_ylabel('y')
ax.set_title('Sinusfunktion')
ax.legend()
ax.grid(True)
plt.show()
```

1. Die Kurve ist gestrichelt (`'dashed'`).
2. `label='sin(t)'` speichert die Beschriftung intern, zeigt aber nichts an.
   Erst `ax.legend()` liest alle gespeicherten Labels aus und zeichnet die
   Legende.
3. `figsize=(3, 6)` erzeugt eine schmale, hohe Figure statt einer breiten,
   flachen. Die Kurve wird optisch gestreckt, weil dieselben Datenwerte
   auf einem anderen Seitenverhältnis abgebildet werden.
````

````{admonition} Übung 2.7 (✩)
:class: tip
Gegeben ist folgender Code:

```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

t = np.linspace(0, 1, 500)

fig, ax = plt.subplots(nrows=2, ncols=1, figsize=(8, 6))

ax[0].plot(t, np.sin(2 * np.pi * 5 * t),  color='#4C72B0')
ax[1].plot(t, np.sin(2 * np.pi * 20 * t), color='#DD8452')

plt.tight_layout()
plt.show()
```

1. Was zeigt der obere Subplot, was der untere? Wie viele Schwingungsperioden
   sind jeweils sichtbar?
2. Was passiert, wenn man `plt.tight_layout()` weglässt?
3. Wie würde man den Code ändern, um beide Kurven im selben Subplot
   darzustellen statt in zwei separaten?
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
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

t = np.linspace(0, 1, 500)

fig, ax = plt.subplots(nrows=2, ncols=1, figsize=(8, 6))
ax[0].plot(t, np.sin(2 * np.pi * 5  * t), color='#4C72B0')
ax[1].plot(t, np.sin(2 * np.pi * 20 * t), color='#DD8452')

plt.tight_layout()
plt.show()
```

1. Der obere Subplot zeigt eine Schwingung mit 5 Hz, also 5 vollständige
   Perioden in 1 Sekunde. Der untere zeigt 20 Hz, also 20 Perioden.
2. Ohne `plt.tight_layout()` können sich Achsenbeschriftungen und Titel
   benachbarter Subplots überlappen, weil Matplotlib die Abstände nicht
   automatisch anpasst.
3. Man ersetzt `ax[0].plot(...)` und `ax[1].plot(...)` durch zweimaliges
   `ax.plot(...)` auf demselben Axes-Objekt, das man mit
   `fig, ax = plt.subplots()` (ohne `nrows` und `ncols`) erzeugt.
````

````{admonition} Übung 2.8 (✩✩)
:class: tip
Ein Windmesser zeichnet an einer Wetterstation über 24 Stunden stündlich die
Windgeschwindigkeit auf. Die Messwerte in m/s von 0 bis 23 Uhr lauten:
3.2, 2.8, 2.5, 2.1, 2.4, 3.0, 4.1, 5.3, 6.2, 7.0, 7.8, 8.1,
7.5, 6.9, 6.3, 5.8, 5.1, 4.7, 4.2, 3.8, 3.5, 3.3, 3.1, 2.9.

1. Legen Sie die Daten als NumPy-Arrays an und erstellen Sie einen Linienplot
   der Windgeschwindigkeit über die Tageszeit. Verwenden Sie Kreise als
   Marker.
2. Markieren Sie den Zeitpunkt der maximalen Windgeschwindigkeit mit einem
   auffälligen Marker in einer anderen Farbe. Hinweis: `np.argmax()` liefert
   den Index des größten Werts.
3. Fügen Sie eine horizontale gestrichelte Linie bei der mittleren
   Windgeschwindigkeit ein. Hinweis: `ax.axhline()` zeichnet eine horizontale
   Linie über den gesamten Plotbereich.
4. Beschriften Sie die Achsen mit Einheiten, vergeben Sie einen Titel und
   zeigen Sie die Legende an.

Strukturieren Sie Ihren Code mit EVA-Kommentaren.
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

# Eingabe
stunden    = np.arange(0, 24)
windgeschw = np.array([3.2, 2.8, 2.5, 2.1, 2.4, 3.0, 4.1, 5.3,
                        6.2, 7.0, 7.8, 8.1, 7.5, 6.9, 6.3, 5.8,
                        5.1, 4.7, 4.2, 3.8, 3.5, 3.3, 3.1, 2.9])

# Verarbeitung
idx_max    = np.argmax(windgeschw)
mittelwert = np.mean(windgeschw)

# Ausgabe
fig, ax = plt.subplots(figsize=(10, 5))

ax.plot(stunden, windgeschw, color='#4C72B0', linewidth=1.5,
        marker='o', markersize=4, label='Windgeschwindigkeit')
ax.scatter(stunden[idx_max], windgeschw[idx_max],
           marker='*', s=200, color='#C44E52', zorder=5,
           label=f'Maximum: {windgeschw[idx_max]:.1f} m/s um {stunden[idx_max]}:00 Uhr')
ax.axhline(y=mittelwert, linestyle='dashed', color='gray',
           label=f'Mittelwert: {mittelwert:.1f} m/s')

ax.set_xlabel('Uhrzeit in h')
ax.set_ylabel('Windgeschwindigkeit in m/s')
ax.set_title('Windgeschwindigkeit im Tagesverlauf')
ax.legend()
ax.grid(True)

plt.tight_layout()
plt.show()
```

Das Maximum tritt um 11:00 Uhr mit 8.1 m/s auf. Der typische Tagesgang
des Windes zeigt niedrige Geschwindigkeiten in den frühen Morgenstunden
und einen Anstieg bis zum Mittag, was durch die solare Erwärmung und die
damit verbundene Konvektion erklärt wird.
````

````{admonition} Übung 2.9 (✩✩)
:class: tip
In einem Materialprüflabor werden für fünf Kunststoffproben jeweils zehn
Zugversuche durchgeführt. Die Mittelwerte und Standardabweichungen der
Bruchspannung lauten:

| Probe | Bruchspannung (MPa) | Standardabweichung (MPa) |
|-------|---------------------|--------------------------|
| 1     | 42.3                | 2.1                      |
| 2     | 38.7                | 3.4                      |
| 3     | 51.2                | 1.8                      |
| 4     | 45.8                | 2.7                      |
| 5     | 39.1                | 3.0                      |

1. Legen Sie die Daten als NumPy-Arrays an und stellen Sie die Bruchspannungen
   als Scatter-Plot mit Fehlerbalken dar. Schränken Sie die y-Achse auf den
   Bereich 30 bis 60 MPa ein.
2. Fügen Sie eine horizontale gestrichelte Linie bei der mittleren
   Bruchspannung aller Proben ein.
3. Simulieren Sie für jede Probe 10 Einzelmesswerte mit `np.random.normal()`
   und dem jeweiligen Mittelwert und der jeweiligen Standardabweichung.
   Verwenden Sie `np.random.seed(3)`. Fassen Sie alle 50 Werte in einem
   Array zusammen und stellen Sie ihre Verteilung in einem zweiten Subplot
   nebeneinander als Histogramm dar.

Strukturieren Sie Ihren Code mit EVA-Kommentaren.
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

# Eingabe
proben    = np.array([1, 2, 3, 4, 5])
bruchspg  = np.array([42.3, 38.7, 51.2, 45.8, 39.1])
std_bruch = np.array([ 2.1,  3.4,  1.8,  2.7,  3.0])

np.random.seed(3)
alle_werte = np.concatenate([
    np.random.normal(m, s, 10) for m, s in zip(bruchspg, std_bruch)
])

# Verarbeitung
mittelwert = np.mean(bruchspg)

# Ausgabe
fig, ax = plt.subplots(nrows=1, ncols=2, figsize=(12, 5))

ax[0].errorbar(proben, bruchspg, yerr=std_bruch,
               fmt='s', color='#4C72B0', capsize=6,
               linewidth=1.5, label='Bruchspannung ± Std')
ax[0].axhline(y=mittelwert, linestyle='dashed', color='gray',
              label=f'Mittelwert: {mittelwert:.1f} MPa')
ax[0].set_ylim(30, 60)
ax[0].set_xlabel('Probennummer')
ax[0].set_ylabel('Bruchspannung in MPa')
ax[0].set_title('Bruchspannungen der Kunststoffproben')
ax[0].legend()
ax[0].grid(True)

ax[1].hist(alle_werte, bins=15, color='#4C72B0', edgecolor='white')
ax[1].set_xlabel('Bruchspannung in MPa')
ax[1].set_ylabel('Haeufigkeit')
ax[1].set_title('Verteilung aller Messwerte (N=50)')
ax[1].grid(True, axis='y')

plt.tight_layout()
plt.show()
```

Probe 3 zeigt mit 51.2 MPa die höchste Bruchspannung und gleichzeitig die
kleinste Streuung, was auf ein hochwertiges und gleichmäßiges Material
hindeutet. Das Histogramm zeigt eine breite Verteilung der Einzelwerte, weil
es Messungen aus fünf Proben mit unterschiedlichen Mittelwerten enthält.
````

````{admonition} Übung 2.10 (✩✩✩) Mini-Projekt: Heizenergie und Außentemperatur
:class: tip
Ein Haushalt zeichnet über ein Jahr monatlich die mittlere Außentemperatur
und den Gasverbrauch zum Heizen auf. Die Daten lauten:

| Monat | Temperatur (°C) | Gasverbrauch (kWh) |
|-------|-----------------|---------------------|
| Jan   | −2              | 350                 |
| Feb   | 1               | 290                 |
| Mär   | 5               | 210                 |
| Apr   | 10              | 115                 |
| Mai   | 15              | 60                  |
| Jun   | 18              | 30                  |
| Jul   | 21              | 15                  |
| Aug   | 20              | 20                  |
| Sep   | 14              | 75                  |
| Okt   | 9               | 130                 |
| Nov   | 3               | 230                 |
| Dez   | −1              | 320                 |

**Teil 1: Lineare Regression**

Legen Sie die Daten als NumPy-Arrays an. Berechnen Sie Steigung $m$ und
Achsenabschnitt $b$ der Regressionsgerade mit den folgenden Formeln, die
die Mittelwerte $\bar{x}$ und $\bar{y}$ verwenden:

$$\bar{x} = \frac{1}{N}\sum_{i=1}^{N} x_i, \qquad
\bar{y} = \frac{1}{N}\sum_{i=1}^{N} y_i$$

$$m = \frac{\displaystyle\sum_{i=1}^{N}(x_i - \bar{x})(y_i - \bar{y})}
           {\displaystyle\sum_{i=1}^{N}(x_i - \bar{x})^2},
\qquad
b = \bar{y} - m\,\bar{x}$$

Geben Sie $m$ und $b$ mit Einheiten aus.

**Teil 2: Visualisierung**

Erstellen Sie eine Figure mit drei Subplots: links ein großer Subplot über
die gesamte Höhe, rechts zwei Subplots untereinander. Hinweis: Mit
`matplotlib.gridspec.GridSpec(2, 2)` und `fig.add_subplot(gs[:, 0])` lässt
sich ein Subplot über beide Zeilen spannen.

- Links (volle Höhe): Scatter-Plot mit Temperatur auf der x-Achse und
  Gasverbrauch auf der y-Achse, überlagert mit der Regressionsgerade.
  Markieren Sie den Vorhersagepunkt aus Teil 3 auffällig.
- Rechts oben: monatlicher Temperaturverlauf als Linienplot mit Markern.
- Rechts unten: monatlicher Gasverbrauch als Linienplot mit Markern.
  Verwenden Sie für beide rechten Subplots die Monatsnamen auf der x-Achse.

**Teil 3: Vorhersage**

Im kommenden Januar wird eine mittlere Außentemperatur von −5 °C erwartet.
Berechnen Sie den vorhergesagten Gasverbrauch mit der Regressionsgerade.

**Teil 4: Reflexion**

Bewerten Sie kurz: Passt ein lineares Modell hier gut? Welche physikalische
Erklärung hat das Vorzeichen der Steigung?

Strukturieren Sie Ihren Code mit EVA-Kommentaren.
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

# Eingabe
temperatur   = np.array([-2.,  1.,  5., 10., 15., 18., 21., 20., 14.,  9.,  3., -1.])
gasverbrauch = np.array([350, 290, 210, 115,  60,  30,  15,  20,  75, 130, 230, 320],
                         dtype=float)
monate = ['Jan', 'Feb', 'Mär', 'Apr', 'Mai', 'Jun',
          'Jul', 'Aug', 'Sep', 'Okt', 'Nov', 'Dez']

# Verarbeitung: Lineare Regression
x_mean = np.mean(temperatur)
y_mean = np.mean(gasverbrauch)

m = np.sum((temperatur - x_mean) * (gasverbrauch - y_mean)) / \
    np.sum((temperatur - x_mean)**2)
b = y_mean - m * x_mean

x_linie    = np.linspace(-6, 23, 200)
y_linie    = m * x_linie + b
vorhersage = m * (-5) + b

print(f"Steigung:        {m:.2f} kWh/°C")
print(f"Achsenabschnitt: {b:.1f} kWh")
print(f"Vorhersage fuer -5 °C: {vorhersage:.0f} kWh")

# Ausgabe
from matplotlib.gridspec import GridSpec

fig = plt.figure(figsize=(14, 10))
gs  = GridSpec(2, 2, figure=fig)

ax_scatter = fig.add_subplot(gs[:, 0])   # links, volle Höhe
ax_temp    = fig.add_subplot(gs[0, 1])   # rechts oben
ax_gas     = fig.add_subplot(gs[1, 1])   # rechts unten

# Links: Scatter + Regression
ax_scatter.scatter(temperatur, gasverbrauch,
                   color='#4C72B0', s=70, zorder=5, label='Monatsdaten')
ax_scatter.plot(x_linie, y_linie, color='#DD8452', linewidth=1.5,
                label=f'Regression: y = {m:.1f}x + {b:.0f}')
ax_scatter.scatter(-5, vorhersage, marker='*', s=250, color='#C44E52',
                   zorder=6, label=f'Vorhersage −5 °C: {vorhersage:.0f} kWh')
ax_scatter.set_xlabel('Außentemperatur in °C')
ax_scatter.set_ylabel('Gasverbrauch in kWh')
ax_scatter.set_title('Regression: Temperatur vs. Gasverbrauch')
ax_scatter.legend(fontsize=9)
ax_scatter.grid(True)

# Rechts oben: monatliche Temperatur
ax_temp.plot(np.arange(12), temperatur, color='#4C72B0',
             marker='o', markersize=5, linewidth=1.5)
ax_temp.set_ylabel('Außentemperatur in °C')
ax_temp.set_title('Monatliche Außentemperatur')
ax_temp.set_xticks(np.arange(12))
ax_temp.set_xticklabels(monate, rotation=45)
ax_temp.grid(True)

# Rechts unten: monatlicher Gasverbrauch
ax_gas.plot(np.arange(12), gasverbrauch, color='#DD8452',
            marker='o', markersize=5, linewidth=1.5)
ax_gas.set_ylabel('Gasverbrauch in kWh')
ax_gas.set_title('Monatlicher Gasverbrauch')
ax_gas.set_xticks(np.arange(12))
ax_gas.set_xticklabels(monate, rotation=45)
ax_gas.grid(True)

plt.tight_layout()
plt.show()
```

Ausgabe:
```
Steigung:        -14.59 kWh/°C
Achsenabschnitt: 291.2 kWh
Vorhersage fuer -5 °C: 364 kWh
```

**Teil 4:** Das lineare Modell beschreibt den Zusammenhang im beobachteten
Bereich gut: Kälter bedeutet mehr Heizenergie. Die negative Steigung
(−14.59 kWh/°C) ist physikalisch plausibel, weil der Temperaturunterschied
zwischen Innen und Außen den Wärmeverlust des Gebäudes bestimmt. Für sehr
hohe Temperaturen würde das Modell negative Gasverbräuche vorhersagen, was
physikalisch unsinnig ist. Im beobachteten Bereich von −2 °C bis +21 °C ist
die Näherung jedoch gut geeignet.
````
