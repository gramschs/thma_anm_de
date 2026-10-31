---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 6.4 Einfahrt in einen Regalgang

In Kapitel 6.3 haben wir aus Bahnpunkten den Gierwinkel berechnet und das FTF
entlang einer Bahn gezeichnet. In diesem Projekt biegt das FTF aus dem
Hauptgang in einen Regalgang ein, und wir prüfen, ob die geplante Kurve genug
Abstand zu den Regalen lässt. Bearbeiten Sie die Teilaufgaben möglichst zu
zweit und der Reihe nach, jeder Teil baut auf dem vorherigen auf.

````{admonition} Projekt: Passt das FTF um die Regalecke? (✩✩)
:class: tip
In einem anderen Teil der Halle stehen vier Regale quer zur Hallenwand.
Zwischen ihnen liegen Regalgänge, davor verläuft der Hauptgang. Das FTF soll
eine Kiste aus dem Regalgang zwischen Regal 2 und Regal 3 holen. Es fährt im
Hauptgang nach rechts und biegt auf einem Viertelkreis nach links in den
Regalgang ein. Damit das FTF zügig um die Kurve fahren kann, plant die
Leitsteuerung einen großen Radius von $2\,\text{m}$. Die Betreiberin der
Halle verlangt aber, dass zwischen FTF und Regal immer mindestens
$0.1\,\text{m}$ frei bleiben. Kritisch ist die **Regalecke** $E$ von Regal 2,
um die das FTF herumfährt.

```{figure} pics/chap06_regalgang.svg
:alt: Draufsicht auf Hauptgang und Regalgang mit Regal 2 und Regal 3, dem FTF im Hauptgang und der gestrichelten Bahn, die geradeaus fährt und auf einem Viertelkreis mit Radius 2 m um den Mittelpunkt M nach links in den 1.4 m breiten Regalgang abbiegt, dazu die Regalecke E an der rechten unteren Ecke von Regal 2
:align: center

Hauptgang und Regalgang mit der geplanten Bahn, dem Mittelpunkt $M$ des
Viertelkreises und der Regalecke $E$.
(Quelle: eigene Abbildung; Lizenz [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0))
```

| Objekt | Lage und Größe |
| --- | --- |
| FTF | $1.2\,\text{m} \times 0.8\,\text{m}$ wie in Kapitel 6.1 |
| Regale | je $1.3\,\text{m}$ breit, von $y = 2.4\,\text{m}$ bis $y = 7.5\,\text{m}$, Regal 2 von $x = 3.0\,\text{m}$ bis $x = 4.3\,\text{m}$, Regal 3 von $x = 5.7\,\text{m}$ bis $x = 7.0\,\text{m}$ |
| Regalgang | zwischen Regal 2 und Regal 3, $1.4\,\text{m}$ breit, Mitte bei $x = 5.0\,\text{m}$ |
| Bahn | Gerade im Hauptgang bei $y = 1.2\,\text{m}$ bis $(3, 1.2)$, Viertelkreis um den Mittelpunkt $M = (3, 3.2)$ mit Radius $2\,\text{m}$ bis $(5, 3.2)$, Gerade im Regalgang bis $(5, 6.2)$ |
| Regalecke $E$ | rechte untere Ecke von Regal 2 bei $(4.3, 2.4)$ |
| Sicherheitsabstand | mindestens $0.1\,\text{m}$ zwischen FTF und Regal |
````

Führen Sie zuerst die beiden folgenden Zellen aus. Die erste enthält die
vorgegebene Zeichenfunktion aus Kapitel 6.1. Die zweite enthält die Importe,
die Funktionen `transformationsmatrix` und `transformiere` aus Kapitel 5.3,
den Umriss aus Kapitel 6.1, die vier Regale und die Regalecke. Dazu kommt die
Bahn. Die Zelle setzt sie aus drei Stücken zusammen und sammelt die
Bahnpunkte dabei wie die Umrisse in Kapitel 6.3 in einer Liste. Die Punkte des
Viertelkreises berechnet sie mit Polarkoordinaten wie in Kapitel 6.1, nur um
den Mittelpunkt des Kreises verschoben. Den Mittelpunkt legt sie aus dem
Radius so fest, dass die Kurve in der Mitte des Regalgangs endet.

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

