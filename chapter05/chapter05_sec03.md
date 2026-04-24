---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 5.3 Homogene Koordinaten

In Abschnitt 5.1 haben wir gelernt, wie wir ein Objekt mit einer einzigen
Matrixmultiplikation drehen. In der Praxis wollen wir ein Bauteil aber nicht
nur drehen, sondern auch verschieben: Ein Roboterarm dreht sein Greifwerkzeug
und bewegt es gleichzeitig an eine neue Position. Mit gewöhnlichen Matrizen
lässt sich beides nicht in einem einzigen Schritt beschreiben. Homogene
Koordinaten lösen dieses Problem elegant.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können erklären, warum Translation nicht als gewöhnliche
  Matrixmultiplikation darstellbar ist.
* [ ] Sie können einen Punkt in **homogene Koordinaten** umwandeln und
  zurück.
* [ ] Sie können die homogene Transformationsmatrix $\mathbf{T}$ aus
  Rotation und Translation aufbauen.
* [ ] Sie können $\mathbf{T}$ auf ein Objekt (eine Punktemenge) anwenden
  und das Ergebnis visualisieren.
* [ ] Sie können zwei Transformationen durch Matrizenmultiplikation
  verketten und kennen die Bedeutung der Reihenfolge.
```

+++

## Warum Translation keine Matrixmultiplikation ist

Wir wollen Eckpunkt B unseres Dreiecks von $(2, 0)$ um den Vektor
$\vec{t} = (1, 3)^\top$ verschieben. Das Ergebnis soll $(3, 3)$ sein.
Schauen wir, ob eine $2 \times 2$-Matrix das leisten kann:

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon as MplPolygon

# --- Unser Dreieck aus Abschnitt 5.1 ---
dreieck = np.array([
    [0.0, 0.0],   # Eckpunkt A
    [2.0, 0.0],   # Eckpunkt B
    [1.0, 2.0],   # Eckpunkt C
])

# --- Verschiebung als Addition ---
# Eine Verschiebung um den Vektor t addiert t zu jedem Punkt.
# Das ist KEINE Matrixmultiplikation, sondern eine Addition.
t = np.array([1.0, 3.0])           # Verschiebungsvektor
dreieck_verschoben = dreieck + t   # Broadcasting: t wird zu jeder Zeile addiert

print("Original:")
print(dreieck)
print("\nVerschoben um t = (1, 3):")
print(dreieck_verschoben)

# --- Darstellung ---
fig, ax = plt.subplots(figsize=(5, 5))
ax.add_patch(MplPolygon(dreieck, closed=True, fill=False,
                         edgecolor='blue', linewidth=2, label='Original'))
ax.add_patch(MplPolygon(dreieck_verschoben, closed=True, fill=False,
                         edgecolor='red', linewidth=2, label='Verschoben'))
ax.set_xlim(-1, 5); ax.set_ylim(-1, 6)
ax.set_aspect('equal'); ax.grid(True); ax.legend()
ax.set_title('Verschiebung als Addition')
plt.show()
```

Die Verschiebung funktioniert als Addition. Warum ist das ein Problem?
Weil wir Drehung und Verschiebung dann nicht kombinieren können, ohne zwei
getrennte Operationen auszuführen: erst multiplizieren, dann addieren.
Im Robotik- und Simulationskontext, wo Dutzende solcher Transformationen
verkettet werden, ist das unpraktisch.

Wir rechnen kurz nach, warum eine $2 \times 2$-Matrix die Translation nicht
ersetzen kann. Gesucht ist eine Matrix $\mathbf{M}$, sodass
$\mathbf{M} \cdot \vec{p} = \vec{p} + \vec{t}$ für alle Punkte $\vec{p}$
gilt. Für $\vec{p} = (0, 0)^\top$ müsste gelten:

\begin{equation*}
\mathbf{M} \begin{pmatrix} 0 \\ 0 \end{pmatrix}
= \begin{pmatrix} 0 \\ 0 \end{pmatrix} + \begin{pmatrix} t_x \\ t_y \end{pmatrix}
= \begin{pmatrix} t_x \\ t_y \end{pmatrix}.
\end{equation*}

Aber jede Matrix multipliziert mit dem Nullvektor ergibt den Nullvektor:
$\mathbf{M} \cdot \vec{0} = \vec{0}$. Eine solche Matrix kann also gar nicht
existieren. Translation ist eine **affine Abbildung**, keine lineare, und
deshalb nicht als $2 \times 2$-Matrixmultiplikation darstellbar.

```{admonition} Mini-Übung
:class: tip
1. Verschieben Sie das Dreieck um $\vec{t} = (-2, 1)^\top$ und zeichnen
   Sie Original und verschobenes Dreieck.
2. Führen Sie danach eine 45°-Drehung des bereits verschobenen Dreiecks
   durch. Welche zwei Python-Operationen brauchen Sie dafür? Wie viele
   Schritte sind das insgesamt, um erst zu drehen und dann zu verschieben?
3. Warum kann eine $2 \times 2$-Matrix den Nullpunkt nicht verschieben?
   Begründen Sie in einem Satz ohne Code.
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

def drehmatrix_2d(phi):
    return np.array([[np.cos(phi), -np.sin(phi)],
                     [np.sin(phi),  np.cos(phi)]])

# Teilaufgabe 1: Verschiebung
t = np.array([-2.0, 1.0])
dreieck_verschoben = dreieck + t

# Teilaufgabe 2: danach Drehung
R = drehmatrix_2d(np.deg2rad(45))
dreieck_gedreht = (R @ dreieck_verschoben.T).T   # Schritt 1: Multiplikation
                                                  # Schritt 2: Addition (war oben)
# Insgesamt zwei Schritte: erst + t, dann @ R

fig, ax = plt.subplots(figsize=(5, 5))
ax.add_patch(MplPolygon(dreieck,           closed=True, fill=False,
                         edgecolor='blue',   linewidth=2, label='Original'))
ax.add_patch(MplPolygon(dreieck_verschoben, closed=True, fill=False,
                         edgecolor='red',    linewidth=2, label='Verschoben'))
ax.add_patch(MplPolygon(dreieck_gedreht,    closed=True, fill=False,
                         edgecolor='green',  linewidth=2, label='Verschoben + gedreht'))
ax.set_xlim(-4, 4); ax.set_ylim(-1, 5)
ax.set_aspect('equal'); ax.grid(True); ax.legend()
plt.show()
```

Zu Teilaufgabe 3: Das Matrixprodukt $\mathbf{M} \cdot \vec{0}$ ergibt
immer $\vec{0}$, weil jede Linearkombination der Nullzeilen gleich null
ist. Der Ursprung kann also durch keine lineare Abbildung verschoben werden.
````

## Homogene Koordinaten und die Transformationsmatrix

Die Idee der **homogenen Koordinaten** ist einfach: Wir fügen jedem Punkt
eine künstliche dritte Koordinate mit dem Wert $1$ hinzu. Aus dem 2D-Punkt
$(x, y)^\top$ wird der 3D-Vektor $(x, y, 1)^\top$. In diesem erweiterten
Raum lassen sich Drehung und Verschiebung zu einer einzigen $3 \times 3$-Matrix
zusammenfassen.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon as MplPolygon

dreieck = np.array([[0.0, 0.0], [2.0, 0.0], [1.0, 2.0]])

# --- Homogene Koordinaten: dritte Spalte mit Einsen anhängen ---
# np.ones((3, 1)) erzeugt eine (3, 1)-Spalte mit Einsen.
# np.hstack fügt sie rechts an das (3, 2)-Array an -> (3, 3)-Array.
# Jede Zeile: [x, y, 1]
einsen = np.ones((dreieck.shape[0], 1))
dreieck_hom = np.hstack([dreieck, einsen])

