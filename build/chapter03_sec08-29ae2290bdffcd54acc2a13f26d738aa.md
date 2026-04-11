---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.8 Übungen

````{admonition} Übung 3.6 (✩)
:class: tip
Gegeben ist folgender Code, der eine vereinfachte Schaltung modelliert:

```python
import numpy as np

R1, R2, R3, U0 = 100., 200., 150., 12.

A = np.array([
    [+1., -1., -1.],
    [ 0.,  R1,  -R2],
    [ 0.,  R1,   R3],
], dtype=float)

b = np.array([0., 0., U0])

x = np.linalg.solve(A, b)
```

1. Welche physikalische Grösse stellt jede Zeile von `A` dar? Tipp: Zeile 1
   ist eine Knotenregel, Zeilen 2 und 3 sind Maschenregeln.
2. Was geben `print(x)` und `print(np.allclose(A @ x, b))` aus?
3. Was passiert, wenn man `R2 = 100.` setzt (also $R_1 = R_2$)? Hat das
   System dann noch eine eindeutige Lösung? Prüfen Sie mit der Determinante.
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

R1, R2, R3, U0 = 100., 200., 150., 12.

A = np.array([
    [+1., -1., -1.],
    [ 0.,  R1, -R2],
    [ 0.,  R1,  R3],
], dtype=float)

b = np.array([0., 0., U0])

print(np.linalg.det(A))       # ungleich null: eindeutige Lösung
x = np.linalg.solve(A, b)
print(x)                      # Ströme in Ampere
print(np.allclose(A @ x, b))  # True

# Test mit R2 = R1 = 100
A2    = A.copy()
A2[1, 2] = -100.
print(f'\nDet bei R2=100: {np.linalg.det(A2):.4f}')
```

Zeile 1 ist die Knotenregel: $I_0 = I_1 + I_2$. Zeilen 2 und 3 sind
Maschenregeln mit den jeweiligen Ohmschen Spannungsabfällen. Bei
$R_1 = R_2 = 100\,\Omega$ wird die Determinante nicht null, weil $R_3$
noch einen Unterschied macht. Das System bleibt lösbar.
````

````{admonition} Übung 3.7 (✩✩)
:class: tip
Eine Wand besteht aus vier Schichten A, B, C, D mit den thermischen
Widerständen $R_A = 0.3$, $R_B = 1.5$, $R_C = 0.8$, $R_D = 0.2$ K/W.
Die Temperaturen an den Aussenseiten betragen $T_{LA} = 293$ K (aussen)
und $T_{DR} = 273$ K (innen, Kühlraum).

Die Gleichgewichtsbedingungen lauten analog zu Notebook 3.3:

$$\frac{T_{LA} - T_{AB}}{R_A} = Q, \quad
\frac{T_{AB} - T_{BC}}{R_B} = Q, \quad
\frac{T_{BC} - T_{CD}}{R_C} = Q, \quad
\frac{T_{CD} - T_{DR}}{R_D} = Q$$

1. Leiten Sie die Matrixform
   $\mathbf{A} \cdot \vec{x} = \vec{b}$ her, wobei
   $\vec{x} = (T_{AB}, T_{BC}, T_{CD}, Q)^T$. Schreiben Sie die vier
   umgeformten Gleichungen zunächst auf Papier.
2. Implementieren Sie das System in NumPy und lösen Sie es.
3. Geben Sie alle vier Unbekannten mit Einheiten aus.
4. Welche Schicht bewirkt den grössten Temperaturabfall? Ist das
   physikalisch plausibel?
5. Erstellen Sie das Temperaturprofil als Linienplot.

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
R_A  = 0.3;  R_B = 1.5;  R_C = 0.8;  R_D = 0.2
T_LA = 293.;  T_DR = 273.

# Verarbeitung: LGS aufstellen
# Unbekannte: [T_AB, T_BC, T_CD, Q]
# Gleichung 1: T_AB + R_A*Q = T_LA
# Gleichung 2: -T_AB + T_BC + R_B*Q = 0
# Gleichung 3: -T_BC + T_CD + R_C*Q = 0
# Gleichung 4: -T_CD + R_D*Q = -T_DR

A = np.array([
    [+1.,  0.,  0., R_A],
    [-1., +1.,  0., R_B],
    [ 0., -1., +1., R_C],
    [ 0.,  0., -1., R_D],
])
b = np.array([T_LA, 0., 0., -T_DR])

x = np.linalg.solve(A, b)
T_AB, T_BC, T_CD, Q = x

# Ausgabe
print(f'T_AB = {T_AB:.2f} K')
print(f'T_BC = {T_BC:.2f} K')
print(f'T_CD = {T_CD:.2f} K')
print(f'Q    = {Q:.3f} W')
print(f'Probe: {np.allclose(A @ x, b)}')

dT_A = T_AB - T_LA
dT_B = T_BC - T_AB
dT_C = T_CD - T_BC
dT_D = T_DR - T_CD
print(f'\nDelta T A: {dT_A:.2f} K')
print(f'Delta T B: {dT_B:.2f} K  (groesster Abfall)')
print(f'Delta T C: {dT_C:.2f} K')
print(f'Delta T D: {dT_D:.2f} K')

# Plot
pos  = [0, R_A, R_A+R_B, R_A+R_B+R_C, R_A+R_B+R_C+R_D]
temp = [T_LA, T_AB, T_BC, T_CD, T_DR]

fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(pos, temp, color='#C44E52', linewidth=2.5,
        marker='o', markersize=8, label='Temperaturprofil')
ax.set_xlabel('Thermischer Widerstand (kumuliert) in K/W')
ax.set_ylabel('Temperatur in K')
ax.set_title('Vierschichtwand: Temperaturprofil')
ax.legend()
ax.grid(True)
plt.tight_layout()
plt.show()
```

