---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 6.5 Übungen

Diese Aufgaben sind für das Selbststudium zuhause gedacht und wiederholen den
Stoff der Kapitel 6.1 bis 6.4. Rechnen Sie mit rund 90 Minuten
Bearbeitungszeit. Führen Sie vor den Aufgaben die beiden folgenden Zellen aus.
Sie enthalten die Zeichenfunktion, die Funktionen aus Kapitel 5.3 sowie
Umriss, Hallenkarte und Übergabestation aus Kapitel 6.1.

Der Schwierigkeitsgrad steht im Titel jeder Aufgabe:

* ✩ Verständnis: Code und Ausgaben vorhersagen und erklären (ca. 10 min)
* ✩✩ Anwendung: eigenen Code schreiben und Ergebnisse interpretieren (ca. 25 min)

```{code-cell} python
:tags: [hide-input]
# Vorgegebene Zeichenfunktion: einfach ausführen, sie muss nicht verstanden werden.
def zeichne_halle(fahrzeuge=None, koordinatensysteme=None, messpunkte=None,
                  scanner=None, schutzfeld=None, ziele=None, bahn=None,
                  umgebung=None, beschriftungen=None, titel='',
                  bereich=(0, 10, 0, 8)):
    """Zeichnet einen Ausschnitt der Halle von oben.

    fahrzeuge: Ecken eines Fahrzeugumrisses als Array, Zeile = [x, y], die
               ersten beiden Ecken bilden die Vorderkante, oder eine Liste
               solcher Arrays für mehrere Fahrzeuge (optional)
    koordinatensysteme: homogene 3x3-Matrix eines Koordinatensystems oder eine
               Liste solcher Matrizen, gezeichnet werden die x-Achse (rot) und
               die y-Achse (blau) mit 0.8 m Länge (optional)
    messpunkte: Messpunkte als Array, Zeile = [x, y], werden als rote Punkte
                gezeichnet (optional)
    scanner: Lage [x, y] des Scanners, zeichnet die Laserstrahlen vom Scanner
             zu den Messpunkten als dünne rote Linien (optional)
    schutzfeld: Ecken eines Schutzfelds als Array, Zeile = [x, y], wird gelb
                gefüllt und gestrichelt umrandet (optional)
    ziele: Zielpunkte als Array, Zeile = [x, y], werden als Kreuze markiert (optional)
    bahn: Bahnpunkte als Array, Zeile = [x, y], werden als gestrichelte Linie
          mit Punkten über den Fahrzeugen gezeichnet (optional)
    umgebung: Ecken von Flächen wie Regalen oder Paletten als Array oder Liste
              von Arrays, werden grau gefüllt (optional)
    beschriftungen: Legendentexte, einer pro Fahrzeug, ohne Beschriftungen
                    werden alle Fahrzeuge blau umrandet (optional)
    titel: Diagrammtitel
    bereich: sichtbarer Ausschnitt (x_min, x_max, y_min, y_max) in m
    """
    blau, rot, orange, grau = '#005A94', '#E60000', '#E87846', '#484949'
    farben = [blau, rot, orange, grau]
    if isinstance(fahrzeuge, np.ndarray):
        fahrzeuge = [fahrzeuge]
    if isinstance(koordinatensysteme, np.ndarray):
        koordinatensysteme = [koordinatensysteme]
    if isinstance(umgebung, np.ndarray):
        umgebung = [umgebung]
    fig, ax = plt.subplots(figsize=(7, 6))

    # Umgebung: Regale, Paletten, Säulen
    if umgebung is not None:
        for flaeche in umgebung:
            ax.fill(flaeche[:, 0], flaeche[:, 1], facecolor='#DDDEDE',
                    edgecolor=grau, linewidth=1, zorder=0)

    # Schutzfeld gelb gefüllt, gestrichelt umrandet
    if schutzfeld is not None:
        ax.fill(schutzfeld[:, 0], schutzfeld[:, 1], facecolor='#FFEC7F',
                edgecolor=orange, linestyle='--', linewidth=1.5,
                label='Schutzfeld', zorder=1)

    # Bahn als gestrichelte Linie mit Punkten, über den Fahrzeugen
    if bahn is not None:
        ax.plot(bahn[:, 0], bahn[:, 1], color=grau, linestyle='--', linewidth=1,
                marker='o', markersize=3, label='Bahn', zorder=4)

    # Fahrzeuge: Umriss hellblau gefüllt, Vorderkante (Ecke 0 bis Ecke 1) orange
    if fahrzeuge is not None:
        for k in range(len(fahrzeuge)):
            farbe = blau if beschriftungen is None else farben[k % len(farben)]
            ecken = fahrzeuge[k]
            label = None if beschriftungen is None else beschriftungen[k]
            ax.fill(ecken[:, 0], ecken[:, 1], facecolor='#CCDEE9', edgecolor=farbe,
                    linewidth=2, label=label, zorder=2)
            ax.plot(ecken[0:2, 0], ecken[0:2, 1], color=orange, linewidth=4,
                    solid_capstyle='butt', zorder=3)

    # Koordinatensysteme: x-Achse rot, y-Achse blau, je 0.8 m lang
    if koordinatensysteme is not None:
        for T in koordinatensysteme:
            ursprung = T[0:2, 2]
            for spalte, farbe, name in [(0, rot, 'x'), (1, blau, 'y')]:
                spitze = ursprung + 0.8 * T[0:2, spalte]
                ax.annotate('', xy=spitze, xytext=ursprung, zorder=4,
                            arrowprops=dict(arrowstyle='-|>', color=farbe, lw=2))
                beschriftung = ursprung + 1.0 * T[0:2, spalte]
                ax.text(beschriftung[0], beschriftung[1], name, color=farbe,
                        ha='center', va='center', fontsize=11, zorder=4)
            ax.scatter(ursprung[0], ursprung[1], s=25, color=grau, zorder=5)

    # Messpunkte als rote Punkte, auf Wunsch mit Laserstrahlen vom Scanner aus
    if messpunkte is not None:
        messpunkte = np.atleast_2d(messpunkte)
        if scanner is not None:
            for punkt in messpunkte:
                ax.plot([scanner[0], punkt[0]], [scanner[1], punkt[1]], color=rot,
                        linewidth=0.8, alpha=0.6, zorder=4)
        ax.scatter(messpunkte[:, 0], messpunkte[:, 1], s=25, color=rot, zorder=5,
                   label='Messpunkte')

    # Zielpunkte als Kreuze
    if ziele is not None:
        ziele = np.atleast_2d(ziele)
        ax.scatter(ziele[:, 0], ziele[:, 1], marker='x', s=80, color=grau,
                   linewidth=2, zorder=5, label='Ziel')

    if (beschriftungen is not None or messpunkte is not None or ziele is not None
            or bahn is not None or schutzfeld is not None):
        ax.legend(loc='upper left')
    ax.set_xlim(bereich[0], bereich[1])
    ax.set_ylim(bereich[2], bereich[3])
    ax.set_aspect('equal')
    ax.grid(True)
    ax.set_xlabel('x in m')
    ax.set_ylabel('y in m')
    ax.set_title(titel)
    plt.show()
```

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Funktionen aus Kapitel 5.3
def transformationsmatrix(winkel, tx, ty):
    """Homogene Matrix: um winkel (Radiant) drehen, dann um (tx, ty) verschieben."""
    return np.array([
        [np.cos(winkel), -np.sin(winkel), tx],
        [np.sin(winkel),  np.cos(winkel), ty],
        [0.0,             0.0,            1.0],
    ])