print("Dreieck in homogenen Koordinaten:")
print(dreieck_hom)

# --- Homogene Transformationsmatrix T (Drehung + Verschiebung) ---
# Aufbau:
#   [ cos(phi)  -sin(phi)  tx ]
#   [ sin(phi)   cos(phi)  ty ]
#   [ 0          0         1  ]
# Die obere linke 2x2-Teilmatrix ist die Drehmatrix R.
# Die rechte Spalte (tx, ty) ist der Verschiebungsvektor.
# Die unterste Zeile (0, 0, 1) sorgt dafür, dass die 1 erhalten bleibt.

def transformationsmatrix(phi, tx, ty):
    """Homogene 2D-Transformationsmatrix: Drehung um phi, dann Verschiebung (tx, ty)."""
    c, s = np.cos(phi), np.sin(phi)
    return np.array([
        [c, -s, tx],
        [s,  c, ty],
        [0,  0,  1],
    ])

# --- Beispiel: 45° drehen, dann um (1, 2) verschieben ---
phi = np.deg2rad(45)
tx, ty = 1.0, 2.0
T = transformationsmatrix(phi, tx, ty)

print("\nTransformationsmatrix T (45°, t=(1,2)):")
print(np.round(T, 4))

# --- Transformation anwenden: T @ dreieck_hom.T ---
# dreieck_hom.T hat Form (3, 3): drei Zeilen (x, y, 1), drei Spalten (Punkte)
# T @ dreieck_hom.T hat Form (3, 3): transformierte Koordinaten
# .T am Ende: zurück in Form (3, 3) mit Punkten als Zeilen
dreieck_transformiert_hom = (T @ dreieck_hom.T).T

# --- Zurück in euklidische Koordinaten: erste zwei Spalten nehmen ---
dreieck_transformiert = dreieck_transformiert_hom[:, :2]

print("\nTransformiertes Dreieck (euklidisch):")
print(np.round(dreieck_transformiert, 4))

# --- Darstellung ---
fig, ax = plt.subplots(figsize=(5, 5))
ax.add_patch(MplPolygon(dreieck,              closed=True, fill=False,
                         edgecolor='blue',      linewidth=2, label='Original'))
ax.add_patch(MplPolygon(dreieck_transformiert, closed=True, fill=False,
                         edgecolor='red',       linewidth=2,
                         label='Gedreht 45° + verschoben (1, 2)'))
ax.set_xlim(-3, 4); ax.set_ylim(-1, 5)
ax.set_aspect('equal'); ax.grid(True); ax.legend()
ax.set_title('Homogene Transformation: Drehen + Verschieben')
plt.show()
```

Mit einer einzigen Matrix-Multiplikation haben wir das Dreieck gedreht und
verschoben. Die Struktur der Transformationsmatrix lässt sich schematisch
so darstellen:

\begin{equation*}
\mathbf{T} =
\begin{pmatrix}
  \cos\varphi & -\sin\varphi & t_x \\
  \sin\varphi &  \cos\varphi & t_y \\
  0           &  0           & 1
\end{pmatrix} =
\left(
\begin{array}{c|c}
  \mathbf{R}(\varphi) & \vec{t} \\
  \hline
  \vec{0}^\top        & 1
\end{array}
\right).
\end{equation*}

Wir rechnen die Transformation von Eckpunkt B $= (2, 0)^\top$ von Hand
durch. Mit $\cos 45° = \sin 45° = \frac{\sqrt{2}}{2} \approx 0.707$:

\begin{equation*}
\mathbf{T} \cdot \tilde{p}_B
= \begin{pmatrix} 0.707 & -0.707 & 1 \\ 0.707 & 0.707 & 2 \\ 0 & 0 & 1 \end{pmatrix}
  \begin{pmatrix} 2 \\ 0 \\ 1 \end{pmatrix}
= \begin{pmatrix} 0.707 \cdot 2 + 1 \\ 0.707 \cdot 2 + 2 \\ 1 \end{pmatrix}
= \begin{pmatrix} 2.414 \\ 3.414 \\ 1 \end{pmatrix}.
\end{equation*}

Die dritte Koordinate bleibt $1$. Die euklidischen Koordinaten des
transformierten Punktes sind $(2.414, 3.414)^\top$.

```{admonition} Homogene Koordinaten in der Robotik
:class: note
In der industriellen Robotik wird jede Stellung eines Gelenks durch eine
homogene $4 \times 4$-Transformationsmatrix beschrieben, die Drehung und
Verschiebung im 3D-Raum kombiniert. Die Gesamtstellung eines Roboterarms
ergibt sich als Produkt all dieser Matrizen. Genau dasselbe Prinzip
wenden wir hier in 2D an.
```

```{admonition} Mini-Übung
:class: tip
1. Erstellen Sie eine Transformationsmatrix für eine reine Verschiebung
   um $(3, -1)^\top$ ohne Drehung ($\varphi = 0$). Wie sieht die Matrix
   aus? Wenden Sie sie auf Eckpunkt A an und überprüfen Sie das Ergebnis.
