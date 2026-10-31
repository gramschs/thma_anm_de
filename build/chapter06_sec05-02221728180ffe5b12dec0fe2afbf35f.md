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
Umriss, Hallenkarte, Übergabestation und Schutzfeld aus den Kapiteln 6.1 und
6.2.

Der Schwierigkeitsgrad steht im Titel jeder Aufgabe:

* ✩ Verständnis: Code und Ausgaben vorhersagen und erklären (ca. 5 min)
* ✩✩ Anwendung: eigenen Code schreiben und Ergebnisse interpretieren (ca. 10 min)
* ✩✩✩ Mini-Projekt: mehrere Konzepte des Parts kombinieren (ca. 30 min)

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

# Schutzfeld aus Kapitel 6.2 im Fahrzeugkoordinatensystem in m
schutzfeld = np.array([
    [0.6, -0.5],
    [1.6, -0.5],
    [1.6,  0.5],
    [0.6,  0.5],
])
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
drei haben wie in Kapitel 6.3 einen Differentialantrieb. Für ihre Abschätzung
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

```{admonition} Aufgabe 6.6 (✩✩)
:class: tip
Am Ende von Kapitel 5.3 haben wir angekündigt, dass Robotersteuerungen jedem
Arm ein eigenes Koordinatensystem geben. Das holen wir jetzt für den
SCARA-Roboter aus Part 5 nach. Arm 1 ist $l_1 = 0.4\,\text{m}$ lang, Arm 2
$l_2 = 0.3\,\text{m}$. Wie das FTF und der Scanner bekommt jeder Arm ein
eigenes Koordinatensystem:

* Koordinatensystem 0 ist fest mit dem Sockel verbunden. Es ist das
  Koordinatensystem aus Part 5 mit dem Ursprung in Gelenk 1.
* Koordinatensystem 1 ist fest mit Arm 1 verbunden. Sein Ursprung liegt in
  Gelenk 1, die $x_1$-Achse zeigt entlang Arm 1 zu Gelenk 2.
* Koordinatensystem 2 ist fest mit Arm 2 verbunden. Sein Ursprung liegt in
  Gelenk 2, die $x_2$-Achse zeigt entlang Arm 2 zum Greifer.

1. Beantworten Sie ohne Code: Welche Koordinaten hat Gelenk 2 im
   Koordinatensystem 1, welche hat der Greifer im Koordinatensystem 2? Ändern
   sie sich, wenn sich die Gelenke drehen? Begründen Sie damit, dass
   $\mathbf{T}_{01}$ = `transformationsmatrix(alpha, 0, 0)` und
   $\mathbf{T}_{12}$ = `transformationsmatrix(beta, 0.4, 0)` gilt.
2. Schreiben Sie eine Funktion `kette(alpha, beta)` mit Docstring, die
   $\mathbf{T}_{02} = \mathbf{T}_{01} \cdot \mathbf{T}_{12}$ zurückgibt.
   Berechnen Sie für $\alpha = 30°$ und $\beta = 60°$ die Lage des Greifers im
   Koordinatensystem 0 und vergleichen Sie mit Kapitel 5.3. Bestimmen Sie mit
   `np.arctan2` aus der ersten Spalte von $\mathbf{T}_{02}$, in welche Richtung
   Arm 2 zeigt.
3. In Kapitel 5.4 hat die Rastersuche für den Bestückplatz
   $P_1 = (0.30, 0.35)$ die Winkel $\alpha = 9°$ und $\beta = 99°$ geliefert.
   Rechnen Sie $P_1$ mit der inversen Matrix in das Koordinatensystem 2 um. Wie
   weit liegt $P_1$ entlang Arm 2 und quer dazu neben dem Greifer, und wie groß
   ist der Abstand insgesamt?
4. Beantworten Sie ohne Code: Der Roboter bekommt am Greifer eine dritte
   Drehachse mit dem Winkel $\gamma$. Wie verlängert sich die Kette?

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
def kette(alpha, beta):
    """Matrix T_02 von Koordinatensystem 2 (Arm 2) nach 0 (Sockel), Winkel in Radiant."""
    # Koordinatensystem 1 ist gegenüber dem Sockel um alpha gedreht
    T_01 = transformationsmatrix(alpha, 0, 0)
    # Koordinatensystem 2 liegt in Gelenk 2 bei (0.4, 0) und ist um beta gedreht
    T_12 = transformationsmatrix(beta, 0.4, 0)
    return T_01 @ T_12

greifer_2 = np.array([[0.3, 0.0]])   # Greifer im Koordinatensystem 2 in m
p1 = np.array([[0.30, 0.35]])        # Bestückplatz P1 im Koordinatensystem 0 in m

# Verarbeitung Teilaufgabe 2: Greifer bei alpha = 30 und beta = 60 Grad
T_02 = kette(np.deg2rad(30), np.deg2rad(60))
greifer_0 = transformiere(T_02, greifer_2)
richtung_arm2 = np.rad2deg(np.arctan2(T_02[1, 0], T_02[0, 0]))

# Verarbeitung Teilaufgabe 3: P1 aus Sicht von Arm 2 bei alpha = 9 und beta = 99 Grad
T_02_p1 = kette(np.deg2rad(9), np.deg2rad(99))
p1_2 = transformiere(np.linalg.inv(T_02_p1), p1)
differenz = p1_2[0] - greifer_2[0]
abstand = np.sqrt(differenz[0]**2 + differenz[1]**2)

# Ausgabe
print('T_02 für α = 30°, β = 60°:')
print(np.round(T_02, 4))
print('Greifer im Koordinatensystem 0 in m:', np.round(greifer_0, 4))
print(f'Richtung von Arm 2: {richtung_arm2:.1f}°')
print('P1 im Koordinatensystem 2 in m:', np.round(p1_2, 4))
print(f'P1 neben dem Greifer: {differenz[0] * 1000:.1f} mm entlang Arm 2, '
      f'{differenz[1] * 1000:.1f} mm quer dazu, Abstand {abstand * 1000:.1f} mm')
```
Ausgabe:

```text
T_02 für α = 30°, β = 60°:
[[ 0.     -1.      0.3464]
 [ 1.      0.      0.2   ]
 [ 0.      0.      1.    ]]
Greifer im Koordinatensystem 0 in m: [[0.3464 0.5   ]]
Richtung von Arm 2: 90.0°
P1 im Koordinatensystem 2 in m: [[0.3027 0.0016]]
P1 neben dem Greifer: 2.7 mm entlang Arm 2, 1.6 mm quer dazu, Abstand 3.2 mm
```

Gelenk 2 liegt im Koordinatensystem 1 immer bei $(0.4, 0)$, der Greifer im
Koordinatensystem 2 immer bei $(0.3, 0)$, denn beide Koordinatensysteme drehen
sich mit ihrem Arm mit. Koordinatensystem 1 hat denselben Ursprung wie der
Sockel und ist um $\alpha$ gedreht, daher
$\mathbf{T}_{01}$ = `transformationsmatrix(alpha, 0, 0)`. Koordinatensystem 2
hat seinen Ursprung in Gelenk 2, also bei $(0.4, 0)$ im Koordinatensystem 1,
und ist gegenüber Arm 1 um $\beta$ gedreht. Das ergibt
$\mathbf{T}_{12}$ = `transformationsmatrix(beta, 0.4, 0)`.

Die Spalten von $\mathbf{T}_{02}$ lesen wir wie bei $\mathbf{T}_{WF}$ in
Kapitel 6.1. Die dritte Spalte $(0.346, 0.2)$ ist der Ursprung von
Koordinatensystem 2, also Gelenk 2, genau wie in Kapitel 5.3. Die erste Spalte
$(0, 1)$ ist die Richtung von Arm 2, um $\alpha + \beta = 90°$ gedreht. Der
Greifer liegt $0.3\,\text{m}$ weiter in dieser Richtung bei $(0.346, 0.5)$,
wie in Kapitel 5.3. Anders als dort brauchen wir aber keine Drehung um einen
Punkt mehr, weil jedes Koordinatensystem direkt in seinem Gelenk sitzt.