# Hallenkarte: vier Regale in Hallenkoordinaten in m
regal_1 = np.array([[0.3, 2.4], [1.6, 2.4], [1.6, 7.5], [0.3, 7.5]])
regal_2 = np.array([[3.0, 2.4], [4.3, 2.4], [4.3, 7.5], [3.0, 7.5]])
regal_3 = np.array([[5.7, 2.4], [7.0, 2.4], [7.0, 7.5], [5.7, 7.5]])
regal_4 = np.array([[8.4, 2.4], [9.7, 2.4], [9.7, 7.5], [8.4, 7.5]])
halle = [regal_1, regal_2, regal_3, regal_4]

# Regalecke: rechte untere Ecke von Regal 2, als Array der Form (1, 2)
ecke = np.array([[4.3, 2.4]])

# Bahn in den Regalgang: Bahnpunkte in Hallenkoordinaten in m, Zeile = [x, y]
# Mittelpunkt des Viertelkreises: Die Kurve beginnt im Hauptgang bei y = 1.2 m
# und endet in der Gangmitte bei x = 5 m, für radius = 2 also bei (3.0, 3.2)
radius = 2.0
mitte_x = 5.0 - radius
mitte_y = 1.2 + radius
bahnpunkte = []
# Stück 1: Gerade im Hauptgang bei y = 1.2 m
for x_W in np.linspace(1.0, mitte_x - 0.5, 4):
    bahnpunkte.append([x_W, 1.2])
# Stück 2: Viertelkreis um (mitte_x, mitte_y), alle 10 Grad ein Punkt
for winkel in np.deg2rad(np.linspace(-90, 0, 10)):
    x_W = mitte_x + radius * np.cos(winkel)
    y_W = mitte_y + radius * np.sin(winkel)
    bahnpunkte.append([x_W, y_W])
# Stück 3: Gerade im Regalgang bei x = 5 m
for y_W in np.linspace(mitte_y + 0.5, 6.2, 6):
    bahnpunkte.append([5.0, y_W])
bahn = np.array(bahnpunkte)
print('Anzahl der Bahnpunkte:', len(bahn))

# FTF am Start der Bahn, Gierwinkel 0
umriss_start = transformiere(transformationsmatrix(0, 1.0, 1.2), umriss)
zeichne_halle(umriss_start, bahn=bahn, ziele=bahn[-1], umgebung=halle,
              titel='Ausgangslage: Bahn in den Regalgang')
```

```{admonition} Teil 1: Gierwinkel entlang der Bahn
:class: tip
Berechnen Sie wie im zweiten Abschnitt von Kapitel 6.3 den Gierwinkel an jedem
Bahnpunkt und speichern Sie ihn in einem Array `psi`. Geben Sie die Winkel in
Grad aus und zeichnen Sie sie über der Nummer des Bahnpunkts.

Welchen Gierwinkel hat das FTF auf den beiden Geraden? Um wie viel Grad dreht
es sich zwischen zwei Punkten des Viertelkreises? Bahnpunkt 4 bei $(3, 1.2)$
ist der Anfang des Viertelkreises. Warum zeigt das FTF dort schon mit $5°$
nach links, obwohl die Bahn an dieser Stelle noch waagerecht verläuft?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 1
:class: tip
:class: dropdown
Die Zelle aus dem zweiten Abschnitt von Kapitel 6.3 funktioniert für jede
Bahn, nur der Titel des Diagramms ändert sich.
```python
# Gierwinkel an jedem Bahnpunkt: Richtung zum nächsten Bahnpunkt
psi = np.zeros(len(bahn))
for k in range(___):
    richtung = ___
    psi[k] = np.arctan2(___, ___)

# Der letzte Bahnpunkt hat keinen Nachfolger und behält die letzte Richtung
psi[-1] = ___

psi_grad = np.rad2deg(psi)
print('Gierwinkel in Grad:', np.round(psi_grad, 1))

fig, ax = plt.subplots(figsize=(7, 4))
ax.plot(range(len(bahn)), psi_grad, marker='o')
ax.set_xlabel('Bahnpunkt k')
ax.set_ylabel('Gierwinkel in Grad')
ax.set_title('Gierwinkel entlang der Bahn in den Regalgang')
ax.grid(True)
plt.show()
```
````

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
```python
# Gierwinkel an jedem Bahnpunkt: Richtung zum nächsten Bahnpunkt
psi = np.zeros(len(bahn))
for k in range(len(bahn) - 1):
    richtung = bahn[k + 1] - bahn[k]
    psi[k] = np.arctan2(richtung[1], richtung[0])

# Der letzte Bahnpunkt hat keinen Nachfolger und behält die letzte Richtung
psi[-1] = psi[-2]

psi_grad = np.rad2deg(psi)
print('Gierwinkel in Grad:', np.round(psi_grad, 1))

fig, ax = plt.subplots(figsize=(7, 4))
ax.plot(range(len(bahn)), psi_grad, marker='o')
ax.set_xlabel('Bahnpunkt k')
ax.set_ylabel('Gierwinkel in Grad')
ax.set_title('Gierwinkel entlang der Bahn in den Regalgang')
ax.grid(True)
plt.show()
```
Ausgabe:

```text
Gierwinkel in Grad: [ 0.  0.  0.  0.  5. 15. 25. 35. 45. 55. 65. 75. 85. 90. 90. 90. 90. 90.
 90. 90.]
```
Auf der Geraden im Hauptgang fährt das FTF mit $0°$ in $x_W$-Richtung, im
Regalgang mit $90°$ in $y_W$-Richtung. Auf dem Viertelkreis liegen die
Bahnpunkte 4 bis 13. Von Bahnpunkt 4 bis 12 wächst der Gierwinkel von Punkt zu
Punkt um $10°$, genau um den Winkelschritt, in dem die Bahnpunkte auf dem Kreis
liegen. Die Winkel sind dort aber $5°$, $15°$ bis $85°$ und nicht $0°$, $10°$
bis $80°$, wie es die Tangenten an diesen Punkten wären. Wie in Kapitel 6.3 ist
`psi[k]` die Richtung der Sekante zum nächsten Bahnpunkt. Am Anfang des
Viertelkreises verläuft die Tangente waagerecht, die Sekante zum nächsten
Punkt auf dem Kreis steigt aber schon an. Auf einem Kreis liegt ihre Richtung
genau in der Mitte zwischen den Tangenten an ihren beiden Enden, also bei
$5°$. Auf den Kreisabschnitten eilt das FTF der Bahn deshalb um einen halben
Winkelschritt voraus. An Bahnpunkt 13 am Ende des Viertelkreises führt die
Sekante dagegen schon entlang der Geraden im Regalgang. Dort stimmt sie mit
der Tangente überein, und das FTF zeigt mit $90°$ genau in Richtung der Bahn.
````

```{admonition} Teil 2: Das FTF entlang der Bahn
:class: tip
Zeichnen Sie wie im dritten Abschnitt von Kapitel 6.3 den Umriss des FTF an
jedem Bahnpunkt. Sammeln Sie die Umrisse in einer Liste `fahrzeuge` und
zeigen Sie mit `bereich=(2, 7, 0, 4.5)` nur die Umgebung der Kurve.

An welcher Stelle kommt das FTF einem Regal am nächsten? Wie viel Platz bleibt
links und rechts, wenn das FTF geradeaus in der Mitte des Regalgangs fährt?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 2
:class: tip
:class: dropdown
Im dritten Abschnitt von Kapitel 6.3 haben wir für jeden Bahnpunkt aus
`bahn[k]` und `psi[k]` die Matrix $\mathbf{T}_{WF}$ aufgebaut und den
umgerechneten Umriss an eine Liste angehängt. Hier ändern sich nur die Bahn
und der Ausschnitt.
```python
# Leere Liste für die Umrisse an allen Bahnpunkten
fahrzeuge = []
for k in range(len(bahn)):
    # Pose am Bahnpunkt k: Lage aus der Bahn, Gierwinkel aus psi
    T_WF_k = transformationsmatrix(___, ___, ___)
    # Umriss in Hallenkoordinaten umrechnen und hinten an die Liste anhängen
    fahrzeuge.append(___)

zeichne_halle(fahrzeuge, bahn=bahn, umgebung=halle,
              titel='Fahrt in den Regalgang', bereich=___)