2. Erstellen Sie eine Transformationsmatrix für eine reine Drehung um
   $90°$ ohne Verschiebung. Vergleichen Sie den oberen linken $2 \times 2$
   Block mit `drehmatrix_2d(np.deg2rad(90))` aus Abschnitt 5.1.
3. Was steht in der dritten Zeile von $\mathbf{T}$, und warum muss sie
   immer $(0, 0, 1)$ lauten? Begründen Sie in einem Satz ohne Code.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

dreieck = np.array([[0.0, 0.0], [2.0, 0.0], [1.0, 2.0]])
einsen  = np.ones((3, 1))
dreieck_hom = np.hstack([dreieck, einsen])

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx],
                     [s,  c, ty],
                     [0,  0,  1]])

# Teilaufgabe 1: reine Verschiebung
T_verschiebung = transformationsmatrix(0, 3.0, -1.0)
print("T (reine Verschiebung):")
print(T_verschiebung)
punkt_A_hom = np.array([0.0, 0.0, 1.0])
print(f"Eckpunkt A transformiert: {T_verschiebung @ punkt_A_hom}")

# Teilaufgabe 2: reine Drehung
T_drehung = transformationsmatrix(np.deg2rad(90), 0, 0)
print("\nT (reine Drehung 90°):")
print(np.round(T_drehung, 4))
print("Oberer linker 2x2-Block:")
print(np.round(T_drehung[:2, :2], 4))
```

Zu Teilaufgabe 1: Die Matrix hat $1$ auf der Diagonalen und $(3, -1)$
in der rechten Spalte. Eckpunkt A landet bei $(3, -1)$, weil er einfach
um den Verschiebungsvektor verschoben wird. Zu Teilaufgabe 3: Die dritte
Zeile $(0, 0, 1)$ stellt sicher, dass die künstliche dritte Koordinate
jedes Punktes nach der Transformation wieder $1$ bleibt, was Voraussetzung
für das korrekte Ablesen der euklidischen Koordinaten ist.
````

## Verkettung von Transformationen

In der Praxis werden mehrere Transformationen hintereinander ausgeführt.
Ein Greifarm dreht ein Bauteil, verschiebt es, dreht es nochmals. Mit
homogenen Koordinaten ergibt die Verkettung zweier Transformationen $\mathbf{T}_1$
(zuerst) und $\mathbf{T}_2$ (danach) das Produkt $\mathbf{T}_2 \cdot \mathbf{T}_1$.
Die Reihenfolge ist entscheidend: die zuerst auszuführende Transformation
steht rechts.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Polygon as MplPolygon

dreieck = np.array([[0.0, 0.0], [2.0, 0.0], [1.0, 2.0]])
einsen  = np.ones((3, 1))
dreieck_hom = np.hstack([dreieck, einsen])

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx],
                     [s,  c, ty],
                     [0,  0,  1]])

# --- Zwei Transformationen definieren ---
# T1: Drehung um 30° (keine Verschiebung)
# T2: Verschiebung um (2, 1) (keine Drehung)
T1 = transformationsmatrix(np.deg2rad(30), 0.0, 0.0)
T2 = transformationsmatrix(0.0,            2.0, 1.0)

# --- Verkettung: erst T1 (Drehung), dann T2 (Verschiebung) ---
# T_ges = T2 @ T1 bedeutet: zuerst T1 anwenden, dann T2
# Die Reihenfolge ist NICHT egal: T2 @ T1 != T1 @ T2
T_ges_A = T2 @ T1   # erst drehen, dann verschieben
T_ges_B = T1 @ T2   # erst verschieben, dann drehen

# --- Ergebnisse berechnen ---
dreieck_A = (T_ges_A @ dreieck_hom.T).T[:, :2]   # erst drehen, dann verschieben
dreieck_B = (T_ges_B @ dreieck_hom.T).T[:, :2]   # erst verschieben, dann drehen

print("Gesamtmatrix (erst drehen, dann verschieben):")
print(np.round(T_ges_A, 4))
print("\nGesamtmatrix (erst verschieben, dann drehen):")
print(np.round(T_ges_B, 4))
print(f"\nGleiche Matrizen: {np.allclose(T_ges_A, T_ges_B)}")

# --- Darstellung: drei Dreiecke im Vergleich ---
fig, ax = plt.subplots(figsize=(6, 6))
ax.add_patch(MplPolygon(dreieck,   closed=True, fill=False,
                         edgecolor='blue',   linewidth=2, label='Original'))
ax.add_patch(MplPolygon(dreieck_A, closed=True, fill=False,
                         edgecolor='red',    linewidth=2,
                         label='Erst drehen, dann verschieben'))
ax.add_patch(MplPolygon(dreieck_B, closed=True, fill=False,
                         edgecolor='green',  linewidth=2, linestyle='--',
                         label='Erst verschieben, dann drehen'))
ax.set_xlim(-3, 5); ax.set_ylim(-2, 5)
ax.set_aspect('equal'); ax.grid(True); ax.legend(loc='upper left')
ax.set_title('Reihenfolge der Transformationen')
plt.show()
```