def transformiere(T, punkte):
    """Wendet die homogene Matrix T auf alle Punkte an, Zeile = [x, y]."""
    punkte_hom = np.ones((len(punkte), 3))
    punkte_hom[:, 0:2] = punkte
    ergebnis = (T @ punkte_hom.T).T
    return ergebnis[:, 0:2]

# Ecken des FTF im Fahrzeugkoordinatensystem in m, Zeile = [x, y]
umriss = np.array([
    [ 0.6, -0.4],   # vorne rechts
    [ 0.6,  0.4],   # vorne links
    [-0.6,  0.4],   # hinten links
    [-0.6, -0.4],   # hinten rechts
])

# Hallenkarte aus Kapitel 6.1: Ecken von Regal, Palette und Säule in m
regal = np.array([[0.5, 6.5], [9.5, 6.5], [9.5, 7.5], [0.5, 7.5]])
palette = np.array([[5.6, 4.0], [6.8, 4.0], [6.8, 4.8], [5.6, 4.8]])
saeule = np.array([[6.3, 0.3], [6.7, 0.3], [6.7, 0.7], [6.3, 0.7]])
halle = [regal, palette, saeule]

# Übergabestation aus Kapitel 6.1 in Hallenkoordinaten in m
station = np.array([[8.0, 2.5]])
```

````{admonition} Aufgabe 6.1 (✩)
:class: tip
Die Leitsteuerung meldet die Pose eines FTF nicht als drei Zahlen, sondern
gleich als Matrix $\mathbf{T}_{WF}$. Gegeben ist folgender Code:

```python
import numpy as np

T_WF = np.array([
    [ 0.0, 1.0, 6.0],
    [-1.0, 0.0, 3.0],
    [ 0.0, 0.0, 1.0],
])
print(np.rad2deg(np.arctan2(T_WF[1, 0], T_WF[0, 0])))
print(np.linalg.det(T_WF))

