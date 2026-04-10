---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.3 Anwendung: Wärmeübertragung in einer Mehrschichtwand

Bisher haben wir LGS an einem Obstmarkt-Beispiel geübt. Jetzt wenden wir
dasselbe Werkzeug auf ein Problem aus dem Maschinenbau an: eine Außenwand
aus drei Schichten mit unterschiedlichen Wärmedurchgangswiderständen. *Wie
hoch ist die Temperatur an den Grenzflächen, und wie groß ist der
Wärmestrom?*

Wir werden sehen, dass der Weg von den physikalischen Gleichungen zur
Matrix immer gleich ist: Unbekannte auf die linke Seite, bekannte Größen
auf die rechte.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können physikalische Gleichgewichtsbedingungen in ein LGS
  $\mathbf{A} \cdot \vec{x} = \vec{b}$ überführen.
* [ ] Sie können die Matrix $\mathbf{A}$ und den Vektor $\vec{b}$ aus
  umgeformten Gleichungen ablesen.
* [ ] Sie können das LGS lösen und das Ergebnis physikalisch interpretieren.
```

+++

## Das physikalische Modell

Wir betrachten eine Wand aus drei Schichten A, B, C mit den thermischen
Widerständen $R_A$, $R_B$, $R_C$ in K/W. Links herrscht die Temperatur
$T_{LA}$, rechts $T_{CR}$.

```{figure} pics/waermeuebertragung_mehrschichtwand.svg
:alt: Querschnitt einer dreischichtigen Wand mit Temperaturprofil und Wärmestrom
:align: center

Temperaturprofil einer Mehrschichtwand im stationären Zustand. Im stationären Fall ist der Wärmestrom durch alle Schichten gleich; der Temperatursprung (Temperaturabfall) über jede Schicht ist proportional zu ihrem thermischen Widerstand, sodass der Temperatursprung über Schicht C am größten ist.
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

Im **stationären Zustand** ist der Wärmestrom $Q$ durch alle Schichten
gleich. Das **Wärmeübertragungsgesetz** (analog zum Ohmschen Gesetz) lautet
für jede Schicht:

$$Q = \frac{\Delta T_i}{R_i}$$

Das liefert drei Gleichungen mit drei Unbekannten ($T_{AB}$, $T_{BC}$, $Q$):

$$\frac{T_{LA} - T_{AB}}{R_A} = Q \qquad (1)$$

$$\frac{T_{AB} - T_{BC}}{R_B} = Q \qquad (2)$$

$$\frac{T_{BC} - T_{CR}}{R_C} = Q \qquad (3)$$

+++

## Von den Gleichungen zur Matrixform

Wir bringen alle Unbekannten auf die linke Seite. Dafür multiplizieren wir
jede Gleichung mit $R_i$ und ordnen um:

$$T_{AB} + R_A \cdot Q = T_{LA} \qquad (1')$$

$$-T_{AB} + T_{BC} + R_B \cdot Q = 0 \qquad (2')$$

$$-T_{BC} + R_C \cdot Q = -T_{CR} \qquad (3')$$

Jetzt lesen wir die Koeffizientenmatrix zeilenweise ab.
Der Lösungsvektor ist $\vec{x} = (T_{AB},\, T_{BC},\, Q)^T$:

$$\underbrace{\begin{pmatrix}
+1 &  0 & R_A \\
-1 & +1 & R_B \\
 0 & -1 & R_C
\end{pmatrix}}_{\mathbf{A}}
\cdot
\underbrace{\begin{pmatrix} T_{AB} \\ T_{BC} \\ Q \end{pmatrix}}_{\vec{x}}
=
\underbrace{\begin{pmatrix} T_{LA} \\ 0 \\ -T_{CR} \end{pmatrix}}_{\vec{b}}$$

```{admonition} Rezept: Matrixform ablesen
:class: note
Jede umgeformte Gleichung liefert eine Zeile von $\mathbf{A}$ und einen
Eintrag in $\vec{b}$. Der Koeffizient der $j$-ten Unbekannten in der
$i$-ten Gleichung steht in $A_{ij}$. Unbekannte, die nicht vorkommen,
erhalten den Koeffizienten 0.
```

+++

## Implementierung und Lösung

```{code-cell} python
import numpy as np

# Gegebene Groessen
R_A  = 0.5   # thermischer Widerstand Schicht A in K/W
R_B  = 0.3   # thermischer Widerstand Schicht B in K/W
R_C  = 0.7   # thermischer Widerstand Schicht C in K/W
T_LA = 300.  # Temperatur linke Seite in K
T_CR = 310.  # Temperatur rechte Seite in K
```

```{code-cell} python
# Koeffizientenmatrix ablesen (Zeile = Gleichung 1', 2', 3')
# Spalten entsprechen den Unbekannten: [T_AB, T_BC, Q]
A = np.array([
    [+1.,  0., R_A],   # Gleichung 1': +T_AB + 0*T_BC + R_A*Q = T_LA
    [-1., +1., R_B],   # Gleichung 2': -T_AB + T_BC  + R_B*Q = 0
    [ 0., -1., R_C],   # Gleichung 3':  0    - T_BC  + R_C*Q = -T_CR
])

# Rechte Seite
b = np.array([T_LA, 0., -T_CR])

print('Koeffizientenmatrix A:')
print(A)
print('Rechte Seite b:', b)
```

```{code-cell} python
# Lösbarkeitstest (gelernt in Kapitel 3.1)
det_A = np.linalg.det(A)
print(f'Determinante: {det_A:.4f}')

if np.isclose(det_A, 0.):
    print('Keine eindeutige Lösung.')
else:
    print('Eindeutige Lösung vorhanden.')
```

```{code-cell} python
# Loesung berechnen (gelernt in Kapitel 3.2)
x = np.linalg.solve(A, b)

# Ergebnis entpacken: x[0]=T_AB, x[1]=T_BC, x[2]=Q
T_AB = x[0]
T_BC = x[1]
Q    = x[2]

print(f'T_AB = {T_AB:.2f} K   (Grenzfläche A-B)')
print(f'T_BC = {T_BC:.2f} K   (Grenzfläche B-C)')
print(f'Q    = {Q:.4f} W   (Wärmestrom)')

# Probe
print('Probe bestanden:', np.allclose(A @ x, b))
```

Da $T_{CR} > T_{LA}$, fließt Wärme von rechts nach links: $Q$ ist negativ.
Schicht C hat den größten Widerstand ($R_C = 0{,}7$ K/W) und damit den
größten Temperatursprung, genau wie der größte Widerstand in einem
elektrischen Schaltkreis den größten Spannungsabfall hat.

```{code-cell} python
# Temperaturdifferenzen ueber jede Schicht zur Kontrolle
delta_A = T_AB - T_LA
delta_B = T_BC - T_AB
delta_C = T_CR - T_BC

print(f'Temperaturdiff. Schicht A: {delta_A:.2f} K')
print(f'Temperaturdiff. Schicht B: {delta_B:.2f} K')
print(f'Temperaturdiff. Schicht C: {delta_C:.2f} K')
print(f'Summe (= T_CR - T_LA):     {delta_A + delta_B + delta_C:.2f} K')
```

+++

```{admonition} Mini-Übung
:class: tip
Eine Kühlhauswand hält die Innentemperatur bei $T_{\text{innen}} = 268$ K
(−5 °C), während außen $T_{\text{außen}} = 293$ K (20 °C) herrschen.

| Schicht | Material               | R (K/W) |
|---------|------------------------|---------|
| A       | Beton                  | 0.2     |
| B       | Polyurethan-Schaum     | 1.8     |
| C       | Stahlblech             | 0.05    |

Die linke Seite ist innen ($T_{LA} = T_{\text{innen}}$), die rechte
außen ($T_{CR} = T_{\text{außen}}$). Das LGS hat dieselbe Struktur wie
oben, nur mit anderen Zahlenwerten.

1. Legen Sie `A` und `b` mit den neuen Werten an.
2. Lösen Sie das System und geben Sie $T_{AB}$, $T_{BC}$ und $Q$ aus.
3. Welche Schicht hat den größten Temperaturabfall? Ist das plausibel?
```

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# Gegebene Groessen
R_A  = 0.2
R_B  = 1.8
R_C  = 0.05
T_LA = 268.   # Innen (kalt)
T_CR = 293.   # Aussen (warm)

# Koeffizientenmatrix: gleiche Struktur wie im Beispiel oben
A = np.array([
    [+1.,  0., R_A],
    [-1., +1., R_B],
    [ 0., -1., R_C],
])

# Rechte Seite
b = np.array([T_LA, 0., -T_CR])

# Loesung
x = np.linalg.solve(A, b)
T_AB, T_BC, Q = x   # entpacken in drei Variablen

print(f'T_AB = {T_AB:.2f} K  ({T_AB - 273.15:.1f} °C)')
print(f'T_BC = {T_BC:.2f} K  ({T_BC - 273.15:.1f} °C)')
print(f'Q    = {Q:.4f} W')
print('Probe bestanden:', np.allclose(A @ x, b))

# Temperaturdifferenzen
print(f'\nTemperaturabfall Schicht A (Beton):   {T_AB - T_LA:.2f} K')
print(f'Temperaturabfall Schicht B (PU-Foam): {T_BC - T_AB:.2f} K')
print(f'Temperaturabfall Schicht C (Stahl):   {T_CR - T_BC:.2f} K')
```

Den größten Temperaturabfall hat Schicht B (Polyurethan-Schaum), weil ihr
thermischer Widerstand mit $R_B = 1{,}8$ K/W bei weitem am größten ist.
Das ist genau der Sinn einer Wärmedämmung: sie übernimmt den Großteil der
Temperaturdifferenz zwischen innen und außen.
````

+++

## Zusammenfassung und Ausblick

Das Vorgehen ist immer dasselbe: physikalische Gleichungen umformen, alle
Unbekannten nach links, alle bekannten Größen nach rechts, dann
$\mathbf{A}$ und $\vec{b}$ ablesen. `np.linalg.solve` liefert die Lösung,
`np.allclose` sichert sie ab.

Dieses Muster lässt sich auf beliebig komplexe Systeme übertragen. Im
nächsten Kapitel verwenden wir es für elektrische Netzwerke mit sechs
Unbekannten, und in Kapitel 3.6 untersuchen wir, wie sich der Wärmestrom
verändert, wenn wir die Dämmstärke systematisch variieren.
