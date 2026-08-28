---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 2.8 Übungen

```{admonition} Warnung
:class: warning
Dieses Kapitel befindet sich derzeit im Umbau und wird rechtzeitig vor der Vorlesung im WiSe 2026/27 zur Verfügung stehen.
```

````{admonition} Übung 2.1 (✩)
:class: tip
Gegeben ist folgender Code:

```python
import numpy as np

a = [1, 2, 3] + [4, 5, 6]
b = np.array([1, 2, 3]) + np.array([4, 5, 6])
c = np.array([1, 2.0, 3])
d = np.array([10, 20, 30, 40, 50])
```

1. Sagen Sie vorher, was `print(a)` und `print(b)` ausgeben. Worin besteht
   der Unterschied?
2. Sagen Sie vorher, was `print(c.dtype)` ausgibt. Warum wählt NumPy diesen
   Typ, obwohl zwei der drei Werte ganze Zahlen sind?
3. Was geben `print(d[-1])` und `print(d[-2])` aus? Erklären Sie, warum
   negative Indizes auch bei NumPy-Arrays funktionieren.
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

a = [1, 2, 3] + [4, 5, 6]
b = np.array([1, 2, 3]) + np.array([4, 5, 6])
c = np.array([1, 2.0, 3])
d = np.array([10, 20, 30, 40, 50])

print(a)         # [1, 2, 3, 4, 5, 6]
print(b)         # [5 7 9]
print(c.dtype)   # float64
print(d[-1])     # 50
print(d[-2])     # 40
```

1. Bei Python-Listen bewirkt `+` eine Verkettung: Die beiden Listen werden zu
   einer zusammengefügt. Bei NumPy-Arrays bedeutet `+` eine elementweise
   Addition: Jedes Element des ersten Arrays wird zum entsprechenden Element
   des zweiten Arrays addiert.
2. NumPy wählt `float64`, weil das Array den Wert `2.0` enthält. Da ein
   Array nur einen einzigen Datentyp enthalten kann, wandelt NumPy alle
   Werte in `float64` um. Ganzzahlen lassen sich verlustfrei als
   Fließkommazahlen darstellen, umgekehrt aber nicht.
3. `d[-1]` liefert `50` (letztes Element), `d[-2]` liefert `40`
   (vorletztes Element). NumPy unterstützt denselben negativen Index wie
   Python-Listen: intern addiert Python die Länge des Arrays zum negativen
   Index.
````

````{admonition} Übung 2.2 (✩)
:class: tip
Gegeben ist folgende Matrix:

\begin{equation*}
A = \begin{pmatrix}
    1.0 & 2.0 & 3.0 \\
    4.0 & 5.0 & 6.0 \\
    7.0 & 8.0 & 9.0 \\
   \end{pmatrix}.
\end{equation*}

Speichern Sie die Matrix als Array in der Variable `A` und beantworten Sie
folgende Fragen zunächst nur im Kopf, ohne Python-Code auszuführen.

1. Was gibt `print(A.shape)` aus?
2. Was gibt `print(A[1, 2])` aus?
3. Was gibt `print(A[0, :])` aus?
4. Was gibt `print(A[:, 1])` aus?
5. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

A = np.array([
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
    [7.0, 8.0, 9.0],
])

print(A.shape)     # (3, 3)
print(A[1, 2])     # 6.0
print(A[0, :])     # [1. 2. 3.]
print(A[:, 1])     # [2. 5. 8.]
```

1. `(3, 3)`: drei Zeilen, drei Spalten.
2. `A[1, 2]` greift auf Zeile 1, Spalte 2 zu: der Wert ist `6.0`.
3. `A[0, :]` liefert die gesamte erste Zeile: `[1. 2. 3.]`.
4. `A[:, 1]` liefert die gesamte zweite Spalte: `[2. 5. 8.]`.
````

````{admonition} Übung 2.3 (✩✩)
:class: tip
Ein Temperatursensor misst in einem Ofen stündlich die Temperatur über einen
Arbeitstag von 8 Stunden. Die acht Messwerte in °C lauten:

```code
180.2, 195.4, 201.7, 198.3, 202.1, 199.8, 197.5, 193.6
```

