---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 5.5 Übungen zum Selbststudium

```{admonition} Warnung
:class: warning
Dieses Kapitel befindet sich derzeit im Umbau und wird rechtzeitig vor der Vorlesung im WiSe 2026/27 zur Verfügung stehen.
```

````{admonition} Übung 5.1 (✩)
:class: tip
Gegeben ist folgender Code:

```python
import numpy as np

def drehmatrix_2d(phi):
    return np.array([
        [np.cos(phi), -np.sin(phi)],
        [np.sin(phi),  np.cos(phi)],
    ])

R = drehmatrix_2d(np.deg2rad(180))
p = np.array([3.0, 4.0])
print(np.round(R @ p, 4))
print(np.round(R.T @ (R @ p), 4))
print(np.round(np.linalg.det(R), 4))
```

1. Sagen Sie vorher: Was gibt die erste `print`-Zeile aus? Überlegen Sie
   geometrisch, wohin eine 180°-Drehung den Punkt $(3, 4)^\top$ schickt.
2. Was gibt die zweite `print`-Zeile aus, und warum?
3. Was gibt die dritte `print`-Zeile aus? Was bedeutet dieses Ergebnis
   für die Eigenschaft der Drehmatrix?
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

def drehmatrix_2d(phi):
    return np.array([
        [np.cos(phi), -np.sin(phi)],
        [np.sin(phi),  np.cos(phi)],
    ])

R = drehmatrix_2d(np.deg2rad(180))
p = np.array([3.0, 4.0])
print(np.round(R @ p, 4))            # [-3. -4.]
print(np.round(R.T @ (R @ p), 4))   # [3. 4.]
print(np.round(np.linalg.det(R), 4)) # 1.0
```

Zu Frage 1: Eine 180°-Drehung spiegelt den Punkt am Ursprung. Aus
$(3, 4)^\top$ wird $(-3, -4)^\top$. Zu Frage 2: Die Transponierte macht
die Drehung rückgängig, weil $\mathbf{R}^\top \mathbf{R} = \mathbf{I}$
gilt. Das Ergebnis ist der ursprüngliche Punkt $(3, 4)^\top$. Zu Frage 3:
Die Determinante ist $1$, was bestätigt, dass $\mathbf{R}$ eine
orientierungserhaltende Isometrie ist.
````

````{admonition} Übung 5.2 (✩)
:class: tip
Gegeben ist folgender Code:

```python
import numpy as np

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx],
                     [s,  c, ty],
                     [0,  0,  1]])

T = transformationsmatrix(np.deg2rad(90), 2.0, -1.0)
p_hom = np.array([1.0, 0.0, 1.0])

print(np.round(T @ p_hom, 4))
print(np.round(T @ T @ p_hom, 4))
print(np.round(np.linalg.det(T), 4))
```

1. Sagen Sie vorher: Was gibt die erste `print`-Zeile aus? Berechnen Sie
   die Transformation von $(1, 0)^\top$ mit $\varphi = 90°$ und
   $\vec{t} = (2, -1)^\top$ von Hand: erst drehen, dann verschieben.
2. Was gibt die zweite `print`-Zeile aus? Überlegen Sie, welche kombinierte
   Transformation $\mathbf{T} \cdot \mathbf{T}$ darstellt.
3. Was ergibt die Determinante einer homogenen Transformationsmatrix stets,
   und warum?
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

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx], [s, c, ty], [0, 0, 1]])

T = transformationsmatrix(np.deg2rad(90), 2.0, -1.0)
p_hom = np.array([1.0, 0.0, 1.0])

print(np.round(T @ p_hom, 4))       # [2. 0. 1.] -> (1,0) -> (0,1) + (2,-1) = (2.0, 0.0)
print(np.round(T @ T @ p_hom, 4))   # zweimal dieselbe Transformation
print(np.round(np.linalg.det(T), 4)) # 1.0
```