Aus Sicht von Arm 2 liegt $P_1$ bei $(0.3027, 0.0016)$. Der Greifer müsste also
noch $2.7\,\text{mm}$ weiter nach außen und $1.6\,\text{mm}$ quer zu Arm 2
nach links. Der Abstand von $3.2\,\text{mm}$ ist derselbe wie in Kapitel 5.4.

Eine dritte Drehachse am Greifer bekommt ein Koordinatensystem 3 mit dem
Ursprung im Greifer, also bei $(0.3, 0)$ im Koordinatensystem 2, um $\gamma$
gedreht:
$\mathbf{T}_{03} = \mathbf{T}_{01} \cdot \mathbf{T}_{12} \cdot \mathbf{T}_{23}$
mit $\mathbf{T}_{23}$ = `transformationsmatrix(gamma, 0.3, 0)`. Jedes weitere
Gelenk hängt eine Matrix rechts an, und jede Matrix hängt nur von ihrem
eigenen Gelenk ab. Der Ursprung von Koordinatensystem 3 bleibt beim Drehen um
$\gamma$ an seinem Platz. Solange kein Werkzeug über den Greifer hinausragt,
ändert die dritte Achse deshalb nur die Orientierung des Bauteils, nicht die
Lage der Greiferspitze.
````

````{admonition} Aufgabe 6.7 (✩✩)
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

```{admonition} Aufgabe 6.8 (✩✩)
:class: tip
Im Hauptgang fahren zwei FTF aufeinander zu. FTF A steht bei $(2.0, 1.5)$ mit
dem Gierwinkel $\psi_A = 10°$, FTF B bei $(4.0, 2.2)$ mit $\psi_B = 200°$.
Beide haben einen Sicherheitsscanner mit dem Schutzfeld `schutzfeld` aus
Kapitel 6.2, also $0.6\,\text{m} \leq x_F \leq 1.6\,\text{m}$ und
$-0.5\,\text{m} \leq y_F \leq 0.5\,\text{m}$ im eigenen
Fahrzeugkoordinatensystem.

1. Bauen Sie $\mathbf{T}_{WA}$ und $\mathbf{T}_{WB}$ auf. Bilden Sie die Matrix
   $\mathbf{T}_{AB}$, die Koordinaten aus dem Fahrzeugkoordinatensystem von B in
   das von A umrechnet. Setzen Sie sie wie $\mathbf{T}_{WS}$ in Kapitel 6.2 aus
   zwei Matrizen zusammen, deren Indizes wie Kettenglieder ineinandergreifen.
2. Wo liegt die Mitte von B aus Sicht von A? Bestimmen Sie mit `np.arctan2`
   aus der ersten Spalte von $\mathbf{T}_{AB}$, um welchen Winkel B gegenüber A
   gedreht ist, und vergleichen Sie mit $\psi_B - \psi_A$.
3. Rechnen Sie den Umriss von B in das Fahrzeugkoordinatensystem von A um.
   Schreiben Sie eine Funktion `ecken_im_schutzfeld(ecken_F)` mit Docstring, die
   die Nummern der Ecken als Liste zurückgibt, die im Schutzfeld liegen. Welche
   Ecken von B liegen im Schutzfeld von A? Welche Ecken von A liegen im
   Schutzfeld von B, und welche Matrix brauchen Sie dafür? Müssen beide FTF
   anhalten?
4. Zeichnen Sie beide FTF mit dem Schutzfeld von A in der Halle,
   `bereich=(0, 6, 0, 4)`.

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
T_WA = transformationsmatrix(np.deg2rad(10), 2.0, 1.5)
T_WB = transformationsmatrix(np.deg2rad(200), 4.0, 2.2)

def ecken_im_schutzfeld(ecken_F):
    """Nummern der Ecken im Schutzfeld, Ecken in Fahrzeugkoordinaten, Zeile = [x, y]."""
    nummern = []
    for k in range(len(ecken_F)):
        x_F = ecken_F[k, 0]
        y_F = ecken_F[k, 1]
        if x_F >= 0.6 and x_F <= 1.6 and y_F >= -0.5 and y_F <= 0.5:
            nummern.append(k)
    return nummern

# Verarbeitung Teilaufgabe 1 und 2: von B über W nach A
T_AB = np.linalg.inv(T_WA) @ T_WB
winkel_AB = np.rad2deg(np.arctan2(T_AB[1, 0], T_AB[0, 0]))

# Verarbeitung Teilaufgabe 3: Umriss von B bei A, Umriss von A bei B
T_BA = np.linalg.inv(T_AB)
ecken_B_in_A = transformiere(T_AB, umriss)
ecken_A_in_B = transformiere(T_BA, umriss)

# Ausgabe
print('T_AB:')
print(np.round(T_AB, 4))
print(f'B ist gegenüber A um {winkel_AB:.1f}° gedreht')
print('Ecken von B im Fahrzeugkoordinatensystem von A in m:')
print(np.round(ecken_B_in_A, 3))
print('Ecken von B im Schutzfeld von A:', ecken_im_schutzfeld(ecken_B_in_A))
print('Ecken von A im Schutzfeld von B:', ecken_im_schutzfeld(ecken_A_in_B))

zeichne_halle([transformiere(T_WA, umriss), transformiere(T_WB, umriss)],
              schutzfeld=transformiere(T_WA, schutzfeld), umgebung=halle,
              beschriftungen=['FTF A', 'FTF B'], titel='Begegnung im Hauptgang',
              bereich=(0, 6, 0, 4))
```
Ausgabe:

```text
T_AB:
[[-0.9848  0.1736  2.0912]
 [-0.1736 -0.9848  0.3421]
 [ 0.      0.      1.    ]]
B ist gegenüber A um -170.0° gedreht
Ecken von B im Fahrzeugkoordinatensystem von A in m:
[[ 1.431  0.632]
 [ 1.57  -0.156]
 [ 2.752  0.052]
 [ 2.613  0.84 ]]
Ecken von B im Schutzfeld von A: [1]
Ecken von A im Schutzfeld von B: [0, 1]
```

$\mathbf{T}_{AB} = \mathbf{T}_{AW} \cdot \mathbf{T}_{WB}$ rechnet zuerst von B
in die Halle und dann von der Halle nach A. Dabei ist
$\mathbf{T}_{AW} = \mathbf{T}_{WA}^{-1}$ wie $\mathbf{T}_{FW}$ in Kapitel 6.1.
Die dritte Spalte zeigt: Die Mitte von B liegt aus Sicht von A
$2.09\,\text{m}$ voraus und $0.34\,\text{m}$ links. B ist gegenüber A um
$-170°$ gedreht. Das ist dieselbe Richtung wie
$\psi_B - \psi_A = 200° - 10° = 190°$, die beiden Werte unterscheiden sich um
$360°$ wie in Aufgabe 6.4. Die beiden FTF kommen sich also fast genau entgegen.

Die vordere linke Ecke von B (Nummer 1) liegt bei $(1.57, -0.156)$ im
Schutzfeld von A. Für die Gegenrichtung brauchen wir
$\mathbf{T}_{BA} = \mathbf{T}_{AB}^{-1}$, und beide vorderen Ecken von A
liegen im Schutzfeld von B. Beide FTF müssen anhalten, und die Leitsteuerung
muss eines von ihnen umleiten. Unsere Prüfung ist vereinfacht. Eine Kante
kann quer durch das Schutzfeld laufen, ohne dass eine Ecke darin liegt. Der
echte Scanner misst dagegen Punkte auf den Kanten und würde auch diesen Fall
erkennen.
````

````{admonition} Aufgabe 6.9 (✩✩✩) Mini-Projekt: Andocken an die Ladestation
:class: tip
Das FTF soll zum Laden an eine Ladestation an der linken Hallenwand bei
$x = 0$ fahren. Die Ladekontakte sitzen an der Vorderkante des FTF und an der
Vorderseite der Station. Damit sie sich treffen, muss das FTF die
**Andockpose** genau erreichen. Die Pose, die seine Steuerung aus den
Drehungen der Räder berechnet, ist dafür nach einer längeren Fahrt zu
ungenau. Links und rechts der Station hängen deshalb zwei Reflektoren an der
Wand, die der Navigationsscanner in der Fahrzeugmitte misst:

| Objekt | Lage |
| --- | --- |
| Reflektor A | $(0, 2)$ in der Halle, gemessen: $r = 3.41\,\text{m}$ unter $\varphi = 21.6°$ |
| Reflektor B | $(0, 4)$ in der Halle, gemessen: $r = 4.05\,\text{m}$ unter $\varphi = -7.9°$ |
| Ladestation | $x$ von $0$ bis $0.15\,\text{m}$, $y$ von $2.6$ bis $3.4\,\text{m}$ |
| Vorpunkt | Pose $(1.75, 3.0, 180°)$, $1\,\text{m}$ vor der Andockpose |
| Andockpose | $(0.75, 3.0, 180°)$, die Vorderkante liegt an der Station |

Die Daten als Code, dazu ein Stück Wand für die Zeichnung:

```python
reflektoren_W = np.array([[0.0, 2.0], [0.0, 4.0]])   # Reflektor A, Reflektor B
entfernungen = np.array([3.41, 4.05])
winkel_grad = np.array([21.6, -7.9])
ladestation = np.array([[0.0, 2.6], [0.15, 2.6], [0.15, 3.4], [0.0, 3.4]])
wand = np.array([[-0.3, 0.0], [0.0, 0.0], [0.0, 6.0], [-0.3, 6.0]])
```

**Teil 1:** Rechnen Sie die beiden Messungen wie in Kapitel 6.1 in
Fahrzeugkoordinaten um. Die Verbindungslinie von Reflektor A zu Reflektor B
zeigt in der Halle in Richtung $90°$. Bestimmen Sie mit `np.arctan2` ihre
Richtung im Fahrzeugkoordinatensystem. Das Fahrzeugkoordinatensystem ist
gegenüber der Halle um $\psi$ gedreht, deshalb ist der Gierwinkel die
Differenz der beiden Richtungen.

**Teil 2:** $\mathbf{T}_{WF}$ dreht zuerst um $\psi$ und verschiebt dann um
die Lage der Fahrzeugmitte. Drehen Sie deshalb Reflektor A mit
`transformationsmatrix(psi, 0, 0)`. Die Verschiebung, die den gedrehten Punkt
auf die Lage von A laut Karte bringt, ist die Fahrzeugmitte. Bauen Sie
$\mathbf{T}_{WF}$ auf und machen Sie die Probe: Wie weit liegt Reflektor B,
mit $\mathbf{T}_{WF}$ in die Halle umgerechnet, neben seiner Lage laut Karte?

**Teil 3:** Das FTF fährt in drei Schritten zum Vorpunkt: auf der Stelle zum
Vorpunkt drehen, geradeaus fahren, auf der Stelle auf $180°$ drehen. Danach
fährt es $1\,\text{m}$ geradeaus an die Station. Rechnen Sie den Vorpunkt mit
der inversen Matrix in Fahrzeugkoordinaten um. Bestimmen Sie daraus die erste
Drehung $\varphi_1$, die Strecke und die zweite Drehung $\varphi_2$. Jeder
Schritt beschreibt wie $\mathbf{T}_{FS}$ in Kapitel 6.2 ein neues
Fahrzeugkoordinatensystem relativ zum aktuellen, zum Beispiel
`transformationsmatrix(0, strecke, 0)` für die Fahrt. Hängen Sie die Schritte
deshalb von rechts an $\mathbf{T}_{WF}$ an. Prüfen Sie mit `np.allclose`, ob
das FTF am Vorpunkt und an der Andockpose ankommt. Zeichnen Sie das FTF am
Start, nach der ersten Drehung, am Vorpunkt und an der Andockpose, dazu Wand,
Ladestation, Reflektoren und die Bahn.

**Abschlussfrage:** Warum fährt das FTF nicht direkt zur Andockpose und dreht
sich erst dort auf $180°$? Denken Sie an Aufgabe 6.7. Angenommen, der Scanner
misst Reflektor B um $1\,\text{cm}$ quer zur Verbindungslinie verschoben. Um
wie viel Grad irrt sich dann der Gierwinkel ungefähr? Wie ändert sich dieser
Fehler, wenn die Reflektoren nur $0.5\,\text{m}$ auseinander hängen?

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
reflektoren_W = np.array([[0.0, 2.0], [0.0, 4.0]])   # Reflektor A, Reflektor B
entfernungen = np.array([3.41, 4.05])
winkel_grad = np.array([21.6, -7.9])
ladestation = np.array([[0.0, 2.6], [0.15, 2.6], [0.15, 3.4], [0.0, 3.4]])
wand = np.array([[-0.3, 0.0], [0.0, 0.0], [0.0, 6.0], [-0.3, 6.0]])
T_WV = transformationsmatrix(np.deg2rad(180), 1.75, 3.0)   # Vorpunkt
T_WD = transformationsmatrix(np.deg2rad(180), 0.75, 3.0)   # Andockpose