1. Legen Sie die Messwerte als NumPy-Array an. Berechnen Sie Mittelwert,
   Standardabweichung, Minimum und Maximum und geben Sie die Ergebnisse
   formatiert aus.
2. Der Ofen läuft innerhalb der Toleranz, wenn kein Messwert mehr als 10 °C vom
   Mittelwert abweicht. Berechnen Sie die maximale Abweichung und prüfen Sie mit
   einer `if`-Bedingung, ob der Ofen im Toleranzbereich läuft.
3. Die Temperatur soll in Kelvin umgerechnet werden: $T_K = T_{°C} + 273.15$.
   Berechnen Sie das Array `temperatur_k` und geben Sie es aus.

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

# Eingabe
temperatur = np.array([180.2, 195.4, 201.7, 198.3, 202.1, 199.8, 197.5, 193.6])
toleranz   = 10.0   # maximale Abweichung in °C

# Verarbeitung
mittelwert     = np.mean(temperatur)
std            = np.std(temperatur)
t_min          = np.min(temperatur)
t_max          = np.max(temperatur)
max_abweichung = np.max(np.abs(temperatur - mittelwert))
temperatur_k   = temperatur + 273.15

# Ausgabe
print(f"Mittelwert:          {mittelwert:.2f} °C")
print(f"Standardabweichung:  {std:.2f} °C")
print(f"Minimum:             {t_min:.2f} °C")
print(f"Maximum:             {t_max:.2f} °C")
print(f"Maximale Abweichung: {max_abweichung:.2f} °C")

if max_abweichung <= toleranz:
    print("Ofen laeuft innerhalb der Toleranz.")
else:
    print("Toleranz ueberschritten!")

print(f"Temperatur in Kelvin: {temperatur_k}")
```

Ausgabe:
```
Mittelwert:          196.07 °C
Standardabweichung:  6.59 °C
Minimum:             180.20 °C
Maximum:             202.10 °C
Maximale Abweichung: 15.88 °C
Toleranz ueberschritten!
Temperatur in Kelvin: [453.35 468.55 474.85 471.45 475.25 472.95 470.65 466.75]
```

Die maximale Abweichung beträgt 15.88 °C und überschreitet die Toleranz von
10 °C. Der erste Messwert von 180.2 °C weicht am stärksten vom Mittelwert ab
und deutet auf eine Aufheizphase zu Beginn des Arbeitstages hin.
````

````{admonition} Übung 2.4 (✩✩)
:class: tip
Drei Sensoren an einer Maschine werden mit drei Referenzbelastungen kalibriert.
Das Gleichungssystem $\mathbf{A} \cdot \vec{x} = \vec{b}$ lautet:

$$\begin{pmatrix} 2.0 & 1.0 & 0.5 \\ 1.0 & 3.0 & 1.0 \\ 0.5 & 1.0 & 2.5
\end{pmatrix} \cdot \vec{x}
= \begin{pmatrix} 4.5 \\ 9.0 \\ 7.0 \end{pmatrix}$$

Die rechte Seite enthält die bekannten Referenzkräfte in kN, der Lösungsvektor
$\vec{x}$ die gesuchten Kalibrierungsfaktoren in kN/V.

1. Lösen Sie das Gleichungssystem mit NumPy.
2. Überprüfen Sie das Ergebnis mit einer Probe: Berechnen Sie $\mathbf{A}
   \cdot \vec{x}$ und vergleichen Sie es mit $\vec{b}$.
3. Berechnen Sie zusätzlich die Inverse von $\mathbf{A}$ und lösen Sie das
   System erneut über $\vec{x} = \mathbf{A}^{-1} \cdot \vec{b}$. Stimmen
   beide Ergebnisse überein?

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

# Eingabe
A = np.array([
    [2.0, 1.0, 0.5],
    [1.0, 3.0, 1.0],
    [0.5, 1.0, 2.5],
])
b = np.array([4.5, 9.0, 7.0])

# Verarbeitung
x     = np.linalg.solve(A, b)
probe = A @ x
x_inv = np.linalg.inv(A) @ b

# Ausgabe
print(f"Loesung (solve): {x}")
print(f"Probe A @ x:     {probe}")
print(f"Loesung (inv):   {x_inv}")
```