```
````

````{admonition} Lösung Teil 2
:class: tip
:class: dropdown
```python
# Leere Liste für die Umrisse an allen Bahnpunkten
fahrzeuge = []
for k in range(len(bahn)):
    # Pose am Bahnpunkt k: Lage aus der Bahn, Gierwinkel aus psi
    T_WF_k = transformationsmatrix(psi[k], bahn[k, 0], bahn[k, 1])
    # Umriss in Hallenkoordinaten umrechnen und hinten an die Liste anhängen
    fahrzeuge.append(transformiere(T_WF_k, umriss))

zeichne_halle(fahrzeuge, bahn=bahn, umgebung=halle,
              titel='Fahrt in den Regalgang', bereich=(2, 7, 0, 4.5))
```
Am engsten wird es in der Kurve an der Regalecke von Regal 2, etwa bei den
Bahnpunkten 8 bis 10. Dort berührt die linke Fahrzeugseite die Ecke fast. Zu
Regal 3 auf der Außenseite der Kurve bleibt deutlich mehr Platz. Im Regalgang
fährt das FTF in der Mitte bei $x = 5\,\text{m}$. Der Gang ist
$1.4\,\text{m}$ breit, das FTF $0.8\,\text{m}$, links und rechts bleiben also
je $(1.4\,\text{m} - 0.8\,\text{m}) / 2 = 0.3\,\text{m}$ frei. Wie viel Platz
an der Regalecke bleibt, können wir im Bild nicht genau ablesen.
````

```{admonition} Teil 3: Abstand zur Regalecke
:class: tip
Wir rechnen den Abstand zur Regalecke jetzt nach, und zwar wie die Prüfung des
Schutzfelds in Kapitel 6.2 im Fahrzeugkoordinatensystem. Dort ist der Umriss
an jedem Bahnpunkt dasselbe Rechteck mit $-0.6 \leq x_F \leq 0.6$ und
$-0.4 \leq y_F \leq 0.4$, seine Kanten liegen parallel zu den Achsen.

Rechnen Sie die Regalecke für jeden Bahnpunkt mit der inversen Matrix
$\mathbf{T}_{FW} = \mathbf{T}_{WF}^{-1}$ wie in Kapitel 6.1 in
Fahrzeugkoordinaten um. Die Regalecke liegt auf der Innenseite der
Linkskurve, also links vom FTF. Liegt sie links neben dem FTF, also bei
$-0.6 \leq x_F \leq 0.6$ und $y_F \geq 0$, dann ist $y_F - 0.4$ die Lücke
zwischen der linken Fahrzeugseite und der Ecke. Eine negative Lücke würde
bedeuten, dass die Ecke in das FTF hineinragt. Sammeln Sie die Lücken und die
Nummern der zugehörigen Bahnpunkte in zwei Listen.

Wie klein wird die Lücke, und an welchem Bahnpunkt? Hält die geplante Kurve
den Sicherheitsabstand von $0.1\,\text{m}$ ein? Zeichnen Sie das FTF an
diesem Bahnpunkt mit `bereich=(3.5, 6, 1, 3.5)`.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 3
:class: tip
:class: dropdown
Im dritten Abschnitt von Kapitel 6.1 haben wir die Übergabestation mit
`np.linalg.inv` in Fahrzeugkoordinaten umgerechnet. In Teil 3 von Kapitel 6.2
haben wir Punkte mit Vergleichen und `and` geprüft und Abstände in einer Liste
gesammelt. Hier kommt beides in der Schleife aus Teil 2 zusammen. Den Index der
kleinsten Lücke liefert `np.argmin`.
```python
# Regalecke für jeden Bahnpunkt in Fahrzeugkoordinaten umrechnen
luecken = []
bahnpunkte_neben = []
for k in range(len(bahn)):
    T_WF_k = transformationsmatrix(psi[k], bahn[k, 0], bahn[k, 1])
    ecke_F = transformiere(___, ecke)
    x_F = ecke_F[0, 0]
    y_F = ecke_F[0, 1]
    # Ecke links neben dem FTF: Lücke zwischen linker Fahrzeugseite und Ecke
    if ___:
        luecken.append(___)
        bahnpunkte_neben.append(k)

print('Bahnpunkte mit der Ecke neben dem FTF:', bahnpunkte_neben)
print('Lücken in m:', np.round(luecken, 3))

# Kleinste Lücke und zugehöriger Bahnpunkt
i_min = ___
k_eng = bahnpunkte_neben[i_min]
print(f'Kleinste Lücke: {luecken[i_min]:.3f} m an Bahnpunkt {k_eng}')
print('Sicherheitsabstand eingehalten:', ___)

zeichne_halle(fahrzeuge[k_eng], bahn=bahn, umgebung=halle,
              titel='Engste Stelle an der Regalecke', bereich=(3.5, 6, 1, 3.5))
```
````