# Messung des Scanners in der Fahrzeugmitte: Entfernung 2 m, Winkel 90 Grad
r = 2.0
phi = np.deg2rad(90)
messpunkt_F = np.array([[r * np.cos(phi), r * np.sin(phi)]])
print(np.round(messpunkt_F, 4))
print(np.round(transformiere(T_WF, messpunkt_F), 4))
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Wo steht die Fahrzeugmitte, und in welche Richtung der Halle zeigt die
   $x_F$-Achse? Welchen Gierwinkel gibt die erste `print`-Zeile aus?
2. Was gibt `np.linalg.det(T_WF)` aus? Begründen Sie Ihre Antwort.
3. Der Scanner meldet einen Gegenstand in $2\,\text{m}$ Entfernung unter dem
   Winkel $\varphi = 90°$. Wo liegt er im Fahrzeugkoordinatensystem? In welche
   Richtung der Halle zeigt die $y_F$-Achse, und wo liegt der Gegenstand in der
   Halle?
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

T_WF = np.array([
    [ 0.0, 1.0, 6.0],
    [-1.0, 0.0, 3.0],
    [ 0.0, 0.0, 1.0],
])
print(np.rad2deg(np.arctan2(T_WF[1, 0], T_WF[0, 0])))
print(np.linalg.det(T_WF))

r = 2.0
phi = np.deg2rad(90)
messpunkt_F = np.array([[r * np.cos(phi), r * np.sin(phi)]])
print(np.round(messpunkt_F, 4))
print(np.round(transformiere(T_WF, messpunkt_F), 4))
```
Ausgabe:

```text
-90.0
1.0
[[0. 2.]]
[[8. 3.]]
```

Die dritte Spalte von $\mathbf{T}_{WF}$ ist die Lage der Fahrzeugmitte, also
$(6, 3)$. In der ersten Spalte steht die Richtung der $x_F$-Achse in
Hallenkoordinaten, hier $(0, -1)$. Das FTF zeigt also in negative
$y_W$-Richtung, zur Hallenwand hin. `np.arctan2` bekommt wie in Kapitel 6.3
zuerst die $y$- und dann die $x$-Komponente dieser Richtung und liefert
$\psi = -90°$. Die Determinante ist 1 wie bei jeder homogenen
Transformationsmatrix, die nur dreht und verschiebt.

Der Gegenstand liegt im Fahrzeugkoordinatensystem bei $(0, 2)$, also
$2\,\text{m}$ links von der Fahrzeugmitte. Der winzige Rest von
$\cos 90° \approx 6 \cdot 10^{-17}$ verschwindet durch `np.round`. Die zweite
Spalte $(1, 0)$ verrät, dass die $y_F$-Achse in $x_W$-Richtung zeigt: Wer in
negative $y_W$-Richtung schaut, hat die positive $x_W$-Richtung zu seiner
Linken. Der Gegenstand liegt deshalb $2\,\text{m}$ weiter in $x_W$-Richtung
als die Fahrzeugmitte, bei $(8, 3)$.
````

````{admonition} Aufgabe 6.2 (✩)
:class: tip
Das FTF steht bei $(4, 2)$ und zeigt mit $\psi = 90°$ in $y_W$-Richtung. In
Kapitel 6.1 haben wir die Umrechnung von Hallen- in Fahrzeugkoordinaten mit
`np.linalg.inv` berechnet. Der folgende Code enthält drei Versuche, die
inverse Matrix ohne `np.linalg.inv` aus einer Drehung und einer Verschiebung
zusammenzusetzen:

```python
import numpy as np

T_WF = transformationsmatrix(np.deg2rad(90), 4, 2)
T_FW = np.linalg.inv(T_WF)

T_a = transformationsmatrix(np.deg2rad(-90), -4, -2)
T_b = transformationsmatrix(0, -4, -2) @ transformationsmatrix(np.deg2rad(-90), 0, 0)
T_c = transformationsmatrix(np.deg2rad(-90), 0, 0) @ transformationsmatrix(0, -4, -2)

print(np.allclose(T_a, T_FW))
print(np.allclose(T_b, T_FW))
print(np.allclose(T_c, T_FW))
print(np.round(T_FW, 4))
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Was wirkt bei `T_b` zuerst, die Drehung oder die Verschiebung? Und bei
   `T_c`? Zwei der drei Versuche ergeben dieselbe Matrix. Welche?
2. Welcher Versuch ist gleich `T_FW`? Begründen Sie Ihre Antwort mit der
   Reihenfolge, in der $\mathbf{T}_{WF}$ dreht und verschiebt.
3. In der rechten Spalte von `T_FW` steht, wohin `T_FW` den Punkt $(0, 0)$
   bringt. Wo liegt also der Ursprung der Halle aus Sicht des Fahrzeugs: vorne
   oder hinten, links oder rechts, und wie weit jeweils?
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