Ausgabe:
```
Loesung (solve): [0.72093023 2.1627907  1.79069767]
Probe A @ x:     [4.5 9.  7. ]
Loesung (inv):   [0.72093023 2.1627907  1.79069767]
```

Beide Methoden liefern dasselbe Ergebnis. Die Probe bestätigt, dass $\mathbf{A}
\cdot \vec{x} = \vec{b}$ exakt erfüllt ist. Die Kalibrierungsfaktoren der drei
Sensoren betragen ca. 0.72, 2.16 und 1.79 kN/V.
````

````{admonition} Übung 2.5 (✩✩✩) Mini-Projekt: Schwingungsanalyse mit Rauschen
:class: tip
In dieser Aufgabe analysieren wir ein verrauschtes Schwingungssignal und
untersuchen, wie gut statistische Kenngrößen das wahre Signal beschreiben.

**Teil 1: Signal erzeugen**

Schreiben Sie eine Funktion `erzeuge_signal(A, delta, f, t)`, die das
gedämpfte Schwingungssignal

$$a(t) = A \cdot e^{-\delta t} \cdot \sin(2\pi f t)$$

als NumPy-Array zurückgibt. Versehen Sie die Funktion mit einem Docstring.

Erzeugen Sie damit ein sauberes Signal mit der Anfangsamplitude $A = 5.0$
m/s², dem Dämpfungskoeffizienten $\delta = 1.5$ 1/s, der Frequenz $f = 10.0$
Hz und einer Zeitachse von 0 bis 2 Sekunden mit 1000 Punkten.

**Teil 2: Rauschen hinzufügen**

Schreiben Sie eine Funktion `addiere_rauschen(signal, sigma, seed)`, die dem
Signal normalverteiltes Rauschen mit Standardabweichung `sigma` hinzufügt.
Der Parameter `seed` soll die Reproduzierbarkeit sicherstellen. Geben Sie
das verrauschte Signal zurück.

Erzeugen Sie drei verrauschte Versionen des Signals mit den Seeds 0, 1 und
2, jeweils mit einer Standardabweichung von $\sigma = 0.5$ m/s².

**Teil 3: Statistische Auswertung**

Berechnen Sie für das saubere Signal und jede der drei verrauschten Versionen
Mittelwert, Standardabweichung, Maximum und Minimum. Geben Sie die Ergebnisse
in einer übersichtlichen Tabelle aus.

**Teil 4: Reflexion**

Welche der vier Kenngrößen wird durch das Rauschen am stärksten beeinflusst,
welche am wenigsten? Begründen Sie kurz.

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

def erzeuge_signal(A, delta, f, t):
    """Berechnet ein gedämpftes Schwingungssignal.

    A:     Anfangsamplitude in m/s²
    delta: Dämpfungskoeffizient in 1/s
    f:     Frequenz in Hz
    t:     Zeitachse als NumPy-Array in s
    Rückgabe: Signal als NumPy-Array in m/s²
    """
    return A * np.exp(-delta * t) * np.sin(2 * np.pi * f * t)

def addiere_rauschen(signal, sigma, seed):
    """Addiert normalverteiltes Rauschen auf ein Signal.

    signal: sauberes Signal als NumPy-Array
    sigma:  Standardabweichung des Rauschens
    seed:   Zufallsseed fuer Reproduzierbarkeit
    Rueckgabe: verrauschtes Signal als NumPy-Array
    """
    np.random.seed(seed)
    rauschen = np.random.normal(0.0, sigma, size=len(signal))
    return signal + rauschen

# Eingabe
A     = 5.0
delta = 1.5
f     = 10.0
t     = np.linspace(0, 2, 1000)
sigma = 0.5

# Verarbeitung: erzeuge verrauschte Signale
a_sauber = erzeuge_signal(A, delta, f, t)
a0 = a_sauber + addiere_rauschen(a_sauber, sigma, 0)
a1 = a_sauber + addiere_rauschen(a_sauber, sigma, 1)
a2 = a_sauber + addiere_rauschen(a_sauber, sigma, 2)