Zu Frage 1: $(1, 0)^\top$ wird durch $90°$ zu $(0, 1)^\top$, dann
verschoben um $(2, -1)^\top$: Ergebnis $(2, 0)^\top$. Zu Frage 2:
$\mathbf{T} \cdot \mathbf{T}$ entspricht einer Drehung um $180°$ mit
einer kombinierten Verschiebung. Zu Frage 3: Die Determinante ist stets
$1$, weil $\det(\mathbf{T}) = \det(\mathbf{R}) \cdot 1 = 1 \cdot 1 = 1$
(Entwicklung nach der letzten Zeile).
````

````{admonition} Übung 5.3 (✩)
:class: tip
Gegeben ist folgender Code:

```python
import numpy as np

def R_nicken(beta):
    return np.array([
        [ np.cos(beta), 0, np.sin(beta)],
        [            0, 1,            0],
        [-np.sin(beta), 0, np.cos(beta)],
    ])

def R_gieren(alpha):
    return np.array([
        [ np.cos(alpha), -np.sin(alpha), 0],
        [ np.sin(alpha),  np.cos(alpha), 0],
        [             0,              0, 1],
    ])

v = np.array([1.0, 0.0, 0.0])
alpha = np.deg2rad(90)
beta  = np.deg2rad(90)

print(np.round(R_nicken(beta) @ R_gieren(alpha) @ v, 4))
print(np.round(R_gieren(alpha) @ R_nicken(beta) @ v, 4))
```

1. Sagen Sie für jede `print`-Zeile vorher: Was ist das Ergebnis? Führen
   Sie die Rechnung schrittweise durch: erst die innere Matrix auf $\vec{v}$,
   dann die äußere auf das Zwischenergebnis.
2. Sind beide Ergebnisse gleich? Was folgt daraus für die
   Matrizenmultiplikation?
3. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

def R_nicken(beta):
    return np.array([[ np.cos(beta), 0, np.sin(beta)],
                     [            0, 1,            0],
                     [-np.sin(beta), 0, np.cos(beta)]])

def R_gieren(alpha):
    return np.array([[ np.cos(alpha), -np.sin(alpha), 0],
                     [ np.sin(alpha),  np.cos(alpha), 0],
                     [             0,              0, 1]])

v = np.array([1.0, 0.0, 0.0])
alpha = np.deg2rad(90)
beta  = np.deg2rad(90)

print(np.round(R_nicken(beta) @ R_gieren(alpha) @ v, 4))  # [0. 0. -1.]
print(np.round(R_gieren(alpha) @ R_nicken(beta) @ v, 4))  # [0. 1.  0.]
```

Variante 1 (`R_nicken(beta) @ R_gieren(alpha) @ v`): Zuerst giert das System um
die z‑Achse: \((1,0,0)^\top \mapsto (0,1,0)^\top\). Anschließend nickt es um die
y‑Achse; der Vektor liegt auf der \(y\)-Achse und bleibt daher \((0,1,0)^\top\).

Variante 2 (`R_gieren(alpha) @ R_nicken(beta) @ v`): Zuerst nickt das System um
die y‑Achse: \((1,0,0)^\top \mapsto (0,0,-1)^\top\). Danach giert es um die
z‑Achse; ein Vektor auf der \(z\)-Achse bleibt unverändert \((0,0,-1)^\top\).

Die Ergebnisse sind verschieden, also ist die Matrizenmultiplikation (und damit
die Reihenfolge der Drehungen) nicht kommutativ.
````

````{admonition} Übung 5.4 (✩✩)
:class: tip
Ein Beschleunigungssensor ist in einem Fahrzeug verbaut und misst die
Beschleunigung im fahrzeugfesten Koordinatensystem. Der Sensor liefert
den Vektor $\vec{a}_\text{sensor} = (0.5, 0.0, -9.81)^\top\,\text{m/s}^2$
(Vorwärtsbeschleunigung, Querbeschleunigung, vertikale Komponente).

Das Fahrzeug ist gegenüber dem Weltkoordinatensystem um die $z$-Achse
(Gieren) um $\alpha = 45°$ gedreht und um die $y$-Achse (Nicken) um
$\beta = 5°$ geneigt (leichte Steigung).

1. Schreiben Sie eine Funktion `sensor_zu_welt(a_sensor, alpha, beta)`,
   die den Messvektor vom Sensor- in das Weltkoordinatensystem transformiert.
   Verwenden Sie die Kardan-Reihenfolge $\mathbf{R} = \mathbf{R}_y(\beta)
   \cdot \mathbf{R}_z(\alpha)$ (zuerst gieren, dann nicken).
2. Berechnen Sie $\vec{a}_\text{welt}$ für die gegebenen Werte und geben
   Sie das Ergebnis aus.
3. Überprüfen Sie: Die Länge des Beschleunigungsvektors muss vor und nach
   der Transformation gleich bleiben. Prüfen Sie das mit `np.linalg.norm`.
4. Warum bleibt die Länge erhalten? Begründen Sie in einem Satz ohne Code.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

def R_gieren(alpha):
    c, s = np.cos(alpha), np.sin(alpha)
    return np.array([[c, -s, 0], [s, c, 0], [0, 0, 1]])

def R_nicken(beta):
    c, s = np.cos(beta), np.sin(beta)
    return np.array([[c, 0, s], [0, 1, 0], [-s, 0, c]])

def sensor_zu_welt(a_sensor, alpha, beta):
    """Transformiert einen Vektor vom Sensor- ins Weltkoordinatensystem."""
    # Gesamtrotation: zuerst Gieren, dann Nicken
    R = R_nicken(beta) @ R_gieren(alpha)
    return R @ a_sensor

a_sensor = np.array([0.5, 0.0, -9.81])
alpha    = np.deg2rad(45)
beta     = np.deg2rad(5)

a_welt = sensor_zu_welt(a_sensor, alpha, beta)
print(f"Beschleunigung im Weltkoordinatensystem: {np.round(a_welt, 4)} m/s²")

# Längenerhalt prüfen
laenge_sensor = np.linalg.norm(a_sensor)
laenge_welt   = np.linalg.norm(a_welt)
print(f"Länge Sensor: {laenge_sensor:.4f} m/s²")
print(f"Länge Welt:   {laenge_welt:.4f} m/s²")
print(f"Gleich:       {np.allclose(laenge_sensor, laenge_welt)}")
```

