---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 5.4 Übung: Homogene Koordinaten

In Abschnitt 5.3 haben wir homogene Koordinaten und die Transformationsmatrix
$\mathbf{T}$ kennengelernt. In dieser Übung wenden wir sie auf konkrete
Aufgaben an.

+++

```{code-cell} python
# --- Importe und Hilfsfunktionen für alle Aufgaben ---
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon as MplPolygon

# Homogene Transformationsmatrix: Drehung um phi, dann Verschiebung (tx, ty)
def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([
        [c, -s, tx],
        [s,  c, ty],
        [0,  0,  1],
    ])

# Hilfsfunktion: euklidische Punktemenge -> homogene Koordinaten
def zu_homogen(punkte):
    """Haengt eine Spalte mit Einsen an: (n, 2) -> (n, 3)."""
    einsen = np.ones((punkte.shape[0], 1))
    return np.hstack([punkte, einsen])

# Hilfsfunktion: homogene Koordinaten -> euklidische Punktemenge
def zu_euklidisch(punkte_hom):
    """Gibt die ersten zwei Spalten zurueck: (n, 3) -> (n, 2)."""
    return punkte_hom[:, :2]

# Transformation auf eine Punktemenge anwenden
def transformieren(T, punkte):
    """Wendet die homogene Matrix T auf alle Punkte an."""
    return zu_euklidisch((T @ zu_homogen(punkte).T).T)

# Unser Dreieck
dreieck = np.array([
    [0.0, 0.0],   # Eckpunkt A
    [2.0, 0.0],   # Eckpunkt B
    [1.0, 2.0],   # Eckpunkt C
])
```

## Aufgabe 1: Transformationsmatrix aufbauen und verstehen

Wir untersuchen, wie sich die Transformationsmatrix verhält, wenn wir nur
Translation, nur Rotation oder beides einsetzen.

```{admonition} Aufgabe 1
:class: tip
1. Bauen Sie drei Transformationsmatrizen auf:
   - $\mathbf{T}_\text{rot}$: reine Drehung um $60°$, keine Verschiebung.
   - $\mathbf{T}_\text{trans}$: keine Drehung, Verschiebung um $(3, -2)^\top$.
   - $\mathbf{T}_\text{kombi}$: Drehung um $60°$ und Verschiebung um $(3, -2)^\top$.
2. Wenden Sie alle drei Matrizen auf Eckpunkt B $= (2, 0)^\top$ in
   homogenen Koordinaten an. Geben Sie die Ergebnisse aus.
3. Vergleichen Sie $\mathbf{T}_\text{kombi}$ mit dem Produkt
   $\mathbf{T}_\text{trans} \cdot \mathbf{T}_\text{rot}$. Sind sie
   identisch? Was sagt das über die Struktur von $\mathbf{T}$ aus?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx], [s, c, ty], [0, 0, 1]])

# Teilaufgabe 1
T_rot   = transformationsmatrix(np.deg2rad(60), 0.0,  0.0)
T_trans = transformationsmatrix(0.0,            3.0, -2.0)
T_kombi = transformationsmatrix(np.deg2rad(60), 3.0, -2.0)

# Teilaufgabe 2: Eckpunkt B in homogenen Koordinaten
B_hom = np.array([2.0, 0.0, 1.0])
print(f"T_rot   @ B: {np.round(T_rot   @ B_hom, 4)}")
print(f"T_trans @ B: {np.round(T_trans @ B_hom, 4)}")
print(f"T_kombi @ B: {np.round(T_kombi @ B_hom, 4)}")

# Teilaufgabe 3: Vergleich
T_produkt = T_trans @ T_rot
print(f"\nT_kombi == T_trans @ T_rot: {np.allclose(T_kombi, T_produkt)}")
print("T_kombi:")
print(np.round(T_kombi, 4))
print("T_trans @ T_rot:")
print(np.round(T_produkt, 4))
```