````{admonition} Lösung Teil 3
:class: tip
:class: dropdown
```python
# Regalecke für jeden Bahnpunkt in Fahrzeugkoordinaten umrechnen
luecken = []
bahnpunkte_neben = []
for k in range(len(bahn)):
    T_WF_k = transformationsmatrix(psi[k], bahn[k, 0], bahn[k, 1])
    ecke_F = transformiere(np.linalg.inv(T_WF_k), ecke)
    x_F = ecke_F[0, 0]
    y_F = ecke_F[0, 1]
    # Ecke links neben dem FTF: Lücke zwischen linker Fahrzeugseite und Ecke
    if x_F >= -0.6 and x_F <= 0.6 and y_F >= 0:
        luecken.append(y_F - 0.4)
        bahnpunkte_neben.append(k)

print('Bahnpunkte mit der Ecke neben dem FTF:', bahnpunkte_neben)
print('Lücken in m:', np.round(luecken, 3))

# Kleinste Lücke und zugehöriger Bahnpunkt
i_min = np.argmin(luecken)
k_eng = bahnpunkte_neben[i_min]
print(f'Kleinste Lücke: {luecken[i_min]:.3f} m an Bahnpunkt {k_eng}')
print('Sicherheitsabstand eingehalten:', luecken[i_min] >= 0.1)

zeichne_halle(fahrzeuge[k_eng], bahn=bahn, umgebung=halle,
              titel='Engste Stelle an der Regalecke', bereich=(3.5, 6, 1, 3.5))
```
Ausgabe:

```text
Bahnpunkte mit der Ecke neben dem FTF: [8, 9, 10, 11, 12]
Lücken in m: [0.107 0.069 0.076 0.13  0.228]
Kleinste Lücke: 0.069 m an Bahnpunkt 9
Sicherheitsabstand eingehalten: False
```
An den Bahnpunkten 8 bis 12 liegt die Regalecke neben dem FTF. Davor liegt sie
vor dem Fahrzeug, danach dahinter. An Bahnpunkt 9 bleiben zwischen linker
Fahrzeugseite und Regalecke nur $0.069\,\text{m}$, also knapp
$7\,\text{cm}$. Das ist weniger als der geforderte Sicherheitsabstand von
$0.1\,\text{m}$ und deutlich weniger als die $0.3\,\text{m}$ im Regalgang. Die
Leitsteuerung darf diese Bahn nicht freigeben.

Im Fahrzeugkoordinatensystem genügt für die Prüfung eine einzige Subtraktion,
weil die linke Fahrzeugseite dort immer auf der Geraden $y_F = 0.4$ liegt. In
Hallenkoordinaten liegt diese Seite an jedem Bahnpunkt anders und schräg. Weil
wir nur an den Bahnpunkten prüfen, ist das Ergebnis eine Näherung. Zwischen
zwei Bahnpunkten kann die Lücke etwas anders ausfallen. Für eine Freigabe
würde die Leitsteuerung die Bahnpunkte deshalb deutlich dichter legen.
````