T_WF = transformationsmatrix(np.deg2rad(90), 4, 2)
T_FW = np.linalg.inv(T_WF)

T_a = transformationsmatrix(np.deg2rad(-90), -4, -2)
T_b = transformationsmatrix(0, -4, -2) @ transformationsmatrix(np.deg2rad(-90), 0, 0)
T_c = transformationsmatrix(np.deg2rad(-90), 0, 0) @ transformationsmatrix(0, -4, -2)

print(np.allclose(T_a, T_FW))
print(np.allclose(T_b, T_FW))
print(np.allclose(T_c, T_FW))
print(np.round(T_FW, 4))
```
Ausgabe:

```text
False
False
True
[[ 0.  1. -2.]
 [-1.  0.  4.]
 [ 0.  0.  1.]]
```

In einem Matrixprodukt wirkt die rechte Matrix zuerst. `T_b` dreht also zuerst
und verschiebt dann, genau wie `transformationsmatrix` selbst. Deshalb sind
`T_a` und `T_b` dieselbe Matrix, die naheliegende, aber falsche Umkehrung aus
Kapitel 6.1. `T_c` verschiebt zuerst und dreht dann. $\mathbf{T}_{WF}$ hat
zuerst gedreht und zuletzt verschoben. Die Umkehrung muss die Schritte in
umgekehrter Reihenfolge rückgängig machen: zuerst die Verschiebung, dann die
Drehung. Das leistet nur `T_c`.

In der rechten Spalte von `T_FW` steht $(-2, 4)$. Der Ursprung der Halle liegt
aus Sicht des Fahrzeugs also $2\,\text{m}$ hinter und $4\,\text{m}$ links von
der Fahrzeugmitte. Das passt: Das FTF zeigt in $y_W$-Richtung, der Ursprung
liegt $2\,\text{m}$ tiefer, also hinten. Für ein Fahrzeug, das in
$y_W$-Richtung zeigt, ist die negative $x_W$-Richtung links, und dort liegt der
Ursprung $4\,\text{m}$ entfernt. Die rechte Spalte ist deshalb nicht einfach
$(-4, -2)$, denn die Rückdrehung erfasst auch den Verschiebungsvektor.
````

````{admonition} Aufgabe 6.3 (✩)
:class: tip
Ein FTF hat zusätzlich einen Scanner am Heck. Er sitzt $0.6\,\text{m}$ hinter
der Fahrzeugmitte und blickt nach hinten, seine $x_S$-Achse zeigt also
entgegen der $x_F$-Achse. Gegeben ist folgender Code:

```python
import numpy as np

T_WF = transformationsmatrix(np.deg2rad(90), 4, 2)
# Heckscanner: 0.6 m hinter der Fahrzeugmitte, um 180 Grad gedreht
T_FS = transformationsmatrix(np.deg2rad(180), -0.6, 0)

# Messung des Heckscanners: Entfernung 1 m, Winkel 0 Grad
messpunkt_S = np.array([[1.0, 0.0]])

print(np.round(transformiere(T_FS, messpunkt_S), 4))
print(np.round(transformiere(T_WF @ T_FS, messpunkt_S), 4))
print(np.round(transformiere(T_FS @ T_WF, messpunkt_S), 4))
print(np.round((T_WF @ T_FS)[0:2, 2], 4))
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Wo liegt der Messpunkt im Fahrzeugkoordinatensystem? Wie weit ist er vom
   Heck des FTF entfernt?
2. Welches der beiden Matrixprodukte rechnet richtig vom Scanner in die Halle
   um? Begründen Sie Ihre Antwort mit den Indizes. Wo liegt der Messpunkt in
   der Halle?
3. Was bedeutet die Ausgabe der letzten Zeile?
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

T_WF = transformationsmatrix(np.deg2rad(90), 4, 2)
T_FS = transformationsmatrix(np.deg2rad(180), -0.6, 0)

messpunkt_S = np.array([[1.0, 0.0]])