Schicht B ($R_B = 1.5$ K/W) hat den grössten Widerstand und daher den
grössten Temperaturabfall. Das ist die Dämmschicht und entspricht dem
Ziel einer Wärmedämmung.
````

````{admonition} Übung 3.8 (✩✩)
:class: tip
In Notebook 3.4 und 3.5 haben wir die Wheatstone-Messbrücke mit
$R_1 = R_2 = R_3 = 100\,\Omega$, $R_B = 10\,\Omega$ und $U_0 = 10$ V
analysiert. In dieser Übung erweitern wir die Analyse.

a) Implementieren Sie die vollständige Funktion `solve_bridge_full(R4)`,
   die nicht nur den Querstrom $I$, sondern den gesamten Lösungsvektor
   $\vec{x} = (I_0, I_1, I_2, I_3, I_4, I)^T$ zurückgibt.

b) Berechnen Sie für $R_4 = 150\,\Omega$ alle sechs Ströme und geben Sie
   sie mit Einheiten aus. Überprüfen Sie die Knotenregel an Knoten K2
   ($I_1 = I + I_2$) numerisch.

c) Berechnen Sie für $R_4 = 150\,\Omega$ die Verlustleistung in jedem
   der fünf Widerstände ($P = R \cdot I^2$) und geben Sie die Gesamtleistung
   aus. Überprüfen Sie, ob die Gesamtleistung gleich $U_0 \cdot I_0$ ist
   (Energieerhaltung).

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
R1 = 100.; R2 = 100.; R3 = 100.; RB = 10.; U0 = 10.
R4 = 150.

def solve_bridge_full(R4):
    """Gibt den vollständigen Lösungsvektor [I0,I1,I2,I3,I4,I] zurück."""
    A = np.array([
        [+1., -1.,  0., -1.,   0.,   0.],
        [ 0., +1., -1.,  0.,   0.,  -1.],
        [ 0.,  0.,  0., +1.,  -1.,  +1.],
        [ 0.,  R1,  R2,  0.,   0.,   0.],
        [ 0.,  0.,  0.,  R3,   R4,   0.],
        [ 0.,  R1,  0., -R3,   0.,  -RB],
    ])
    b = np.array([0., 0., 0., U0, U0, 0.])
    return np.linalg.solve(A, b)

# Verarbeitung
x = solve_bridge_full(R4)
I0, I1, I2, I3, I4, I = x

# Ausgabe Teil b
print(f'I0 = {I0*1000:.4f} mA  (Gesamtstrom)')
print(f'I1 = {I1*1000:.4f} mA  (durch R1)')
print(f'I2 = {I2*1000:.4f} mA  (durch R2)')
print(f'I3 = {I3*1000:.4f} mA  (durch R3)')
print(f'I4 = {I4*1000:.4f} mA  (durch R4)')
print(f'I  = {I*1000:.4f} mA   (Querstrom durch RB)')

print(f'\nKnotenregel K2: I1 = I + I2 -> {np.isclose(I1, I + I2)}')

# Verarbeitung Teil c
widerstaende = [R1, R2, R3, R4, RB]
stroeme      = [I1, I2, I3, I4, I]
namen        = ['R1', 'R2', 'R3', 'R4', 'RB']

P_gesamt = 0.
for name, R, Ii in zip(namen, widerstaende, stroeme):
    P_i      = R * Ii**2
    P_gesamt += P_i
    print(f'P_{name} = {P_i*1000:.4f} mW')

P_quelle = U0 * I0