Der Beschleunigungsvektor im Weltkoordinatensystem zeigt überwiegend in
die Richtung, die der globalen Fahrtrichtung und der Gravitationsachse
entspricht. Die Länge bleibt erhalten, weil Drehmatrizen orthogonal sind
und daher Abstände und Winkel invariant lassen.
````

````{admonition} Übung 5.5 (✩✩)
:class: tip
Wir untersuchen die Rückdrehung mit der Transponierten systematisch.

1. Schreiben Sie eine Funktion `drehe_und_pruefe(punkte, phi)`, die
   folgende Schritte ausführt:
   - Alle Punkte in `punkte` um den Winkel `phi` drehen.
   - Die Drehung mit $\mathbf{R}^\top$ rückgängig machen.
   - Prüfen, ob die Originalpunkte wiederhergestellt werden (`np.allclose`).
   - Den maximalen Fehler `np.max(np.abs(...))` ausgeben.
   Die Funktion soll `True` oder `False` zurückgeben.
2. Testen Sie die Funktion mit dem Dreieck aus Abschnitt 5.1 für die
   Winkel $15°$, $90°$, $137°$ und $270°$.
3. Schreiben Sie zusätzlich eine Funktion
   `ist_orthogonal(M, toleranz=1e-10)`, die prüft, ob eine gegebene
   Matrix orthogonal ist, und geben Sie für die vier Drehmatrizen
   aus Teilaufgabe 2 aus, ob sie diese Bedingung erfüllen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

dreieck = np.array([[0.0, 0.0], [2.0, 0.0], [1.0, 2.0]])

def drehmatrix_2d(phi):
    return np.array([[np.cos(phi), -np.sin(phi)],
                     [np.sin(phi),  np.cos(phi)]])

def drehe_und_pruefe(punkte, phi):
    """Dreht Punkte, macht Drehung rueckgaengig und prueft Uebereinstimmung."""
    R = drehmatrix_2d(phi)
    gedreht   = (R @ punkte.T).T
    zurueck   = (R.T @ gedreht.T).T
    fehler    = np.max(np.abs(punkte - zurueck))
    wiederhergestellt = np.allclose(punkte, zurueck)
    print(f"phi={np.rad2deg(phi):6.1f}°  max. Fehler={fehler:.2e}  "
          f"Wiederhergestellt: {wiederhergestellt}")
    return wiederhergestellt