# Verarbeitung Teil 1: Reflektoren in Fahrzeugkoordinaten
winkel = np.deg2rad(winkel_grad)
reflektoren_F = np.zeros((2, 2))
reflektoren_F[:, 0] = entfernungen * np.cos(winkel)
reflektoren_F[:, 1] = entfernungen * np.sin(winkel)

# Gierwinkel: Richtung von A nach B in der Halle minus Richtung im Fahrzeug
richtung_W = reflektoren_W[1] - reflektoren_W[0]
richtung_F = reflektoren_F[1] - reflektoren_F[0]
psi = np.arctan2(richtung_W[1], richtung_W[0]) - np.arctan2(richtung_F[1], richtung_F[0])

# Verarbeitung Teil 2: Fahrzeugmitte, die den gedrehten Reflektor A auf die Karte bringt
T_dreh = transformationsmatrix(psi, 0, 0)
a_gedreht = transformiere(T_dreh, reflektoren_F[0:1])[0]
mitte = reflektoren_W[0] - a_gedreht
T_WF = transformationsmatrix(psi, mitte[0], mitte[1])

# Probe: beide Reflektoren mit T_WF in die Halle umrechnen
probe = transformiere(T_WF, reflektoren_F)
differenz = probe[1] - reflektoren_W[1]
abweichung_B = np.sqrt(differenz[0]**2 + differenz[1]**2)

# Verarbeitung Teil 3: Vorpunkt aus Sicht des Fahrzeugs
T_FV = np.linalg.inv(T_WF) @ T_WV
x_V = T_FV[0, 2]
y_V = T_FV[1, 2]

# Drehen, fahren, drehen, andocken
phi_1 = np.arctan2(y_V, x_V)
strecke = np.sqrt(x_V**2 + y_V**2)
phi_2 = np.deg2rad(180) - psi - phi_1
T_dreh_1 = transformationsmatrix(phi_1, 0, 0)
T_fahrt = transformationsmatrix(0, strecke, 0)
T_dreh_2 = transformationsmatrix(phi_2, 0, 0)
T_andocken = transformationsmatrix(0, 1.0, 0)

# Jeder Schritt im aktuellen Fahrzeugkoordinatensystem, also von rechts
T_nach_drehung = T_WF @ T_dreh_1
T_am_vorpunkt = T_nach_drehung @ T_fahrt @ T_dreh_2
T_angedockt = T_am_vorpunkt @ T_andocken

# Ausgabe
print('Reflektoren in Fahrzeugkoordinaten in m:')
print(np.round(reflektoren_F, 3))
print(f'Gierwinkel: {np.rad2deg(psi):.1f}°')
print('Fahrzeugmitte in m:', np.round(mitte, 3))
print(f'Probe: Reflektor B liegt {abweichung_B * 1000:.1f} mm neben der Karte')
print('Vorpunkt in Fahrzeugkoordinaten in m:', np.round([x_V, y_V], 3))
print(f'Manöver: {np.rad2deg(phi_1):.1f}° drehen, {strecke:.2f} m fahren, '
      f'{np.rad2deg(phi_2):.1f}° drehen, 1 m andocken')
print('Am Vorpunkt angekommen:', np.allclose(T_am_vorpunkt, T_WV))
print('An der Station angedockt:', np.allclose(T_angedockt, T_WD))

fahrzeuge = []
for T in [T_WF, T_nach_drehung, T_am_vorpunkt, T_angedockt]:
    fahrzeuge.append(transformiere(T, umriss))