```{admonition} Abschlussfrage
:class: tip
1. Im Regalgang bleiben links und rechts je $0.3\,\text{m}$ frei. Warum wird
   die Lücke in der Kurve so viel kleiner? Vergleichen Sie dazu die Höhe des
   Kreismittelpunkts mit dem Anfang der Regale.
2. Soll die Leitsteuerung den Radius vergrößern oder verkleinern? Was spricht
   trotzdem dafür, Kurven mit möglichst großem Radius zu fahren?
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
1. Der Mittelpunkt des Viertelkreises liegt bei $y = 3.2\,\text{m}$, die
   Regale beginnen schon bei $y = 2.4\,\text{m}$. Der Viertelkreis endet
   deshalb erst $0.8\,\text{m}$ tief im Regalgang. Wenn die Vorderkante des
   FTF zwischen die Regale fährt, dreht es sich also noch, und seine Mitte
   liegt links von der Gangmitte. Die linke Fahrzeugseite schiebt sich dabei
   schräg an die Regalecke heran. Das FTF schneidet die Kurve.
2. Die Leitsteuerung sollte den Radius verkleinern. Mit einem Radius von
   $1.2\,\text{m}$ liegt der Mittelpunkt bei $y = 1.2\,\text{m} +
   1.2\,\text{m} = 2.4\,\text{m}$, genau auf Höhe der Regalanfänge. Die Kurve
   endet dann am Eingang des Regalgangs, und die Lücke bleibt bei etwa
   $0.3\,\text{m}$. Rechnet man Teil 3 für verschiedene Radien nach, halten
   Radien bis etwa $1.9\,\text{m}$ den Sicherheitsabstand ein. Für große
   Radien spricht die Geschwindigkeit. In der Kurve wirkt auf FTF und Ladung
   die Zentripetalbeschleunigung $a = v^2 / R$. Damit die Ladung nicht
   verrutscht, begrenzt die Steuerung $a$, zum Beispiel auf
   $0.5\,\text{m/s}^2$. Mit $R = 2\,\text{m}$ darf das FTF dann mit
   $v = \sqrt{a R} = 1.0\,\text{m/s}$ durch die Kurve fahren, mit
   $R = 1.2\,\text{m}$ nur mit $0.77\,\text{m/s}$. Die Leitsteuerung wählt
   deshalb den größten Radius, der den Sicherheitsabstand noch einhält.
````

```{admonition} Zusatzaufgabe: Koppelnavigation in der Kurve (✩✩✩)
:class: tip
Woher kennt die Steuerung des FTF während der Fahrt seine Pose? Sie zählt die
Umdrehungen der beiden Antriebsräder. Daraus berechnet sie nach jedem
Fahrtabschnitt, wie weit das FTF gefahren ist und um wie viel es sich gedreht
hat, und zwar aus Sicht des Fahrzeugs selbst. Dieses Verfahren heißt
**Koppelnavigation**.

Auf dem Viertelkreis meldet die Radmessung neunmal denselben Abschnitt: einen
Kreisbogen mit Radius $R = 2\,\text{m}$, auf dem sich das FTF um
$\Delta\psi = 10°$ dreht. Im Fahrzeugkoordinatensystem vor dem Abschnitt liegt
der Kreismittelpunkt bei $(0, R)$. Das FTF landet deshalb bei
$(R \sin\Delta\psi,\ R\,(1 - \cos\Delta\psi))$ und ist um $\Delta\psi$
gedreht. Wie die Lage des Scanners in Kapitel 6.2 beschreibt das ein neues
Fahrzeugkoordinatensystem relativ zum alten:
`T_schritt = transformationsmatrix(dpsi, radius * np.sin(dpsi), radius * (1 - np.cos(dpsi)))`.
Die neue Pose erhält die Steuerung deshalb wie $\mathbf{T}_{WS}$ in Kapitel
6.2 durch Multiplikation von rechts:

$$\mathbf{T}_{WF,\text{neu}} = \mathbf{T}_{WF,\text{alt}} \cdot \mathbf{T}_{\text{schritt}}.$$

1. Starten Sie mit der Pose $(3, 1.2, 0°)$ am Anfang des Viertelkreises und
   wenden Sie den Abschnitt in einer Schleife neunmal an. Prüfen Sie mit
   `np.allclose`, ob die berechnete Matrix zur Pose $(5, 3.2, 90°)$ am Ende
   des Viertelkreises passt.
2. Ein Antriebsrad rutscht in der Kurve etwas durch. Die Radmessung meldet
   deshalb für jeden Abschnitt $\Delta\psi = 10.2°$, obwohl sich das FTF
   tatsächlich nur um $10°$ dreht. Wiederholen Sie die Rechnung mit dem
   gemeldeten Wert. Hängen Sie danach an die berechnete und an die wahre Pose
   die Gerade im Regalgang als einen Abschnitt von $3\,\text{m}$ an,
   `transformationsmatrix(0, 3, 0)`. Wie weit liegen berechnete und wahre
   Endposition auseinander? Welchen Gierwinkel berechnet die Steuerung? Den
   Gierwinkel liefert `np.arctan2` aus der ersten Spalte der Matrix, denn dort
   steht die Richtung der $x_F$-Achse.