def ist_orthogonal(M, toleranz=1e-10):
    """Prueft, ob M^T * M = I gilt."""
    return np.allclose(M.T @ M, np.eye(M.shape[0]), atol=toleranz)

print("Rückdrehungstest:")
for grad in [15, 90, 137, 270]:
    drehe_und_pruefe(dreieck, np.deg2rad(grad))

print("\nOrthogonalitaetstest:")
for grad in [15, 90, 137, 270]:
    R = drehmatrix_2d(np.deg2rad(grad))
    print(f"phi={grad:3d}°  orthogonal: {ist_orthogonal(R)}")
```

Für alle vier Winkel wird der Ausgangspunkt auf numerische Genauigkeit
wiederhergestellt (maximaler Fehler in der Größenordnung $10^{-16}$, also
im Bereich der Maschinengenauigkeit). Alle vier Matrizen bestehen den
Orthogonalitätstest.
````

````{admonition} Übung 5.6 (✩✩)
:class: tip
Ein CAD-Bauteil hat fünf Kontrollpunkte, die eine Kontur beschreiben:

```python
kontur = np.array([
    [0.0, 0.0],
    [4.0, 0.0],
    [4.0, 2.0],
    [2.0, 3.0],
    [0.0, 2.0],
])
```

1. Schreiben Sie eine Funktion `kontur_transformieren(kontur, phi, tx, ty)`,
   die die gesamte Kontur mit einer homogenen Transformationsmatrix
   transformiert und die transformierte Kontur zurückgibt.
2. Wenden Sie folgende drei Transformationen der Reihe nach an (jedes Mal
   auf das Ergebnis der vorherigen):
   - $\mathbf{T}_1$: Drehung $30°$, Verschiebung $(1, 0)^\top$
   - $\mathbf{T}_2$: Drehung $0°$, Verschiebung $(0, 2)^\top$
   - $\mathbf{T}_3$: Drehung $-15°$, Verschiebung $(0, 0)^\top$
3. Stellen Sie Original und alle drei Zwischenergebnisse in einer Abbildung
   dar. Verbinden Sie die Kontrollpunkte jeder Kontur mit einer
   geschlossenen Linie (kein `MplPolygon` nötig: `ax.plot` mit dem ersten
   Punkt am Ende wiederholen).
4. Berechnen Sie auch die Gesamttransformation als einzige Matrix
   $\mathbf{T}_\text{ges} = \mathbf{T}_3 \cdot \mathbf{T}_2 \cdot
   \mathbf{T}_1$ und wenden Sie sie direkt auf die Originalkontur an.
   Stimmt das Ergebnis mit dem dreifach transformierten Ergebnis überein?
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

kontur = np.array([[0.0, 0.0], [4.0, 0.0], [4.0, 2.0],
                   [2.0, 3.0], [0.0, 2.0]])

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx], [s, c, ty], [0, 0, 1]])

def kontur_transformieren(kontur, phi, tx, ty):
    """Transformiert alle Kontrollpunkte mit einer homogenen Matrix."""
    T     = transformationsmatrix(phi, tx, ty)
    einsen = np.ones((kontur.shape[0], 1))
    hom   = np.hstack([kontur, einsen])
    return (T @ hom.T).T[:, :2]

T1 = transformationsmatrix(np.deg2rad(30),   1.0, 0.0)
T2 = transformationsmatrix(0.0,              0.0, 2.0)
T3 = transformationsmatrix(np.deg2rad(-15),  0.0, 0.0)

k1 = kontur_transformieren(kontur, np.deg2rad(30),   1.0, 0.0)
k2 = kontur_transformieren(k1,     0.0,              0.0, 2.0)
k3 = kontur_transformieren(k2,     np.deg2rad(-15),  0.0, 0.0)

# Gesamttransformation
T_ges = T3 @ T2 @ T1
einsen = np.ones((kontur.shape[0], 1))
k_ges = (T_ges @ np.hstack([kontur, einsen]).T).T[:, :2]

print(f"Schrittweise == Gesamtmatrix: {np.allclose(k3, k_ges)}")

# Darstellung: Kontur schliessen durch ersten Punkt am Ende wiederholen
fig, ax = plt.subplots(figsize=(8, 6))
for k, label, farbe in [(kontur, 'Original', 'blue'),
                         (k1,     'Nach T1',  'green'),
                         (k2,     'Nach T2',  'orange'),
                         (k3,     'Nach T3',  'red')]:
    geschlossen = np.vstack([k, k[0]])  # ersten Punkt ans Ende haengen
    ax.plot(geschlossen[:, 0], geschlossen[:, 1], 'o-',
            color=farbe, linewidth=1.8, markersize=5, label=label)

ax.set_aspect('equal'); ax.grid(True); ax.legend()
ax.set_title('CAD-Kontur: schrittweise Transformation')
plt.show()
```

