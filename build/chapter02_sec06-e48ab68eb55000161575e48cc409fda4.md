---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 2.6 Subplots

Zuletzt haben wir einzelne Diagramme erstellt. In der Ingenieurpraxis
wollen wir aber oft mehrere zusammenhängende Größen gleichzeitig darstellen.
Bei der Analyse unserer Maschine interessiert uns zum Beispiel nicht nur die
Beschleunigung, sondern auch das Frequenzspektrum oder der Vergleich
verschiedener Betriebszustände. Dafür nutzen wir **Subplots**: mehrere
Zeichenbereiche in einer gemeinsamen Figure, die nebeneinander oder
untereinander angeordnet sind.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können mit `plt.subplots(nrows, ncols)` mehrere Subplots in einer
  Figure anlegen.
* [ ] Sie können einzelne Subplots über `ax[i]` (1D) oder `ax[i, j]` (2D)
  ansprechen.
* [ ] Sie können Subplots mit `plt.tight_layout()` automatisch ausrichten.
* [ ] Sie können Achsenbereiche mit `ax.set_xlim()` und `ax.set_ylim()`
  einschränken.
```

## Mehrere Subplots untereinander

Wir starten mit dem einfachsten Fall: zwei Subplots untereinander. Dazu
übergeben wir `plt.subplots()` die Anzahl der Zeilen und Spalten:

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

# Schwingungssignale (wie in Kapitel 2.1)
A      = 5.0
daempf = 1.5
f      = 10.0
t      = np.linspace(0, 2, 1000)
a      = A * np.exp(-daempf * t) * np.sin(2 * np.pi * f * t)
a2     = (A / 2) * np.exp(-daempf * t) * np.sin(2 * np.pi * 2 * f * t)

fig, ax = plt.subplots(nrows=2, ncols=1, figsize=(8, 6))

ax[0].plot(t, a,  color='#4C72B0', linewidth=1.5, label='Grundschwingung')
ax[1].plot(t, a2, color='#DD8452', linewidth=1.5, label='Oberschwingung')

for axes in ax:
    axes.set_xlabel('Zeit in s')
    axes.set_ylabel('Beschleunigung in m/s²')
    axes.legend()
    axes.grid(True)

ax[0].set_title('Grundschwingung (10 Hz)')
ax[1].set_title('Oberschwingung (20 Hz)')

plt.tight_layout()
plt.show()
```

`plt.subplots(nrows=2, ncols=1)` erzeugt zwei Zeichenbereiche untereinander
und gibt sie als Array `ax` zurück. Wir sprechen den oberen mit `ax[0]` und
den unteren mit `ax[1]` an, genau wie bei einem NumPy-Array.

`plt.tight_layout()` passt die Abstände zwischen den Subplots automatisch an,
sodass sich Beschriftungen nicht überlappen. Wir rufen es immer direkt vor
`plt.show()` auf.

```{admonition} for-Schleife über Subplots
:class: note
Da wir für jeden Subplot dieselben Formatierungsbefehle aufrufen
(`set_xlabel`, `set_ylabel`, `legend`, `grid`), lohnt sich eine
`for`-Schleife über das `ax`-Array. Das vermeidet Wiederholungen und macht
den Code kürzer. Schreibt man später mehr Subplots, muss man die
Formatierung nur einmal anpassen.
```

````{admonition} Mini-Übung
:class: tip
Erzeugen Sie zwei Subplots untereinander:

- Oben: das saubere Schwingungssignal `a` in `'#4C72B0'`
- Unten: das verrauschte Signal
  `a_verrauscht = a + np.random.normal(0.0, 0.5, size=len(t))`
  in `'gray'` mit `linewidth=0.8`

Verwenden Sie `np.random.seed(42)`. Beschriften Sie beide Subplots mit
Achsen, Titel und Gitter.
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

A      = 5.0
daempf = 1.5
f      = 10.0
t      = np.linspace(0, 2, 1000)
a      = A * np.exp(-daempf * t) * np.sin(2 * np.pi * f * t)

np.random.seed(42)
a_verrauscht = a + np.random.normal(0.0, 0.5, size=len(t))

fig, ax = plt.subplots(nrows=2, ncols=1, figsize=(8, 6))

ax[0].plot(t, a,            color='#4C72B0', linewidth=1.5, label='Sauber')
ax[1].plot(t, a_verrauscht, color='gray',    linewidth=0.8, label='Verrauscht')

for axes in ax:
    axes.set_xlabel('Zeit in s')
    axes.set_ylabel('Beschleunigung in m/s²')
    axes.legend()
    axes.grid(True)

ax[0].set_title('Sauberes Signal')
ax[1].set_title('Verrauschtes Signal')

plt.tight_layout()
plt.show()
```

Die beiden Subplots zeigen denselben Zeitbereich auf derselben y-Skala,
was den direkten Vergleich erleichtert. Das Rauschen ist im unteren
Subplot deutlich als Zackigkeit der Kurve erkennbar, während die
Grundform der gedämpften Schwingung erhalten bleibt.
````

## Subplots nebeneinander und im Raster

Für mehr als zwei Subplots bietet sich ein Raster an. Mit `nrows=2, ncols=2`
erhalten wir vier Subplots, die wir über `ax[zeile, spalte]` ansprechen:

```{code-cell} python
# Drei weitere Signale mit unterschiedlichen Daempfungen
a_stark   = A * np.exp(-3.0 * t) * np.sin(2 * np.pi * f * t)
a_schwach = A * np.exp(-0.5 * t) * np.sin(2 * np.pi * f * t)
a_summe   = a + a2