print(np.round(transformiere(T_FS, messpunkt_S), 4))
print(np.round(transformiere(T_WF @ T_FS, messpunkt_S), 4))
print(np.round(transformiere(T_FS @ T_WF, messpunkt_S), 4))
print(np.round((T_WF @ T_FS)[0:2, 2], 4))
```
Ausgabe:

```text
[[-1.6  0. ]]
[[4.  0.4]]
[[-4.6 -3. ]]
[4.  1.4]
```

$\mathbf{T}_{FS}$ dreht den Messpunkt zuerst um $180°$ nach $(-1, 0)$ und
verschiebt ihn dann um $(-0.6, 0)$. Er liegt also bei $(-1.6, 0)$,
$1.6\,\text{m}$ hinter der Fahrzeugmitte. Das Heck liegt bei
$x_F = -0.6\,\text{m}$, der Gegenstand ist also $1\,\text{m}$ vom Heck
entfernt, genau die gemessene Entfernung.

Richtig ist $\mathbf{T}_{WF} \cdot \mathbf{T}_{FS}$. Wie in Kapitel 6.2 greifen
die Indizes wie Kettenglieder ineinander: $\mathbf{T}_{FS}$ rechnet von S nach
F, $\mathbf{T}_{WF}$ von F nach W. Das FTF zeigt in $y_W$-Richtung, hinten ist
also die negative $y_W$-Richtung, und der Messpunkt liegt bei
$(4, 2 - 1.6) = (4, 0.4)$. Bei $\mathbf{T}_{FS} \cdot \mathbf{T}_{WF}$ passen
die inneren Indizes S und W nicht zusammen. Das Produkt wendet zuerst
$\mathbf{T}_{WF}$ auf die Scannerkoordinaten an, als wären es
Fahrzeugkoordinaten, und danach $\mathbf{T}_{FS}$ auf Hallenkoordinaten, als
wären es Scannerkoordinaten. Der Punkt $(-4.6, -3)$ liegt nicht einmal in der
Halle.
Die letzte Zeile gibt die dritte Spalte von $\mathbf{T}_{WS}$ aus, also die
Lage des Heckscanners in der Halle. Er liegt bei $(4, 1.4)$, $0.6\,\text{m}$
hinter der Fahrzeugmitte.
````

````{admonition} Aufgabe 6.4 (✩)
:class: tip
Nach der Übergabe fährt das FTF von der Station bei $(8, 2.5)$ auf einer
flachen Bahn nach links zurück. Der folgende Code berechnet wie in Kapitel 6.3
die Gierwinkel an den Bahnpunkten und die Drehungen zwischen benachbarten
Bahnpunkten:

```python
import numpy as np

bahn = np.array([[8.0, 2.5], [7.0, 2.7], [6.0, 2.7], [5.0, 2.5]])

psi_grad = np.zeros(len(bahn))
for k in range(len(bahn) - 1):
    richtung = bahn[k + 1] - bahn[k]
    psi_grad[k] = np.rad2deg(np.arctan2(richtung[1], richtung[0]))
psi_grad[-1] = psi_grad[-2]
print(np.round(psi_grad, 1))

drehungen = np.zeros(len(bahn) - 1)
for k in range(len(bahn) - 1):
    drehungen[k] = psi_grad[k + 1] - psi_grad[k]
print(np.round(drehungen, 1))
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. In welche Richtung der Halle zeigt das FTF auf der ganzen Fahrt ungefähr?
   Welche Werte erwarten Sie in `psi_grad`? Denken Sie daran, dass
   `np.arctan2` Winkel von $-180°$ bis $180°$ liefert. Tipp: Zur Steigung
   $0.2$ gehört der Winkel $11.3°$.
2. Skizzieren Sie die Bahn. Dreht sich das FTF von Abschnitt zu Abschnitt nach
   links oder nach rechts, und um wie viel Grad?
3. Welche Werte gibt die zweite `print`-Zeile aus? Dreht sich das FTF zwischen
   Bahnpunkt 1 und Bahnpunkt 2 wirklich so weit? Wie lässt sich der Wert
   korrigieren?
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

bahn = np.array([[8.0, 2.5], [7.0, 2.7], [6.0, 2.7], [5.0, 2.5]])

psi_grad = np.zeros(len(bahn))
for k in range(len(bahn) - 1):
    richtung = bahn[k + 1] - bahn[k]
    psi_grad[k] = np.rad2deg(np.arctan2(richtung[1], richtung[0]))
psi_grad[-1] = psi_grad[-2]
print(np.round(psi_grad, 1))

drehungen = np.zeros(len(bahn) - 1)
for k in range(len(bahn) - 1):
    drehungen[k] = psi_grad[k + 1] - psi_grad[k]