$\mathbf{T}_\text{kombi}$ und $\mathbf{T}_\text{trans} \cdot \mathbf{T}_\text{rot}$
sind identisch. Das zeigt, dass die Transformationsmatrix mit einem
einzigen Funktionsaufruf genau das Gleiche liefert wie die explizite
Verkettung: zuerst drehen, dann verschieben. Die rechte Spalte von
$\mathbf{T}$ kodiert immer die Verschiebung, die nach der Drehung
angewendet wird.
````

## Aufgabe 2: Das Dreieck in vier Transformationen

Wir wenden vier verschiedene Transformationen auf das Dreieck an und
stellen alle Ergebnisse in einer Abbildung dar.

```{admonition} Aufgabe 2
:class: tip
1. Definieren Sie vier Transformationsmatrizen:
   - $\mathbf{T}_1$: Drehung $90°$, Verschiebung $(0, 0)^\top$
   - $\mathbf{T}_2$: Drehung $0°$, Verschiebung $(3, 0)^\top$
   - $\mathbf{T}_3$: Drehung $45°$, Verschiebung $(-2, 2)^\top$
   - $\mathbf{T}_4$: Drehung $180°$, Verschiebung $(1, -1)^\top$
2. Wenden Sie jede Transformation auf das Dreieck an und zeichnen Sie
   alle vier Ergebnisse zusammen mit dem Original in einer Abbildung.
   Verwenden Sie für jedes Dreieck eine andere Farbe und eine Legende.
3. Welches der vier transformierten Dreiecke ist kongruent zum Original,
   und welche Transformationen ändern die Form oder Größe des Dreiecks?
   Begründen Sie kurz ohne Code.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon as MplPolygon

dreieck = np.array([[0.0, 0.0], [2.0, 0.0], [1.0, 2.0]])

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx], [s, c, ty], [0, 0, 1]])

def transformieren(T, punkte):
    einsen = np.ones((punkte.shape[0], 1))
    hom = np.hstack([punkte, einsen])
    return (T @ hom.T).T[:, :2]

transformationen = [
    (transformationsmatrix(np.deg2rad(90),  0.0,  0.0), 'T1: Drehen 90°',           'red'),
    (transformationsmatrix(0.0,             3.0,  0.0), 'T2: Verschieben (3,0)',     'green'),
    (transformationsmatrix(np.deg2rad(45), -2.0,  2.0), 'T3: Drehen 45° + (-2, 2)', 'orange'),
    (transformationsmatrix(np.deg2rad(180), 1.0, -1.0), 'T4: Drehen 180° + (1,-1)', 'purple'),
]

fig, ax = plt.subplots(figsize=(7, 7))
ax.add_patch(MplPolygon(dreieck, closed=True, fill=False,
                         edgecolor='blue', linewidth=2.5, label='Original'))

for T, label, farbe in transformationen:
    d = transformieren(T, dreieck)
    ax.add_patch(MplPolygon(d, closed=True, fill=False,
                             edgecolor=farbe, linewidth=1.8, label=label))

ax.set_xlim(-4, 6); ax.set_ylim(-4, 5)
ax.set_aspect('equal'); ax.grid(True); ax.legend(loc='upper right')
ax.set_title('Vier Transformationen im Vergleich')
plt.show()
```

Alle vier Dreiecke sind kongruent zum Original: homogene Transformationen
aus Drehung und Translation ändern weder Form noch Größe, weil die
Drehmatrix orthogonal ist ($\det(\mathbf{R}) = 1$) und Translation
Abstände erhält. Skalierung würde die Form verändern, ist in
$\mathbf{T}$ jedoch nicht enthalten.
````

## Aufgabe 3: Reihenfolge der Verkettung

Die Reihenfolge zweier Transformationen ist im Allgemeinen nicht egal.
Wir untersuchen das systematisch.