# Ausgabe
print("Signal  | Mittelw.| Std    | Max     | Min")
print("-" * 50)
print(f"sauber  | {np.mean(a_sauber):.4f}  | {np.std(a_sauber):.4f} | {np.max(a_sauber):.4f}  | {np.min(a_sauber):.4f}")
print(f"Seed 0  | {np.mean(a0):.4f}  | {np.std(a0):.4f} | {np.max(a0):.4f} | {np.min(a0):.4f}")
print(f"Seed 1  | {np.mean(a1):.4f}  | {np.std(a1):.4f} | {np.max(a1):.4f}  | {np.min(a1):.4f}")
print(f"Seed 2  | {np.mean(a2):.4f}  | {np.std(a2):.4f} | {np.max(a2):.4f} | {np.min(a2):.4f}")
```

Ausgabe:
```
Signal  | Mittelw.| Std    | Max     | Min
--------------------------------------------------
sauber  | 0.0377  | 1.4400 | 4.8140  | -4.4666
Seed 0  | 0.0528  | 2.9286 | 10.2333 | -9.0948
Seed 1  | 0.0948  | 2.9243 | 9.9822  | -9.3825
Seed 2  | 0.0515  | 2.8949 | 10.6523 | -9.3554
```

**Teil 4:** Maximum und Minimum werden am stärksten durch das Rauschen
beeinflusst, weil ein einzelner großer Ausreißer den Extremwert deutlich
verschieben kann. Der Mittelwert hingegen wird kaum beeinflusst: Da das
Rauschen normalverteilt mit Erwartungswert null ist, heben sich positive
und negative Ausreißer beim Mitteln fast vollständig auf. Die
Standardabweichung liegt bei allen verrauschten Signalen höher als beim
sauberen Signal, weil das Rauschen zur Gesamtvarianz beiträgt.
````

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

```code
3.2, 2.8, 2.5, 2.1, 2.4, 3.0, 4.1, 5.3, 6.2, 7.0, 7.8, 8.1,
7.5, 6.9, 6.3, 5.8, 5.1, 4.7, 4.2, 3.8, 3.5, 3.3, 3.1, 2.9.
```

1. Legen Sie die Daten (Stunden und Windgeschwindigkeit) als NumPy-Arrays an und
   erstellen Sie einen Linienplot der Windgeschwindigkeit über die Tageszeit.
   Verwenden Sie Kreise als Marker.
2. Markieren Sie den Zeitpunkt der maximalen Windgeschwindigkeit mit einem
   auffälligen Marker in einer anderen Farbe. Hinweis: `np.argmax()` liefert den
   Index des größten Werts.
3. Fügen Sie eine horizontale gestrichelte Linie bei der mittleren
   Windgeschwindigkeit ein. Hinweis: `ax.axhline()` zeichnet eine horizontale
   Linie über den gesamten Plotbereich.
4. Beschriften Sie die Achsen mit Einheiten, vergeben Sie einen Titel und zeigen
   Sie die Legende an.

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
   Verwenden Sie `np.random.seed(17)`. Fassen Sie alle 50 Werte in einem
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
proben            = np.array([1, 2, 3, 4, 5])  # Probennummer 
bruchspannung     = np.array([42.3, 38.7, 51.2, 45.8, 39.1])  # Mittelwert Bruchspannung in MPa
std_bruchspannung = np.array([ 2.1,  3.4,  1.8,  2.7,  3.0])  # Standardabweichung Bruchspannung in MPa

# Simulation Bruchspannungen
np.random.seed(17)
simulierte_bruchspannungen = np.zeros(50)
idx = 0
for i in range(5):
    simulation = np.random.normal(bruchspannung[i], std_bruchspannung[i], 10)
    for j in range(10):
        simulierte_bruchspannungen[idx] = simulation[j]  # idx läuft von 0 bis 49
        idx += 1

# Verarbeitung
mittelwert = np.mean(bruchspannung)

# Ausgabe
fig, ax = plt.subplots(nrows=1, ncols=2, figsize=(12, 5))

ax[0].errorbar(proben, bruchspannung, yerr=std_bruchspannung,
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

ax[1].hist(simulierte_bruchspannungen, bins=15, color='#4C72B0', edgecolor='white')
ax[1].set_xlabel('Simulierte Bruchspannung in MPa')
ax[1].set_ylabel('Häufigkeit')
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

```{admonition} Übung 2.10 (✩✩✩) Mini-Projekt: Heizenergie und Außentemperatur
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