print(np.round(drehungen, 1))
```
Ausgabe:

```text
[ 168.7  180.  -168.7 -168.7]
[  11.3 -348.7    0. ]
```

Das FTF fährt nach links und zeigt deshalb ungefähr in negative
$x_W$-Richtung, also um $180°$. Der erste Abschnitt $(-1, 0.2)$ steigt leicht
an, das ergibt $180° - 11.3° = 168.7°$. Der zweite Abschnitt verläuft genau
nach links, $180°$. Der dritte Abschnitt $(-1, -0.2)$ fällt leicht ab. Seine
Richtung liegt bei $180° + 11.3° = 191.3°$. Weil `np.arctan2` nur Werte bis
$180°$ liefert, erhalten wir stattdessen $-168.7°$. Beide Winkel beschreiben
dieselbe Richtung, sie unterscheiden sich um eine volle Umdrehung.

Von Abschnitt zu Abschnitt dreht sich das FTF um $11.3°$ nach links: Wer nach
links fährt, hat die negative $y_W$-Richtung zu seiner Linken, und dorthin
biegt die Bahn ab. Die Rechnung meldet zwischen Bahnpunkt 1 und 2 aber
$-348.7°$, also fast eine volle Umdrehung nach rechts. Der Wert ist um $360°$
falsch, weil der Gierwinkel von $180°$ auf $-168.7°$ springt. Liegt eine
Drehung unter $-180°$, addieren wir $360°$, liegt sie über $180°$, ziehen wir
$360°$ ab. Hier ergibt das $-348.7° + 360° = 11.3°$. In den Kapiteln 6.3 und
6.4 trat das Problem nicht auf, weil die Bahnen dort nie in negative
$x_W$-Richtung führten.
````

```{admonition} Aufgabe 6.5 (✩✩)
:class: tip
An der Übergabestation `station` wartet eine Kiste. Drei FTF stehen gerade frei
in der Halle:

| FTF | Pose $(x, y, \psi)$ |
| --- | --- |
| FTF 1 | $(4.0, 2.0, 30°)$ |
| FTF 2 | $(9.0, 5.0, -90°)$ |
| FTF 3 | $(6.5, 3.0, 180°)$ |

Die Leitsteuerung schickt das FTF, das am schnellsten an der Station ist. Alle
drei haben einen **Differentialantrieb**: Ihre Antriebsräder links und rechts
werden einzeln angetrieben. Drehen sie gegenläufig, dreht sich das FTF auf der
Stelle, sofern Fahrwerk und Last das zulassen. Für ihre Abschätzung
nimmt die Leitsteuerung an, dass sich jedes FTF zuerst mit $45°$ pro Sekunde
auf der Stelle zur Station dreht und dann mit $1\,\text{m/s}$ geradeaus zur
Station fährt.

1. Schreiben Sie eine Funktion `ziel_aus_sicht(T_WF, ziel)` mit Docstring. Sie
   soll ein Ziel von Hallen- in Fahrzeugkoordinaten umrechnen und zusätzlich
   seinen Abstand $d$ von der Fahrzeugmitte sowie den Winkel $\varphi$ in Grad
   zurückgeben, unter dem das Ziel gegen die $x_F$-Achse liegt. $d$ und
   $\varphi$ sind die Polarkoordinaten aus Kapitel 6.1, nur in umgekehrter
   Richtung berechnet. Tipp: Pythagoras und `np.arctan2`.
2. Berechnen Sie für jedes FTF $d$, $\varphi$ und die geschätzte Zeit. Welches
   FTF schickt die Leitsteuerung?
3. Zeichnen Sie die drei FTF mit der Station und der Hallenkarte.
4. Beantworten Sie ohne Code: Den Abstand $d$ könnten wir auch direkt in
   Hallenkoordinaten berechnen. Warum? Den Winkel $\varphi$ könnten wir ohne
   inverse Matrix als Differenz zweier Winkel bestimmen. Wie, und welche
   Schwierigkeit aus Aufgabe 6.4 tritt dabei bei FTF 3 auf?

Strukturieren Sie Ihren Code mit EVA-Kommentaren.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Eingabe
def ziel_aus_sicht(T_WF, ziel):
    """Rechnet ein Ziel von Hallen- in Fahrzeugkoordinaten um.

    T_WF: homogene Matrix der Pose des FTF
    ziel: Ziel in Hallenkoordinaten als Array der Form (1, 2)
    Rückgabe: Ziel in Fahrzeugkoordinaten, Abstand d in m, Winkel phi in Grad
    """
    ziel_F = transformiere(np.linalg.inv(T_WF), ziel)[0]
    d = np.sqrt(ziel_F[0]**2 + ziel_F[1]**2)
    phi_grad = np.rad2deg(np.arctan2(ziel_F[1], ziel_F[0]))
    return ziel_F, d, phi_grad

# Posen der drei FTF, Zeile = [x in m, y in m, Gierwinkel in Grad]
posen = np.array([
    [4.0, 2.0,  30],
    [9.0, 5.0, -90],
    [6.5, 3.0, 180],
])
drehrate = 45.0          # Drehen auf der Stelle in Grad pro Sekunde
geschwindigkeit = 1.0    # Geradeausfahrt in m/s

# Verarbeitung: Station aus Sicht jedes FTF und geschätzte Zeit
fahrzeuge = []
zeiten = np.zeros(len(posen))
for k in range(len(posen)):
    T_WF_k = transformationsmatrix(np.deg2rad(posen[k, 2]), posen[k, 0], posen[k, 1])
    fahrzeuge.append(transformiere(T_WF_k, umriss))
    station_F, d, phi_grad = ziel_aus_sicht(T_WF_k, station)
    zeiten[k] = np.abs(phi_grad) / drehrate + d / geschwindigkeit
    print(f'FTF {k + 1}: Station bei {np.round(station_F, 3)} m, d = {d:.2f} m, '
          f'φ = {phi_grad:.1f}°, Zeit {zeiten[k]:.2f} s')