```{admonition} Aufgabe 3
:class: tip
1. Definieren Sie:
   - $\mathbf{T}_A$: Drehung $45°$, keine Verschiebung.
   - $\mathbf{T}_B$: keine Drehung, Verschiebung $(2, 1)^\top$.
2. Berechnen Sie $\mathbf{T}_{AB} = \mathbf{T}_B \cdot \mathbf{T}_A$
   (erst drehen, dann verschieben) und $\mathbf{T}_{BA} = \mathbf{T}_A
   \cdot \mathbf{T}_B$ (erst verschieben, dann drehen). Sind die Matrizen
   gleich?
3. Wenden Sie beide auf Eckpunkt C $= (1, 2)^\top$ an. Berechnen Sie den
   euklidischen Abstand zwischen den beiden Ergebnissen mit
   `np.linalg.norm`. Wie weit liegen die Endpunkte auseinander?
4. Überlegen Sie ohne Code: Für welchen Spezialfall wäre die Reihenfolge
   egal, d.h. $\mathbf{T}_B \cdot \mathbf{T}_A = \mathbf{T}_A \cdot
   \mathbf{T}_B$?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx], [s, c, ty], [0, 0, 1]])

T_A = transformationsmatrix(np.deg2rad(45), 0.0, 0.0)
T_B = transformationsmatrix(0.0,            2.0, 1.0)

T_AB = T_B @ T_A   # erst drehen, dann verschieben
T_BA = T_A @ T_B   # erst verschieben, dann drehen

print("T_AB (erst A, dann B):")
print(np.round(T_AB, 4))
print("\nT_BA (erst B, dann A):")
print(np.round(T_BA, 4))
print(f"\nGleich: {np.allclose(T_AB, T_BA)}")

C_hom = np.array([1.0, 2.0, 1.0])
C_AB = T_AB @ C_hom
C_BA = T_BA @ C_hom
abstand = np.linalg.norm(C_AB[:2] - C_BA[:2])
print(f"\nErgebnis T_AB @ C: {np.round(C_AB[:2], 4)}")
print(f"Ergebnis T_BA @ C: {np.round(C_BA[:2], 4)}")
print(f"Abstand:           {abstand:.4f}")
```

Die Matrizen sind verschieden. Zu Teilaufgabe 4: Die Reihenfolge ist
egal, wenn entweder keine Drehung vorhanden ist ($\varphi = 0$), oder
die Verschiebung null ist ($\vec{t} = \vec{0}$), oder wenn die Drehung
um $0°$ oder $360°$ beträgt. Im Allgemeinen gilt: Nur wenn beide
Transformationen dieselbe Achse betreffen und eine reine Translation
oder reine Rotation vorliegt, kommutieren sie.
````

## Aufgabe 4: Greifarm-Simulation

Ein vereinfachter Robotergreifer besteht aus zwei Segmenten. Segment 1
beginnt im Ursprung und zeigt entlang der $x$-Achse mit Länge $l_1 = 3$.
Am Ende von Segment 1 setzt Segment 2 mit Länge $l_2 = 2$ an. Wir
berechnen die Position der Greiferspitze für verschiedene Winkelstellungen.