bahn_andocken = np.array([mitte, [1.75, 3.0], [0.75, 3.0]])
zeichne_halle(fahrzeuge, messpunkte=reflektoren_W, scanner=mitte,
              bahn=bahn_andocken, umgebung=[wand, ladestation],
              beschriftungen=['Start', 'nach Drehung 1', 'Vorpunkt', 'angedockt'],
              titel='Andocken an die Ladestation', bereich=(-0.5, 4.5, 0.5, 6.5))
```
Ausgabe:

```text
Reflektoren in Fahrzeugkoordinaten in m:
[[ 3.171  1.255]
 [ 4.012 -0.557]]
Gierwinkel: 155.1°
Fahrzeugmitte in m: [3.404 1.804]
Probe: Reflektor B liegt 2.4 mm neben der Karte
Vorpunkt in Fahrzeugkoordinaten in m: [ 2.004 -0.389]
Manöver: -11.0° drehen, 2.04 m fahren, 35.9° drehen, 1 m andocken
Am Vorpunkt angekommen: True
An der Station angedockt: True
```

Im Fahrzeugkoordinatensystem zeigt die Verbindungslinie von A nach B in
Richtung $-65.1°$, in der Halle in Richtung $90°$. Jede Richtung im
Fahrzeugkoordinatensystem erscheint in der Halle um $\psi$ weitergedreht.
Das Fahrzeugkoordinatensystem ist also um $90° - (-65.1°) = 155.1°$ gegen die
Halle gedreht. Für die Fahrzeugmitte erhalten wir
$(3.404, 1.804)$. Reflektor A passt nach Konstruktion genau, Reflektor B liegt
bei der Probe $2.4\,\text{mm}$ neben seiner Lage laut Karte. Der Rest stammt
aus den gerundeten Messwerten. Die Messwerte gehören zur Pose
$(3.4, 1.8, 155°)$, unsere Schätzung weicht davon um rund $6\,\text{mm}$ und
$0.1°$ ab.

Der Vorpunkt liegt aus Sicht des FTF $2.0\,\text{m}$ voraus und
$0.39\,\text{m}$ rechts. Das FTF dreht sich deshalb zuerst um $11.0°$ nach
rechts, fährt $2.04\,\text{m}$ geradeaus und steht dann mit
$155.1° - 11.0° = 144.1°$ am Vorpunkt. Die zweite Drehung um $35.9°$ nach
links richtet es auf $180°$ aus. Die Kette aus vier Schritten landet genau am
Vorpunkt und an der Andockpose. Im Bild fährt das FTF den letzten Meter
gerade auf die Station zu.

**Abschlussfrage:** Beim Drehen auf der Stelle überstreichen die Ecken einen
Kreis mit $0.721\,\text{m}$ Radius, wie in Aufgabe 6.7. An der Andockpose liegt
die Fahrzeugmitte nur $0.75\,\text{m}$ vor der Wand. Die Ecken würden bis
$x = 0.03\,\text{m}$ reichen und die Station rammen, die bis
$x = 0.15\,\text{m}$ in die Halle ragt. Am Vorpunkt ist genug Platz für die
Drehung. Danach fährt das FTF geradeaus und senkrecht auf die Station zu, so
treffen die Ladekontakte gerade aufeinander. Viele FTF messen am Vorpunkt die
Reflektoren außerdem noch einmal und korrigieren ihre Pose vor dem letzten
Meter.

Liegt Reflektor B $1\,\text{cm}$ quer zur Verbindungslinie daneben, dreht sich
die gemessene Linie um ungefähr $0.01\,\text{m} / 2\,\text{m} = 0.005$ Radiant,
also um $0.29°$. Wie beim Strahlabstand in Kapitel 6.2 ist der kurze Bogen
dabei fast so lang wie die gerade Strecke. Teil 2 dreht Reflektor A mit diesem
falschen Gierwinkel, deshalb liegt auch die berechnete Fahrzeugmitte daneben,
um ungefähr $3.41\,\text{m} \cdot 0.005 = 1.7\,\text{cm}$. Hängen die
Reflektoren nur $0.5\,\text{m}$ auseinander, wird der Winkelfehler viermal so
groß, rund $1.15°$. Reflektoren für die Lokalisierung hängen deshalb möglichst
weit auseinander.
````