# Ausgabe Teil c
print(f'\nGesamtleistung Widerstände: {P_gesamt*1000:.4f} mW')
print(f'Leistung Quelle U0*I0:      {P_quelle*1000:.4f} mW')
print(f'Energieerhaltung: {np.isclose(P_gesamt, P_quelle)}')
```
````

````{admonition} Übung 3.9 (✩✩✩) Mini-Projekt: Brücke mit variablem R_B
:class: tip
In Notebook 3.5 haben wir $R_4$ variiert und $I(R_4)$ untersucht. In
dieser Übung variieren wir stattdessen den Brückenwiderstand $R_B$ bei
festem $R_4 = 120\,\Omega$ (Messwert, unbekannt von der abgeglichenen
Brücke bei $R_4^* = 100\,\Omega$).

**Teil 1: Parameterstudie**

Berechnen Sie $I(R_B)$ und $P_B(R_B) = R_B \cdot I^2$ für
$R_B \in [1\,\Omega, 500\,\Omega]$. Zeigen Sie beide Grössen in einem
Subplot übereinander.

**Teil 2: Optimaler Brückenwiderstand**

Für welchen Wert von $R_B$ wird die Verlustleistung $P_B$ maximal? Bestimmen
Sie diesen Wert numerisch mit `np.argmax` und markieren Sie ihn im Diagramm.

**Teil 3: Interpretation**

Der Wert von $R_B$, bei dem $P_B$ maximal wird, ist bekannt als
Leistungsanpassung: Die maximale Leistung wird an den Verbraucher
übertragen, wenn sein Widerstand gleich dem Innenwiderstand der Quelle ist.
Berechnen Sie den Innenwiderstand der Quelle, gesehen vom Brückenwiderstand
aus (Thevenin-Widerstand), analytisch:

$$R_{Th} = \frac{R_1 R_2}{R_1 + R_2} + \frac{R_3 R_4}{R_3 + R_4}$$

Vergleichen Sie $R_{Th}$ mit dem numerisch gefundenen optimalen $R_B$.

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
R1 = 100.; R2 = 100.; R3 = 100.; R4 = 120.; U0 = 10.

def solve_bridge_RB(RB):
    """Querstrom I und Verlustleistung PB für gegebenes RB."""
    A = np.array([
        [+1., -1.,  0., -1.,   0.,   0.],
        [ 0., +1., -1.,  0.,   0.,  -1.],
        [ 0.,  0.,  0., +1.,  -1.,  +1.],
        [ 0.,  R1,  R2,  0.,   0.,   0.],
        [ 0.,  0.,  0.,  R3,   R4,   0.],
        [ 0.,  R1,  0., -R3,   0.,  -RB],
    ])
    b  = np.array([0., 0., 0., U0, U0, 0.])
    x  = np.linalg.solve(A, b)
    I  = x[5]
    PB = RB * I**2
    return I, PB

# Verarbeitung: Parameterstudie
RB_werte = np.linspace(1., 500., 1000)
I_werte  = np.zeros(len(RB_werte))
PB_werte = np.zeros(len(RB_werte))

for i, RB in enumerate(RB_werte):
    I_werte[i], PB_werte[i] = solve_bridge_RB(RB)

# Verarbeitung: Optimum
idx_max  = np.argmax(PB_werte)
RB_opt   = RB_werte[idx_max]
PB_max   = PB_werte[idx_max]

# Thevenin-Widerstand
R_Th = (R1 * R2) / (R1 + R2) + (R3 * R4) / (R3 + R4)

# Ausgabe
print(f'Optimales RB (numerisch): {RB_opt:.1f} Ohm')
print(f'Thevenin-Widerstand:      {R_Th:.1f} Ohm')
print(f'Übereinstimmung: {np.isclose(RB_opt, R_Th, atol=5.)}')

fig, ax = plt.subplots(nrows=2, ncols=1, figsize=(9, 7), sharex=True)

ax[0].plot(RB_werte, I_werte * 1000, color='#4C72B0', linewidth=2)
ax[0].set_ylabel('Querstrom I in mA')
ax[0].set_title('Messbrücke: Querstrom und Verlustleistung als Funktion von R_B')
ax[0].grid(True)

ax[1].plot(RB_werte, PB_werte * 1000, color='#DD8452', linewidth=2,
           label='Verlustleistung')
ax[1].scatter([RB_opt], [PB_max * 1000], color='#C44E52', s=100, zorder=5,
              label=f'Maximum bei R_B = {RB_opt:.1f} Ohm')
ax[1].axvline(RB_opt, color='#C44E52', linestyle='dashed', linewidth=1.5)
ax[1].set_xlabel('R_B in Ohm')
ax[1].set_ylabel('Verlustleistung P_B in mW')
ax[1].legend()
ax[1].grid(True)

plt.tight_layout()
plt.show()
```
````