Die schrittweise Verkettung und die direkte Gesamtmatrix liefern dasselbe
Ergebnis. Das zeigt, dass Matrizenprodukte die Transformationssequenz
kompakt kodieren: Statt drei separate Operationen auszuführen, genügt
eine einzige Matrizenmultiplikation.
````

````{admonition} Übung 5.7 (✩✩)
:class: tip
Wir untersuchen, wie sich eine Drehung um einen beliebigen Punkt
(nicht um den Ursprung) mit homogenen Koordinaten realisieren lässt.

Ein Zahnrad hat seinen Mittelpunkt bei $M = (3, 2)^\top$. Es soll um
seinen eigenen Mittelpunkt um $\varphi = 72°$ gedreht werden
(ein Fünftel einer vollen Umdrehung).

1. Erklären Sie in eigenen Worten, welche drei Transformationen dafür
   nötig sind und in welcher Reihenfolge sie ausgeführt werden müssen.
2. Schreiben Sie eine Funktion `drehen_um_punkt(phi, mx, my)`, die die
   homogene Gesamtmatrix für eine Drehung um den Punkt $(m_x, m_y)^\top$
   zurückgibt.
3. Das Zahnrad wird durch fünf gleichmäßig auf einem Kreis mit Radius
   $r = 1.5$ um den Mittelpunkt verteilte Punkte beschrieben. Berechnen
   Sie diese fünf Punkte und wenden Sie die Drehung an. Stellen Sie
   Original und gedrehtes Zahnrad dar.
4. Überprüfen Sie: Der Mittelpunkt $M$ selbst soll durch die Drehung
   unverändert bleiben. Verifizieren Sie das mit `np.allclose`.
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

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx], [s, c, ty], [0, 0, 1]])

def drehen_um_punkt(phi, mx, my):
    """Drehung um den Punkt (mx, my): erst verschieben, drehen, zurueckverschieben."""
    T_hin    = transformationsmatrix(0.0, -mx, -my)  # Schritt 1: M in Ursprung
    T_drehen = transformationsmatrix(phi,  0.0, 0.0) # Schritt 2: drehen
    T_zurueck = transformationsmatrix(0.0,  mx,  my) # Schritt 3: M zurueck
    return T_zurueck @ T_drehen @ T_hin

# Fünf Punkte auf einem Kreis um M = (3, 2)
mx, my, r = 3.0, 2.0, 1.5
winkel_punkte = np.linspace(0, 2 * np.pi, 5, endpoint=False)
zahnrad = np.column_stack([
    mx + r * np.cos(winkel_punkte),
    my + r * np.sin(winkel_punkte),
])

phi = np.deg2rad(72)
T = drehen_um_punkt(phi, mx, my)

einsen = np.ones((zahnrad.shape[0], 1))
zahnrad_gedreht = (T @ np.hstack([zahnrad, einsen]).T).T[:, :2]

# Mittelpunkt pruefen
M_hom      = np.array([mx, my, 1.0])
M_gedreht  = T @ M_hom
print(f"Mittelpunkt original: ({mx}, {my})")
print(f"Mittelpunkt gedreht:  {np.round(M_gedreht[:2], 4)}")
print(f"Unveraendert:         {np.allclose(M_hom[:2], M_gedreht[:2])}")