Die beiden Ergebnisse landen an verschiedenen Positionen. Das ist intuitiv
nachvollziehbar: Wenn wir zuerst drehen und dann verschieben, wandert das
gedrehte Objekt in eine feste Richtung. Drehen wir dagegen nach der
Verschiebung, dreht sich das Objekt um den Ursprung und seine neue Position
hängt davon ab, wie weit es bereits vom Ursprung entfernt war.

Wir rechnen nach, wohin Eckpunkt B $= (2, 0)^\top$ bei Variante A landet.
Mit $\cos 30° \approx 0.866$ und $\sin 30° = 0.5$:

\begin{equation*}
\vec{p}^{(1)} = \mathbf{T}_1 \cdot \tilde{p}_B
= \begin{pmatrix} 0.866 & -0.5 & 0 \\ 0.5 & 0.866 & 0 \\ 0 & 0 & 1 \end{pmatrix}
  \begin{pmatrix} 2 \\ 0 \\ 1 \end{pmatrix}
= \begin{pmatrix} 1.732 \\ 1.000 \\ 1 \end{pmatrix},
\qquad
\vec{p}^{(2)} = \mathbf{T}_2 \cdot \vec{p}^{(1)}
= \begin{pmatrix} 1 & 0 & 2 \\ 0 & 1 & 1 \\ 0 & 0 & 1 \end{pmatrix}
  \begin{pmatrix} 1.732 \\ 1.000 \\ 1 \end{pmatrix}
= \begin{pmatrix} 3.732 \\ 2.000 \\ 1 \end{pmatrix}.
\end{equation*}

Dasselbe Ergebnis liefert direkt $\mathbf{T}_\text{ges} \cdot \tilde{p}_B$,
ohne Zwischenschritt.