k_best = np.argmin(zeiten)

# Ausgabe
print(f'Die Leitsteuerung schickt FTF {k_best + 1}.')
zeichne_halle(fahrzeuge, ziele=station, umgebung=halle,
              beschriftungen=['FTF 1', 'FTF 2', 'FTF 3'],
              titel='Welches FTF übernimmt den Auftrag?')
```
Ausgabe:

```text
FTF 1: Station bei [ 3.714 -1.567] m, d = 4.03 m, φ = -22.9°, Zeit 4.54 s
FTF 2: Station bei [ 2.5 -1. ] m, d = 2.69 m, φ = -21.8°, Zeit 3.18 s
FTF 3: Station bei [-1.5  0.5] m, d = 1.58 m, φ = 161.6°, Zeit 5.17 s
Die Leitsteuerung schickt FTF 2.
```

FTF 3 steht der Station am nächsten, aber die Station liegt hinter ihm
($x_F < 0$). Es müsste sich um $161.6°$ drehen und bräuchte allein dafür
$3.6\,\text{s}$. FTF 2 zeigt schon fast zur Station, dreht sich nur um
$21.8°$ nach rechts und ist nach $3.18\,\text{s}$ dort. FTF 1 kennen wir aus
Kapitel 6.1: Für die Station bei $(3.714, -1.567)$ dreht es sich ähnlich
wenig, ist aber am weitesten entfernt. Die Leitsteuerung schickt FTF 2.

Drehungen und Verschiebungen ändern keine Abstände, das haben wir in Kapitel
5.1 gesehen. Deshalb ist $d$ in beiden Koordinatensystemen gleich, zum
Beispiel für FTF 3: $\sqrt{(8 - 6.5)^2 + (2.5 - 3)^2} = 1.58\,\text{m}$. Den
Winkel könnten wir als Richtung zur Station in der Halle minus Gierwinkel
berechnen. Bei FTF 3 liefert `np.arctan2(-0.5, 1.5)` die Richtung $-18.4°$,
und $-18.4° - 180° = -198.4°$. Das ist dieselbe Richtung wie $161.6°$, aber
außerhalb des Bereichs von $-180°$ bis $180°$. Wie in Aufgabe 6.4 müssten wir
$360°$ addieren, sonst würde die Leitsteuerung mit einer Drehung über $198°$
rechnen. In Fahrzeugkoordinaten liefert `np.arctan2` den Winkel dagegen
direkt im richtigen Bereich.
````

````{admonition} Aufgabe 6.6 (✩✩)
:class: tip
Das FTF hat im Regalgang aus Kapitel 6.4 eine Kiste aufgenommen. Es steht in
der Mitte des Gangs bei $(5.0, 5.0)$ und zeigt mit $\psi = 90°$ tiefer in den
Gang hinein. Für den Rückweg soll es sich auf der Stelle umdrehen, bis es mit
$\psi = 270°$ zum Hauptgang zeigt. Die beiden Regale links und rechts des
Gangs:

```python
regal_2 = np.array([[3.0, 2.4], [4.3, 2.4], [4.3, 7.5], [3.0, 7.5]])
regal_3 = np.array([[5.7, 2.4], [7.0, 2.4], [7.0, 7.5], [5.7, 7.5]])
```

1. Berechnen Sie den Umriss des FTF für die Gierwinkel
   $90°, 100°, \ldots, 270°$ und sammeln Sie die Umrisse in einer Liste.
   Sammeln Sie dabei für jeden Gierwinkel in zwei weiteren Listen die kleinste
   und die größte $x$-Koordinate der vier Ecken. Zeichnen Sie alle Umrisse mit den beiden
   Regalen und `bereich=(3.5, 6.5, 3.5, 6.5)`.
2. Wie weit ragt das FTF beim Drehen höchstens nach links und nach rechts, und
   bei welchem Gierwinkel? Stößt es an eines der Regale?
3. Beantworten Sie ohne Code: Wie weit ist jede Ecke von der Fahrzeugmitte
   entfernt? Wie breit muss ein Regalgang mindestens sein, damit das FTF darin
   mit $0.1\,\text{m}$ Sicherheitsabstand auf beiden Seiten wenden kann?
4. Beantworten Sie ohne Code: In Kapitel 6.4 haben wir den Abstand zur
   Regalecke im Fahrzeugkoordinatensystem geprüft. Warum prüfen wir hier in
   Hallenkoordinaten?

Strukturieren Sie Ihren Code mit EVA-Kommentaren.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Eingabe
regal_2 = np.array([[3.0, 2.4], [4.3, 2.4], [4.3, 7.5], [3.0, 7.5]])
regal_3 = np.array([[5.7, 2.4], [7.0, 2.4], [7.0, 7.5], [5.7, 7.5]])
psi_grad = np.linspace(90, 270, 19)

# Verarbeitung: Umriss für jeden Gierwinkel, kleinstes und größtes x der Ecken
fahrzeuge = []
x_links = []
x_rechts = []
for k in range(len(psi_grad)):
    T_WF_k = transformationsmatrix(np.deg2rad(psi_grad[k]), 5.0, 5.0)
    ecken = transformiere(T_WF_k, umriss)
    fahrzeuge.append(ecken)
    x_links.append(np.min(ecken[:, 0]))
    x_rechts.append(np.max(ecken[:, 0]))

k_links = np.argmin(x_links)
k_rechts = np.argmax(x_rechts)

# Ausgabe
print(f'Am weitesten links:  x = {x_links[k_links]:.3f} m bei ψ = {psi_grad[k_links]:.0f}°')
print(f'Am weitesten rechts: x = {x_rechts[k_rechts]:.3f} m bei ψ = {psi_grad[k_rechts]:.0f}°')
# Schon eine Berührung zählt als Kollision
print('Stößt an Regal 2:', x_links[k_links] <= 4.3)
print('Stößt an Regal 3:', x_rechts[k_rechts] >= 5.7)
zeichne_halle(fahrzeuge, umgebung=[regal_2, regal_3],
              titel='Wenden im Regalgang', bereich=(3.5, 6.5, 3.5, 6.5))
```
Ausgabe:

```text
Am weitesten links:  x = 4.280 m bei ψ = 150°
Am weitesten rechts: x = 5.720 m bei ψ = 150°
Stößt an Regal 2: True
Stößt an Regal 3: True
```

In unserer Liste ragt das FTF bei $\psi = 150°$ am weitesten hinaus, bei
$\psi = 210°$ noch einmal genauso. Das sind aber nur die äußersten Werte
unseres $10°$-Rasters. Jede Ecke ist $\sqrt{0.6^2 + 0.4^2} = 0.721\,\text{m}$
von der Fahrzeugmitte entfernt. Beim Drehen auf der Stelle überstreichen die
Ecken deshalb einen Kreis mit diesem Radius. Am weitesten nach außen reichen
sie, wenn eine Diagonale des FTF genau waagerecht liegt. Die Diagonalen sind um
$33.7°$ gegen die $x_F$-Achse geneigt, denn $\tan 33.7° = 0.4 / 0.6$.
Waagerecht liegen sie deshalb bei $\psi = 180° - 33.7° = 146.3°$ und bei
$\psi = 180° + 33.7° = 213.7°$. Dann reichen die Ecken genau bis
$x = 5 - 0.721 = 4.279\,\text{m}$ und $x = 5 + 0.721 = 5.721\,\text{m}$. Das
FTF ragt also auf beiden Seiten $2.1\,\text{cm}$ in die Regale hinein und kann
im Regalgang nicht wenden. Der Gang müsste mindestens
$2 \cdot 0.721\,\text{m} = 1.44\,\text{m}$ breit sein, mit dem
Sicherheitsabstand $1.64\,\text{m}$. Unser Gang hat nur $1.4\,\text{m}$. Das
FTF muss rückwärts aus dem Gang fahren oder den Gang am anderen Ende
verlassen. Schon eine Berührung zählt als Kollision, deshalb vergleichen wir
mit `<=` und `>=`.

Die Regalseiten sind Geraden $x = 4.3$ und $x = 5.7$ parallel zur
$y_W$-Achse. In Hallenkoordinaten genügt deshalb ein Vergleich der
$x$-Koordinaten. Das gilt hier allerdings nur, weil das FTF beim Drehen ganz
im $y$-Bereich der Regale bleibt: Seine Ecken liegen zwischen
$y = 4.28\,\text{m}$ und $y = 5.72\,\text{m}$, die Regale reichen von
$y = 2.4\,\text{m}$ bis $y = 7.5\,\text{m}$. Am Ende eines Regals müssten wir
zusätzlich prüfen, ob sich FTF und Regal auch in $y$-Richtung überdecken.
Im Fahrzeugkoordinatensystem lägen die Regalseiten schräg und
bei jedem Gierwinkel anders. In Kapitel 6.4 war es umgekehrt: Die Regalecke ist
ein einzelner Punkt, und dort liegen die Fahrzeugseiten im
Fahrzeugkoordinatensystem parallel zu den Achsen. Wir prüfen also immer in dem
Koordinatensystem, in dem die Grenze parallel zu den Achsen liegt.
````