# Darstellung
fig, ax = plt.subplots(figsize=(6, 6))
geschlossen_orig   = np.vstack([zahnrad,        zahnrad[0]])
geschlossen_dreht  = np.vstack([zahnrad_gedreht, zahnrad_gedreht[0]])
ax.plot(geschlossen_orig[:, 0],  geschlossen_orig[:, 1],
        'bo-', linewidth=1.8, markersize=7, label='Original')
ax.plot(geschlossen_dreht[:, 0], geschlossen_dreht[:, 1],
        'ro-', linewidth=1.8, markersize=7, label='Gedreht 72°')
ax.plot(mx, my, 'k+', markersize=12, markeredgewidth=2, label='Mittelpunkt M')
ax.set_aspect('equal'); ax.grid(True); ax.legend()
ax.set_title('Drehung um eigenen Mittelpunkt')
plt.show()
```

Der Mittelpunkt bleibt durch die Drehung unverändert, weil er durch die
erste Transformation in den Ursprung verschoben wird, dort durch die
Drehung nicht bewegt wird ($\mathbf{R} \cdot \vec{0} = \vec{0}$) und
anschließend wieder an seine ursprüngliche Position zurückverschoben wird.
````

````{admonition} Übung 5.8 (✩✩✩)
:class: tip
**Teil 1: Animiertes Pendel**

Ein Pendel der Länge $l = 2$ ist im Ursprung aufgehängt. Die Pendelspitze
beschreibt bei Auslenkung $\theta$ die Position
$(l \sin\theta,\, -l \cos\theta)^\top$.

1. Schreiben Sie eine Funktion `pendelspitze(theta, l)`, die die Position
   der Pendelspitze als homogene Transformation berechnet: Drehung um
   $\theta$ im Aufhängepunkt $(0, 0)^\top$, dann Verschiebung um $(0, -l)^\top$.
   Vergleichen Sie das Ergebnis mit der analytischen Formel.

**Teil 2: Doppelpendel**

Ein Doppelpendel besteht aus zwei Segmenten: Segment 1 hat Länge $l_1 = 1.5$
und Winkel $\theta_1 = 30°$, Segment 2 hat Länge $l_2 = 1.0$ und Winkel
$\theta_2 = -60°$ relativ zu Segment 1.

2. Berechnen Sie die Positionen von Aufhängepunkt, erstem Gelenk und
   zweiter Pendelspitze mithilfe von verketteten homogenen Transformationen.
3. Zeichnen Sie das Doppelpendel als Linie mit Punkten an den drei
   charakteristischen Stellen.

**Teil 3: Trajektorie**

4. Variieren Sie $\theta_1$ von $-60°$ bis $60°$ in 25 Schritten und
   behalten Sie $\theta_2 = -45°$ fest. Zeichnen Sie die Trajektorie der
   zweiten Pendelspitze als Kurve in dieselbe Abbildung wie das Pendel
   aus Teil 2.
5. Was ist die geometrische Form dieser Trajektorie, und warum?
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

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx], [s, c, ty], [0, 0, 1]])

# Teil 1: Pendelspitze als Transformation
def pendelspitze(theta, l):
    """Berechnet die Pendelspitze ueber eine homogene Transformation."""
    # Drehung um theta im Ursprung, dann Verschiebung um (0, -l)
    T = transformationsmatrix(theta, 0.0, -l)
    ursprung_hom = np.array([0.0, 0.0, 1.0])
    return (T @ ursprung_hom)[:2]

theta_test = np.deg2rad(30)
l_test     = 2.0
spitze_T    = pendelspitze(theta_test, l_test)
spitze_ana  = np.array([l_test * np.sin(theta_test),
                        -l_test * np.cos(theta_test)])
print(f"Transformation: {np.round(spitze_T, 4)}")
print(f"Analytisch:     {np.round(spitze_ana, 4)}")
print(f"Gleich:         {np.allclose(spitze_T, spitze_ana)}")

# Teil 2: Doppelpendel
l1, l2     = 1.5, 1.0
theta1     = np.deg2rad(30)
theta2     = np.deg2rad(-60)

ursprung_hom = np.array([0.0, 0.0, 1.0])

T1      = transformationsmatrix(theta1, 0.0, -l1)
gelenk  = (T1 @ ursprung_hom)[:2]

T2      = transformationsmatrix(theta2, 0.0, -l2)
spitze2 = (T1 @ T2 @ ursprung_hom)[:2]

print(f"\nGelenk:   {np.round(gelenk, 4)}")
print(f"Spitze 2: {np.round(spitze2, 4)}")

# Teil 3: Trajektorie von Spitze 2
theta1_werte = np.linspace(np.deg2rad(-60), np.deg2rad(60), 25)
traj = np.array([
    (transformationsmatrix(t1, 0.0, -l1) @
     transformationsmatrix(theta2, 0.0, -l2) @
     ursprung_hom)[:2]
    for t1 in theta1_werte
])

# Darstellung
fig, ax = plt.subplots(figsize=(6, 7))

# Pendel aus Teil 2
pendel_punkte = np.array([[0.0, 0.0], gelenk, spitze2])
ax.plot(pendel_punkte[:, 0], pendel_punkte[:, 1],
        'bo-', linewidth=2.5, markersize=9, label='Doppelpendel (30°, -60°)')

# Trajektorie
ax.plot(traj[:, 0], traj[:, 1], 'r--', linewidth=1.5, label='Trajektorie Spitze 2')
ax.plot(traj[0, 0], traj[0, 1], 'rs', markersize=7)
ax.plot(traj[-1, 0], traj[-1, 1], 'rs', markersize=7)

ax.axhline(0, color='gray', linewidth=0.8)
ax.axvline(0, color='gray', linewidth=0.8)
ax.set_aspect('equal'); ax.grid(True); ax.legend()
ax.set_title('Doppelpendel und Trajektorie der Spitze')
plt.show()
```