```{admonition} Aufgabe 4
:class: tip
1. Definieren Sie eine homogene Transformationsmatrix für ein Segment der
   Länge $L$, das im Ursprung startet, zunächst um den Winkel $\varphi$
   gedreht wird und sich dann um $L$ entlang seiner **lokalen** $x$-Achse
   verschiebt. Nennen Sie diese Matrix `T_segment(phi, L)`.
2. Definieren Sie für Segment 1 die Transformation
   $\mathbf{T}_1 = \text{T\_segment}(\alpha, l_1)$ mit
   $\alpha = 30°$, $l_1 = 3$. Berechnen Sie die Position des
   Ellenbogengelenks (Ende von Segment 1), indem Sie $\mathbf{T}_1$ auf den
   Ursprung anwenden.
3. Definieren Sie für Segment 2 die Transformation
   $\mathbf{T}_2 = \text{T\_segment}(\beta, l_2)$ mit
   $\beta = -45°$, $l_2 = 2$. Die Gesamttransformation von Ursprung zur
   Greiferspitze ist dann $\mathbf{T}_\text{ges} = \mathbf{T}_1 \cdot \mathbf{T}_2$.
   Berechnen Sie die Position der Greiferspitze.
4. Zeichnen Sie den Greifarm: Ursprung, Ellenbogengelenk und
   Greiferspitze als Punkte, verbunden durch Linien. Beschriften Sie
   die drei Punkte.
5. Variieren Sie $\alpha$ von $0°$ bis $180°$ in 13 Schritten und
   behalten Sie $\beta = -45°$ fest. Zeichnen Sie für jeden Schritt
   die Greiferspitze als kleinen Punkt. Welche geometrische Figur
   beschreiben die Greiferspitzen? Begründen Sie in einem Satz ohne Code.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np
import matplotlib.pyplot as plt

# Homogene Transformation für ein Segment:
# 1) Drehung um phi im Ursprung
# 2) Verschiebung um L entlang der lokalen x-Achse
def T_segment(phi, L):
    c, s = np.cos(phi), np.sin(phi)
    # Verschiebung entlang lokaler x-Achse: R(phi) @ (L, 0)^T = (c*L, s*L)
    tx = c * L
    ty = s * L
    return np.array([
        [c, -s, tx],
        [s,  c, ty],
        [0,  0,  1],
    ])

l1, l2 = 3.0, 2.0
alpha  = np.deg2rad(30)
beta   = np.deg2rad(-45)

# Teilaufgabe 2: Segment 1
T1 = T_segment(alpha, l1)

ursprung = np.array([0.0, 0.0, 1.0])
ellenbogen = T1 @ ursprung   # Ende von Segment 1

# Teilaufgabe 3: Segment 2 relativ zu Segment 1
T2 = T_segment(beta, l2)     # im lokalen Ellenbogen-Koordinatensystem
T_ges = T1 @ T2              # Gesamttransformation Ursprung -> Greifer
greifer = T_ges @ ursprung

print(f"Ursprung:      (0.000, 0.000)")
print(f"Ellenbogen:    ({ellenbogen[0]:.3f}, {ellenbogen[1]:.3f})")
print(f"Greiferspitze: ({greifer[0]:.3f}, {greifer[1]:.3f})")

# Teilaufgabe 4: Arm zeichnen
fig, ax = plt.subplots(figsize=(6, 6))

punkte = np.array([
    [0.0,           0.0],
    [ellenbogen[0], ellenbogen[1]],
    [greifer[0],    greifer[1]],
])
ax.plot(punkte[:, 0], punkte[:, 1], 'o-', color='blue',
        linewidth=2.5, markersize=8)

for name, p in zip(['Ursprung', 'Ellenbogen', 'Greiferspitze'], punkte):
    ax.annotate(name, p, textcoords='offset points',
                xytext=(8, 4), fontsize=10)

# Teilaufgabe 5: Trajektorie der Greiferspitze bei variierendem alpha
alphas = np.linspace(0, np.pi, 13)
for a in alphas:
    T1_var = T_segment(a, l1)
    T_ges_var = T1_var @ T2
    g = T_ges_var @ ursprung
    ax.plot(g[0], g[1], 'o', color='red', markersize=5)

ax.set_xlim(-6, 6); ax.set_ylim(-2, 7)
ax.set_aspect('equal'); ax.grid(True)
ax.set_title('Greifarm: Segmente und Trajektorie der Greiferspitze')
ax.axhline(0, color='gray', linewidth=0.8)
ax.axvline(0, color='gray', linewidth=0.8)
plt.show()
```

Die homogene Matrix `T_segment(phi, L)` besteht aus der Drehmatrix
$\mathbf{R}(\varphi)$ und einer Verschiebung $R(\varphi)\,(L, 0)^\top$,
also einer Verschiebung entlang der lokalen $x$-Achse des jeweiligen
Segments.

Zu Teilaufgabe 5: Die Greiferspitzen beschreiben einen Kreisbogen um den
Ursprung mit konstantem Radius. Der Abstand der Greiferspitze vom Ursprung
ist
$$
r = \bigl\lVert\,l_1 (1,0)^\top + l_2(\cos\beta, \sin\beta)^\top\bigr\rVert,
$$
der nur von $l_1$, $l_2$ und dem festen Winkel $\beta$ abhängt, nicht aber
von $\alpha$. Daher liegen alle Greiferspitzenpunkte auf einem Kreis.
````