signale = [a, a_stark, a_schwach, a_summe]
titel   = ['Normaldaempfung (δ=1.5)',
           'Starke Daempfung (δ=3.0)',
           'Schwache Daempfung (δ=0.5)',
           'Grundschwingung + Oberschwingung']
farben  = ['#4C72B0', '#DD8452', '#55A868', '#8172B2']

fig, ax = plt.subplots(nrows=2, ncols=2, figsize=(12, 8))

for i, (sig, tit, farbe) in enumerate(zip(signale, titel, farben)):
    zeile  = i // 2
    spalte = i  % 2
    ax[zeile, spalte].plot(t, sig, color=farbe, linewidth=1.5)
    ax[zeile, spalte].set_title(tit)
    ax[zeile, spalte].set_xlabel('Zeit in s')
    ax[zeile, spalte].set_ylabel('Beschleunigung in m/s²')
    ax[zeile, spalte].grid(True)

plt.tight_layout()
plt.show()
```

Die Kombination aus `enumerate()` und `zip()` erlaubt es, gleichzeitig über
Signal, Titel und Farbe zu iterieren und den laufenden Index `i` zu nutzen.
Mit `i // 2` berechnen wir die Zeile und mit `i % 2` die Spalte im Raster.

## Achsenbereiche einschränken

Manchmal wollen wir nur einen bestimmten Ausschnitt eines Signals zeigen.
`ax.set_xlim()` und `ax.set_ylim()` legen den sichtbaren Bereich fest:

```{code-cell} python
fig, ax = plt.subplots(nrows=1, ncols=2, figsize=(12, 4))

# Links: vollständiges Signal
ax[0].plot(t, a, color='#4C72B0', linewidth=1.5)
ax[0].set_title('Vollstaendiges Signal')
ax[0].set_xlabel('Zeit in s')
ax[0].set_ylabel('Beschleunigung in m/s²')
ax[0].grid(True)

# Rechts: Ausschnitt der ersten 0.3 Sekunden
ax[1].plot(t, a, color='#4C72B0', linewidth=1.5)
ax[1].set_xlim(0.0, 0.3)
ax[1].set_ylim(-6.0, 6.0)
ax[1].set_title('Ausschnitt: 0 bis 0.3 s')
ax[1].set_xlabel('Zeit in s')
ax[1].set_ylabel('Beschleunigung in m/s²')
ax[1].grid(True)

plt.tight_layout()
plt.show()
```

Im rechten Subplot sehen wir die ersten drei Schwingungsperioden deutlich
vergrößert. Das ist in der Messtechnik häufig nützlich, um den Einschwingvorgang
einer Maschine genauer zu untersuchen.

````{admonition} Mini-Übung
:class: tip
Erstellen Sie eine Figure mit drei Subplots nebeneinander (`nrows=1, ncols=3`),
die drei verschiedene Betriebszustände unserer Maschine zeigen:

- Links: Normalbetrieb (`daempf=1.5`, `A=5.0`)
- Mitte: Überlastbetrieb (`daempf=0.3`, `A=8.0`)
- Rechts: Nachlauf (`daempf=4.0`, `A=3.0`)

Alle drei Signale sollen dieselbe Frequenz `f=10.0` und dieselbe Zeitachse
`t = np.linspace(0, 2, 1000)` verwenden. Schränken Sie die y-Achse aller
drei Subplots auf `(-9, 9)` ein, damit die Amplituden vergleichbar sind.
Vergeben Sie Titel, Achsenbeschriftungen und Gitter.
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

t = np.linspace(0, 2, 1000)
f = 10.0

parameter = [
    (1.5, 5.0, 'Normalbetrieb',    '#4C72B0'),
    (0.3, 8.0, 'Ueberlastbetrieb', '#DD8452'),
    (4.0, 3.0, 'Nachlauf',         '#55A868'),
]

fig, ax = plt.subplots(nrows=1, ncols=3, figsize=(14, 4))

for i, (daempf, amp, tit, farbe) in enumerate(parameter):
    sig = amp * np.exp(-daempf * t) * np.sin(2 * np.pi * f * t)
    ax[i].plot(t, sig, color=farbe, linewidth=1.5)
    ax[i].set_title(tit)
    ax[i].set_xlabel('Zeit in s')
    ax[i].set_ylabel('Beschleunigung in m/s²')
    ax[i].set_ylim(-9, 9)
    ax[i].grid(True)

plt.tight_layout()
plt.show()
```

Die einheitliche y-Achse macht den Amplitudenvergleich erst möglich. Im
Überlastbetrieb ist die Anfangsamplitude deutlich höher, klingt aber
langsam ab. Der Nachlauf zeigt eine kleine Amplitude, die schnell auf null
fällt. Ohne `set_ylim()` würde Matplotlib die y-Achse für jeden Subplot
automatisch skalieren, was die Amplituden optisch gleich groß erscheinen
ließe.
````

## Zusammenfassung und Ausblick

Mit `plt.subplots(nrows, ncols)` legen wir mehrere Zeichenbereiche in einer
Figure an und sprechen sie über `ax[i]` oder `ax[i, j]` an. `plt.tight_layout()`
verhindert überlappende Beschriftungen. Mit `set_xlim()` und `set_ylim()`
schränken wir den sichtbaren Bereich ein, um Ausschnitte zu vergrößern oder
mehrere Subplots auf dieselbe Skala zu bringen.

Im nächsten Kapitel ergänzen wir unsere Diagramme um weitere Diagrammtypen:
den **Scatter-Plot** für Messpunkte ohne Verbindungslinien, das **Histogramm**
für die Verteilung von Messwerten und **Error Bars** für die Darstellung von
Messunsicherheiten.