Die Trajektorie von Spitze 2 ist ein Kreisbogen. Da $\theta_2$ und damit
der Winkel zwischen den Segmenten konstant bleibt, hat Spitze 2 einen
festen Abstand vom ersten Gelenk. Weil das erste Gelenk selbst auf einem
Kreisbogen um den Ursprung läuft, ergibt die Überlagerung eine komplexere
Kurve, die bei konstantem $\theta_2$ jedoch einem Kreisbogen ähnelt, dessen
Radius durch die Gesamtlänge $l_1 + l_2 \cos\theta_2$ bestimmt wird.
````

````{admonition} Übung 5.9 (✩✩✩)
:class: tip
**Teil 1: Sensorpose im Fahrzeug**

Ein Lidar-Sensor ist auf einem Fahrzeug montiert. Sein Ursprung liegt bei
$(x_s, y_s) = (1.5, 0.0)^\top$ (1.5 m vor dem Fahrzeugmittelpunkt) und er
ist gegenüber dem Fahrzeug um $\psi_s = 15°$ gedreht (leicht nach links
ausgerichtet). Das Fahrzeug selbst steht im Weltkoordinatensystem bei
$(x_F, y_F) = (10, 5)^\top$ und ist um $\psi_F = 45°$ gedreht (diagonal
zur Weltachse).

Der Sensor misst einen Hindernis-Punkt bei
$\vec{p}_\text{sensor} = (3.0, 0.5)^\top$ (3 m geradeaus, 0.5 m links).

1. Bauen Sie die Transformationsmatrix $\mathbf{T}_{FS}$ vom Sensor- ins
   Fahrzeugkoordinatensystem und $\mathbf{T}_{WF}$ vom Fahrzeug- ins
   Weltkoordinatensystem auf.
2. Berechnen Sie die Position des Hindernisses im Weltkoordinatensystem
   durch Verkettung: $\mathbf{T}_{WF} \cdot \mathbf{T}_{FS} \cdot
   \tilde{p}_\text{sensor}$.
3. Geben Sie die Zwischenergebnisse aus: Wo liegt der Punkt im
   Fahrzeugkoordinatensystem, und wo im Weltkoordinatensystem?

**Teil 2: Visualisierung**