Erstellen Sie eine Figure mit drei nebeneinanderliegenden Subplots:

- Links: Scatter-Plot mit Temperatur auf der x-Achse und Gasverbrauch auf
  der y-Achse, überlagert mit der Regressionsgerade. Markieren Sie den
  Vorhersagepunkt aus Teil 3 auffällig.
- Mitte: monatlicher Temperaturverlauf als Linienplot mit Markern.
- Rechts: monatlicher Gasverbrauch als Linienplot mit Markern. Verwenden
  Sie für die mittleren und rechten Subplots die Monatsnamen auf der x-Achse.

**Teil 3: Vorhersage**

Im kommenden Januar wird eine mittlere Außentemperatur von −5 °C erwartet.
Berechnen Sie den vorhergesagten Gasverbrauch mit der Regressionsgerade.

**Teil 4: Reflexion**

Bewerten Sie kurz: Passt ein lineares Modell hier gut? Welche physikalische
Erklärung hat das Vorzeichen der Steigung?

Strukturieren Sie Ihren Code mit Kommentaren.
```

```{code-cell}
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
temperatur   = np.array([-2.,  1.,  5., 10., 15., 18., 21., 20., 14.,  9.,  3., -1.])  # in °C
gasverbrauch = np.array([350, 290, 210, 115,  60,  30,  15,  20,  75, 130, 230, 320],
                         dtype=float)  # in kWh, float da sonst Integer-Array
monate = ['Jan', 'Feb', 'Mär', 'Apr', 'Mai', 'Jun',
          'Jul', 'Aug', 'Sep', 'Okt', 'Nov', 'Dez']

# Verarbeitung: Mittelwerte berechnen
x_mean = np.mean(temperatur)
y_mean = np.mean(gasverbrauch)

# Verarbeitung: Steigung und Achsenabschnitt der Regressionsgeraden
m = np.sum((temperatur - x_mean) * (gasverbrauch - y_mean)) / \
    np.sum((temperatur - x_mean)**2)
b = y_mean - m * x_mean

print(f"Steigung:        {m:.2f} kWh/°C")
print(f"Achsenabschnitt: {b:.1f} kWh")

# Verarbeitung: Regressionslinie und Vorhersage
x_linie    = np.linspace(-6, 23, 200)   # Temperaturbereich für die Linie
y_linie    = m * x_linie + b            # Gasverbrauch entlang der Linie
vorhersage = m * (-5) + b               # Vorhersage für -5 °C

print(f"Vorhersage für -5 °C: {vorhersage:.0f} kWh")

# Ausgabe: drei Subplots nebeneinander
fig, ax = plt.subplots(nrows=1, ncols=3, figsize=(14, 5))
ax_scatter, ax_temp, ax_gas = ax

# Linker Subplot: Scatter-Plot mit Regressionsgerade
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

# Mittlerer Subplot: monatliche Außentemperatur
ax_temp.plot(np.arange(12), temperatur, color='#4C72B0',
             marker='o', markersize=5, linewidth=1.5)
ax_temp.set_ylabel('Außentemperatur in °C')
ax_temp.set_title('Monatliche Außentemperatur')
ax_temp.set_xticks(np.arange(12))
ax_temp.set_xticklabels(monate, rotation=45)
ax_temp.grid(True)

# Rechter Subplot: monatlicher Gasverbrauch
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
Achsenabschnitt: 291.1 kWh
Vorhersage für -5 °C: 364 kWh
```

**Teil 4:** Das lineare Modell beschreibt den Zusammenhang im beobachteten
Bereich gut: Kälter bedeutet mehr Heizenergie. Die negative Steigung
(−14.59 kWh/°C) ist physikalisch plausibel, weil der Temperaturunterschied
zwischen Innen und Außen den Wärmeverlust des Gebäudes bestimmt. Für sehr
hohe Temperaturen würde das Modell negative Gasverbräuche vorhersagen, was
physikalisch unsinnig ist. Im beobachteten Bereich von −2 °C bis +21 °C ist
die Näherung jedoch gut geeignet.
````