3. Beantworten Sie ohne Code: Auf der Geraden misst die Radmessung richtig.
   Warum wächst der Abstand zwischen berechneter und wahrer Position dort
   trotzdem weiter? Was kann die Steuerung dagegen tun?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Zusatzaufgabe
:class: tip
:class: dropdown
```python
# Ein Abschnitt auf dem Kreisbogen, gesehen aus dem Fahrzeug vor dem Abschnitt
radius = 2.0
dpsi = np.deg2rad(10)
T_schritt = transformationsmatrix(dpsi, radius * np.sin(dpsi),
                                  radius * (1 - np.cos(dpsi)))

# Teilaufgabe 1: neunmal von rechts anmultiplizieren
T_wahr = transformationsmatrix(0, 3.0, 1.2)
for i in range(9):
    T_wahr = T_wahr @ T_schritt
print('Pose am Ende des Viertelkreises:')
print(np.round(T_wahr, 4))
print('Passt zu (5, 3.2, 90°):',
      np.allclose(T_wahr, transformationsmatrix(np.deg2rad(90), 5.0, 3.2)))

# Teilaufgabe 2: Radmessung meldet 10.2 statt 10 Grad pro Abschnitt
dpsi_gemeldet = np.deg2rad(10.2)
T_schritt_gemeldet = transformationsmatrix(dpsi_gemeldet,
                                           radius * np.sin(dpsi_gemeldet),
                                           radius * (1 - np.cos(dpsi_gemeldet)))
T_berechnet = transformationsmatrix(0, 3.0, 1.2)
for i in range(9):
    T_berechnet = T_berechnet @ T_schritt_gemeldet

# Gerade im Regalgang: 3 m geradeaus, ohne Drehung
T_gerade = transformationsmatrix(0, 3.0, 0)
T_wahr = T_wahr @ T_gerade
T_berechnet = T_berechnet @ T_gerade

differenz = T_berechnet[0:2, 2] - T_wahr[0:2, 2]
abstand = np.sqrt(differenz[0]**2 + differenz[1]**2)
psi_berechnet = np.arctan2(T_berechnet[1, 0], T_berechnet[0, 0])
print('Wahre Endposition in m:', np.round(T_wahr[0:2, 2], 3))
print('Berechnete Endposition in m:', np.round(T_berechnet[0:2, 2], 3))
print(f'Abstand: {abstand:.3f} m')
print(f'Berechneter Gierwinkel: {np.rad2deg(psi_berechnet):.1f}°')
```
Ausgabe:

```text
Pose am Ende des Viertelkreises:
[[ 0.  -1.   5. ]
 [ 1.   0.   3.2]
 [ 0.   0.   1. ]]
Passt zu (5, 3.2, 90°): True
Wahre Endposition in m: [5.  6.2]
Berechnete Endposition in m: [4.905 6.261]
Abstand: 0.113 m
Berechneter Gierwinkel: 91.8°
```
Neun gleiche Abschnitte von rechts anmultipliziert ergeben genau das Ende des
Viertelkreises. In der ersten Spalte steht $(0, 1)$, die $x_F$-Achse zeigt
also in $y_W$-Richtung. Weil jeder Abschnitt im jeweils aktuellen
Fahrzeugkoordinatensystem beschrieben ist, gehört er wie $\mathbf{T}_{FS}$ in
Kapitel 6.2 nach rechts.

Mit dem Schlupf summiert sich der kleine Fehler von $0.2°$ in neun Abschnitten
zu $1.8°$. Nach weiteren $3\,\text{m}$ geradeaus liegt die berechnete Position
schon $11\,\text{cm}$ von der wahren entfernt, davon $9.5\,\text{cm}$ quer zum
Regalgang. Das ist fast ein Drittel der $0.3\,\text{m}$ Platz links und rechts
vom FTF. Auf der Geraden kommt kein neuer
Messfehler hinzu, aber der falsche Gierwinkel bleibt. Die Steuerung rechnet
jeden weiteren Meter in eine um $1.8°$ falsche Richtung, und der seitliche
Fehler wächst um etwa $1\,\text{m} \cdot \sin 1.8° \approx 3\,\text{cm}$ pro
Meter. Koppelnavigation allein driftet deshalb immer weiter ab. Die Steuerung
muss ihre Pose regelmäßig mit Messungen von außen korrigieren, zum Beispiel
mit Reflektoren wie in der Zusatzaufgabe von Kapitel 6.2.
````