4. Zeichnen Sie die Szene: Fahrzeugposition als Kreuz, Sensorposition
   als Dreieck, den gemessenen Punkt im Sensor-, Fahrzeug- und
   Weltkoordinatensystem als Punkte verschiedener Farbe. Verbinden Sie
   alle drei Positionen des Hindernispunktes mit gestrichelten Linien.

**Teil 3: Reflexion**

5. Warum reicht es, zwei Matrizen zu verketten, statt drei getrennte
   Transformationen auszuführen? Welchen praktischen Vorteil hat das in
   einem echten Fahrzeugsystem, das Dutzende von Sensoren hat?
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

def transformationsmatrix(phi, tx, ty):
    c, s = np.cos(phi), np.sin(phi)
    return np.array([[c, -s, tx], [s, c, ty], [0, 0, 1]])

# Teil 1: Transformationsmatrizen aufbauen
# T_FS: vom Sensor- ins Fahrzeugkoordinatensystem
# Sensor sitzt bei (1.5, 0) im Fahrzeug und ist um 15 Grad gedreht
T_FS = transformationsmatrix(np.deg2rad(15), 1.5, 0.0)

# T_WF: vom Fahrzeug- ins Weltkoordinatensystem
# Fahrzeug ist bei (10, 5) und um 45 Grad gedreht
T_WF = transformationsmatrix(np.deg2rad(45), 10.0, 5.0)

# Hindernispunkt im Sensorkoordinatensystem
p_sensor_hom = np.array([3.0, 0.5, 1.0])

# Teil 2 und 3: Verkettung
p_fahrzeug_hom = T_FS @ p_sensor_hom
p_welt_hom     = T_WF @ T_FS @ p_sensor_hom

print(f"Hindernis im Sensorkoordinatensystem:   {p_sensor_hom[:2]}")
print(f"Hindernis im Fahrzeugkoordinatensystem: {np.round(p_fahrzeug_hom[:2], 4)}")
print(f"Hindernis im Weltkoordinatensystem:     {np.round(p_welt_hom[:2], 4)}")

# Positionen von Sensor und Fahrzeug im Weltkoordinatensystem
fahrzeug_pos = np.array([10.0, 5.0])
sensor_pos   = (T_WF @ np.array([1.5, 0.0, 1.0]))[:2]

# Teil 4: Darstellung
fig, ax = plt.subplots(figsize=(8, 8))

# Fahrzeug und Sensor
ax.plot(*fahrzeug_pos, 'k+', markersize=14, markeredgewidth=2.5,
        label='Fahrzeugmittelpunkt')
ax.plot(*sensor_pos, '^', color='gray', markersize=10,
        label='Sensorposition (Welt)')

# Hindernispunkt in drei Koordinatensystemen
ax.plot(*p_sensor_hom[:2], 'bo', markersize=9, label='Hindernis (Sensor-KS)')
ax.plot(*p_fahrzeug_hom[:2], 'go', markersize=9,
        label='Hindernis (Fahrzeug-KS)')
ax.plot(*p_welt_hom[:2], 'ro', markersize=9, label='Hindernis (Welt-KS)')

# Verbindungslinien
for start, ende in [(p_sensor_hom[:2],   p_fahrzeug_hom[:2]),
                    (p_fahrzeug_hom[:2], p_welt_hom[:2])]:
    ax.plot([start[0], ende[0]], [start[1], ende[1]],
            'k--', linewidth=0.8, alpha=0.5)

ax.set_xlim(-1, 16); ax.set_ylim(-2, 14)
ax.set_aspect('equal'); ax.grid(True); ax.legend(loc='upper left')
ax.set_title('Sensor-Fahrzeug-Welt: Koordinatentransformation')
plt.show()
```

Die Verkettung $\mathbf{T}_{WF} \cdot \mathbf{T}_{FS}$ erlaubt es, jeden
neu gemessenen Punkt direkt ins Weltkoordinatensystem zu transformieren,
ohne die Zwischenschritte explizit auszuführen. In einem echten Fahrzeug
mit zehn oder mehr Sensoren berechnet man einmalig für jeden Sensor die
kombinierte Matrix und wendet sie dann auf alle eingehenden Messpunkte an.
Das spart Rechenzeit und verhindert Programmierfehler durch vergessene
Transformationsschritte.
````