```{admonition} Mini-Übung
:class: tip
1. Berechnen Sie die Gesamtmatrix $\mathbf{T}_3 = \mathbf{T}_1 \cdot \mathbf{T}_1$
   (dieselbe Drehung zweimal hintereinander). Was ergibt sich geometrisch?
   Vergleichen Sie mit `transformationsmatrix(np.deg2rad(60), 0, 0)`.
2. Bestimmen Sie die **inverse Transformation** von $\mathbf{T}_\text{ges\_A}$
   mit `np.linalg.inv`. Wenden Sie sie auf `dreieck_A` an und überprüfen Sie
   mit `np.allclose`, dass Sie das Originaldreieck zurückbekommen.
3. Überlegen Sie ohne Code: Ein Bauteil soll um seinen eigenen Schwerpunkt
   bei $(1, 1)^\top$ gedreht werden (nicht um den Ursprung). Welche drei
   Transformationen müssen in welcher Reihenfolge verkettet werden?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

dreieck = np.array([[0.0, 0.0], [2.0, 0.0], [1.0, 2.0]])
einsen  = np.ones((3, 1))
dreieck_hom = np.hstack([dreieck, einsen])

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx],
                     [s,  c, ty],
                     [0,  0,  1]])

T1     = transformationsmatrix(np.deg2rad(30), 0.0, 0.0)
T2     = transformationsmatrix(0.0,            2.0, 1.0)
T_ges_A = T2 @ T1

# Teilaufgabe 1: T1 zweimal
T3    = T1 @ T1
T60   = transformationsmatrix(np.deg2rad(60), 0, 0)
print("T1 @ T1:")
print(np.round(T3, 4))
print(f"Gleich wie T(60°): {np.allclose(T3, T60)}")

# Teilaufgabe 2: Inverse
T_inv = np.linalg.inv(T_ges_A)
dreieck_A_hom = (T_ges_A @ dreieck_hom.T).T
dreieck_zurueck = (T_inv @ dreieck_A_hom.T).T[:, :2]
dreieck_zurueck = (T_inv @ dreieck_A_hom.T).T[:, :2]
print(f"\nOriginal zurück: {np.allclose(dreieck, dreieck_zurueck)}")
```

Zu Teilaufgabe 1: Dieselbe Drehung zweimal ergibt eine Drehung um den
doppelten Winkel, also um 60°. Die Matrizen sind identisch.
Zu Teilaufgabe 3: Zuerst den Schwerpunkt $(1, 1)$ in den Ursprung
verschieben ($\vec{t} = (-1, -1)^\top$), dann drehen, dann zurück
verschieben ($\vec{t} = (1, 1)^\top$). Als Gesamtmatrix:
$\mathbf{T}_\text{ges} = \mathbf{T}(0, 1, 1) \cdot \mathbf{R}(\varphi)
\cdot \mathbf{T}(0, -1, -1)$.
````

## Zusammenfassung und Ausblick

Homogene Koordinaten lösen das Problem, das eine gewöhnliche Matrixmultiplikation
nicht lösen kann: Translation lässt sich nicht als lineare Abbildung darstellen,
weil der Nullpunkt immer auf den Nullpunkt abgebildet wird. Durch das Anhängen
einer künstlichen Koordinate $1$ wird jede Kombination aus Drehung und
Verschiebung zu einer einzigen $3 \times 3$-Matrixmultiplikation. Mehrere
solcher Transformationen werden durch Matrizenprodukte verkettet, wobei die
Reihenfolge entscheidend ist.

Im nächsten Abschnitt wenden wir dieses Werkzeug auf eine typische
Aufgabe aus der Fahrzeugtechnik an: die Umrechnung von Sensor-Messdaten
aus dem fahrzeugfesten Koordinatensystem in das Weltkoordinatensystem, die
so genannte Koordinatentransformation zwischen bewegten Bezugssystemen.
