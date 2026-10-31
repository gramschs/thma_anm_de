---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 6.2 Sicherheitsscanner und Schutzfeld

In Kapitel 6.1 saß der Scanner in der Fahrzeugmitte. Jetzt kommt ein zweiter
Scanner vorne am Fahrzeug dazu, der darüber wacht, dass das FTF rechtzeitig
anhält. Bearbeiten Sie die Teile möglichst zu zweit und der Reihe nach.

````{admonition} Projekt: Muss das FTF anhalten? (✩✩)
:class: tip
Das FTF aus Kapitel 6.1 steht bei $(4, 2)$ mit $\psi = 30°$. In der Mitte
seiner Vorderkante sitzt ein **Sicherheitsscanner** mit eigenem
**Scannerkoordinatensystem** S, dessen Achsen in dieselben Richtungen zeigen
wie die des Fahrzeugs. Er misst wie in Kapitel 6.1 Entfernung $r$ und Winkel
$\varphi$, jetzt gegen die $x_S$-Achse, und überwacht das **Schutzfeld** vor
dem Fahrzeug. In unserem vereinfachten Modell gilt: Liegt ein Messpunkt im
Schutzfeld, muss das FTF anhalten.

```{figure} pics/chap06_schutzfeld.svg
:alt: Draufsicht auf das FTF mit dem Fahrzeugkoordinatensystem F in der Fahrzeugmitte, dem Scannerkoordinatensystem S 0.6 m weiter vorne in der Mitte der Vorderkante und dem gelben Schutzfeld, das 1.0 m lang und 1.0 m breit vor dem Fahrzeug liegt, dazu ein Messstrahl mit Entfernung r und Winkel phi gegen die x_S-Achse
:align: center

Fahrzeugkoordinatensystem F, Scannerkoordinatensystem S und Schutzfeld des
Sicherheitsscanners.
(Quelle: eigene Abbildung; Lizenz [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0))
```

| Objekt | Lage und Größe |
| --- | --- |
| FTF | $1.2\,\text{m} \times 0.8\,\text{m}$, Pose $(4, 2, 30°)$ |
| Sicherheitsscanner | bei $(0.6, 0)$ im Fahrzeugkoordinatensystem, nicht gedreht |
| Scan | $-90°$ bis $90°$ in Schritten von $5°$, Reichweite $5\,\text{m}$ |
| Schutzfeld | $x_F$ von $0.6\,\text{m}$ bis $1.6\,\text{m}$, $y_F$ von $-0.5\,\text{m}$ bis $0.5\,\text{m}$ |
| Hallenwand | bei $y = 0$ am unteren Bildrand, nicht in der Hallenkarte |
````

Führen Sie zuerst die beiden folgenden Zellen aus: die Zeichenfunktion aus
Kapitel 6.1, deren Parameter `schutzfeld` ein Schutzfeld gelb einzeichnet, und
eine Zelle mit den Funktionen und Daten aus den Kapiteln 5.3 und 6.1, den
Ecken des Schutzfelds und dem Scan. Strahlen, die innerhalb von
$5\,\text{m}$ nichts treffen, fehlen im Scan.

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

# Hallenkarte: Ecken von Regal, Palette und Säule in Hallenkoordinaten in m
regal = np.array([[0.5, 6.5], [9.5, 6.5], [9.5, 7.5], [0.5, 7.5]])
palette = np.array([[5.6, 4.0], [6.8, 4.0], [6.8, 4.8], [5.6, 4.8]])
saeule = np.array([[6.3, 0.3], [6.7, 0.3], [6.7, 0.7], [6.3, 0.7]])
halle = [regal, palette, saeule]

# Pose des FTF wie in Kapitel 6.1: Mitte bei (4, 2) m, Gierwinkel 30 Grad
T_WF = transformationsmatrix(np.deg2rad(30), 4, 2)
umriss_W = transformiere(T_WF, umriss)

# Ecken des Schutzfelds im Fahrzeugkoordinatensystem in m
schutzfeld = np.array([
    [0.6, -0.5],
    [1.6, -0.5],
    [1.6,  0.5],
    [0.6,  0.5],
])

# Scan des Sicherheitsscanners: Winkel gegen die x_S-Achse in Grad, Entfernung in m
winkel_grad = np.array([-90, -85, -80, -75, -70, -65, -60,
                        -35, -30, -25, -20, -15, -10,  -5,
                         10,  15,  20,  25,  30,  35,  40,  45,  50,
                         55,  60,  65,  70,  75,  80,  85,  90])
entfernungen = np.array([2.66, 2.81, 3.00, 2.52, 2.49, 4.01, 4.60,
                         1.04, 0.88, 0.83, 0.89, 0.96, 1.06, 1.18,
                         2.64, 2.40, 2.22, 2.08, 2.16, 2.56, 4.47, 4.35, 4.26,
                         4.22, 4.20, 4.22, 4.26, 4.35, 4.47, 4.63, 4.85])

zeichne_halle(umriss_W, koordinatensysteme=T_WF, umgebung=halle,
              titel='Ausgangslage')
```

```{admonition} Teil 1: Der Scan aus Sicht des Scanners
:class: tip
Rechnen Sie die Messungen wie im zweiten Abschnitt von Kapitel 6.1 in
kartesische Koordinaten um. Weil die Winkel gegen die $x_S$-Achse gemessen
sind, erhalten Sie Scannerkoordinaten `punkte_S`, eine Zeile pro Messung.
Bestimmen Sie mit `np.argmin` die kürzeste Entfernung und ihren Winkel.
Zeichnen Sie den Scan mit

`zeichne_halle(messpunkte=punkte_S, scanner=[0, 0], bereich=(-1, 5, -4.5, 6), titel='Scan in Scannerkoordinaten')`.

Liegt der nächste Gegenstand links oder rechts? Verrät das Bild, wo er in der
Halle steht?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 1
:class: tip
:class: dropdown
Im zweiten Abschnitt von Kapitel 6.1 haben wir fünf Messungen auf einmal in
Fahrzeugkoordinaten umgerechnet. Derselbe Code funktioniert für beliebig viele
Messungen, nur die Namen ändern sich.
```python
# Polarkoordinaten in Scannerkoordinaten umrechnen, alle Messungen auf einmal
winkel = np.deg2rad(___)
punkte_S = np.zeros((len(entfernungen), 2))
punkte_S[:, 0] = ___
punkte_S[:, 1] = ___
print('Form von punkte_S:', punkte_S.shape)

# Index der kürzesten Entfernung
k_nah = ___
print(f'Nächster Messpunkt: {entfernungen[k_nah]} m unter {winkel_grad[k_nah]}°')

zeichne_halle(messpunkte=punkte_S, scanner=[0, 0], bereich=(-1, 5, -4.5, 6),
              titel='Scan in Scannerkoordinaten')
```
````

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
```python
# Polarkoordinaten in Scannerkoordinaten umrechnen, alle Messungen auf einmal
winkel = np.deg2rad(winkel_grad)
punkte_S = np.zeros((len(entfernungen), 2))
punkte_S[:, 0] = entfernungen * np.cos(winkel)
punkte_S[:, 1] = entfernungen * np.sin(winkel)
print('Form von punkte_S:', punkte_S.shape)

# Index der kürzesten Entfernung
k_nah = np.argmin(entfernungen)
print(f'Nächster Messpunkt: {entfernungen[k_nah]} m unter {winkel_grad[k_nah]}°')

zeichne_halle(messpunkte=punkte_S, scanner=[0, 0], bereich=(-1, 5, -4.5, 6),
              titel='Scan in Scannerkoordinaten')
```
Ausgabe:

```text
Form von punkte_S: (31, 2)
Nächster Messpunkt: 0.83 m unter -25°
```
Der nächste Messpunkt liegt nur $0.83\,\text{m}$ vor dem Scanner, $25°$
rechts von der Fahrtrichtung. Er gehört zu einer Gruppe von sieben Punkten
zwischen $-35°$ und $-5°$, die alle etwa einen Meter entfernt sind. Links
vorne liegen sechs Punkte in gut zwei Metern Entfernung, dahinter zieht sich
eine lange Reihe von Punkten schräg durch das Bild. Rechts liegen einige
Punkte in zweieinhalb bis fünf Metern Entfernung.

Was diese Punkte sind und wo sie in der Halle liegen, verrät das Bild nicht.
Es zeigt die Halle so, wie der Scanner sie sieht, und der Scanner weiß nicht,
wo er selbst steht. Dafür brauchen wir die Lage des Scanners am Fahrzeug und
die Pose des Fahrzeugs in der Halle.
````

```{admonition} Teil 2: Der Scan in der Halle
:class: tip
Bauen Sie mit `transformationsmatrix` die Matrix $\mathbf{T}_{FS}$ auf, die von
S nach F rechnet. Rechnen Sie die Messpunkte in zwei Schritten um: mit
$\mathbf{T}_{FS}$ in `punkte_F`, danach mit $\mathbf{T}_{WF}$ in `punkte_W`.
Wie in Kapitel 5.3 lassen sich beide Schritte verketten:

$$\mathbf{T}_{WS} = \mathbf{T}_{WF} \cdot \mathbf{T}_{FS}.$$

Die Indizes greifen wie Kettenglieder ineinander: von S über F nach W. Prüfen
Sie mit `np.allclose`, ob `transformiere(T_WS, punkte_S)` dasselbe liefert wie
die zwei Schritte. Zeichnen Sie Fahrzeug, Scan und Hallenkarte. Die Lage des
Scanners steht in der dritten Spalte von $\mathbf{T}_{WS}$.

Welche Messpunkte passen zu keinem Gegenstand der Hallenkarte, und wo liegen
sie?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 2
:class: tip
:class: dropdown
Der Scanner ist gegenüber dem Fahrzeug nur verschoben, nicht gedreht.
`transformationsmatrix` aus Kapitel 5.3 bekommt deshalb den Winkel 0. Die
beiden Umrechnungen erledigt `transformiere` wie beim Umriss in Kapitel 6.1,
die Verkettung schreiben wir wie in Kapitel 5.3 mit `@`.
```python
# Scanner in der Mitte der Vorderkante, nicht gedreht
T_FS = transformationsmatrix(___, ___, ___)

# Zwei Schritte: vom Scanner ins Fahrzeug, vom Fahrzeug in die Halle
punkte_F = transformiere(___, punkte_S)
punkte_W = transformiere(___, punkte_F)

# Ein Schritt: verkettete Matrix von S nach W
T_WS = ___
print('T_WS:')
print(np.round(T_WS, 4))
print('Beide Wege gleich:', np.allclose(transformiere(T_WS, punkte_S), punkte_W))

zeichne_halle(umriss_W, messpunkte=punkte_W, scanner=___, umgebung=halle,
              titel='Scan in Hallenkoordinaten')
```
````

````{admonition} Lösung Teil 2
:class: tip
:class: dropdown
```python
# Scanner in der Mitte der Vorderkante, nicht gedreht
T_FS = transformationsmatrix(0, 0.6, 0)

# Zwei Schritte: vom Scanner ins Fahrzeug, vom Fahrzeug in die Halle
punkte_F = transformiere(T_FS, punkte_S)
punkte_W = transformiere(T_WF, punkte_F)

# Ein Schritt: verkettete Matrix von S nach W
T_WS = T_WF @ T_FS
print('T_WS:')
print(np.round(T_WS, 4))
print('Beide Wege gleich:', np.allclose(transformiere(T_WS, punkte_S), punkte_W))

zeichne_halle(umriss_W, messpunkte=punkte_W, scanner=T_WS[0:2, 2], umgebung=halle,
              titel='Scan in Hallenkoordinaten')
```
Ausgabe:

```text
T_WS:
[[ 0.866  -0.5     4.5196]
 [ 0.5     0.866   2.3   ]
 [ 0.      0.      1.    ]]
Beide Wege gleich: True
```
Die dritte Spalte von $\mathbf{T}_{WS}$ ist die Lage des Scanners in der
Halle. Sie liegt $0.6\,\text{m}$ vor der Fahrzeugmitte in Fahrtrichtung:
$4 + 0.6 \cdot \cos 30° = 4.520$ und $2 + 0.6 \cdot \sin 30° = 2.300$. Links
oben steht dieselbe Drehung wie in $\mathbf{T}_{WF}$, weil der Scanner
gegenüber dem Fahrzeug nicht gedreht ist.

Fast alle Messpunkte können wir zuordnen: rechts die Hallenwand am unteren
Bildrand bei $y = 0$ und die Säule, vorne links die untere und die linke Kante
der Palette, links die untere Kante des Regals bei $y = 6.5$. Sieben Punkte
zwischen Fahrzeug und Palette, etwa bei $x = 5.3$ bis $5.6$ und $y = 2.2$ bis
$2.8$, passen weder zur Wand noch zu einem Gegenstand der Karte. Sie liegen auf zwei
Kanten, die einen rechten Winkel bilden. Dort steht also ein kastenförmiger
Gegenstand, der nicht in der Karte verzeichnet ist, zum Beispiel eine
abgestellte Kiste. Die Lücken im Scan gehören zu Strahlen, die nach rechts aus
unserem Hallenausschnitt hinauszeigen. Sie treffen innerhalb der Reichweite
von $5\,\text{m}$ nichts.
````

````{admonition} Teil 3: Liegt ein Messpunkt im Schutzfeld?
:class: tip
Das Schutzfeld fährt mit dem Fahrzeug mit und hat in Fahrzeugkoordinaten
immer dieselben Grenzen. Ein Messpunkt liegt darin, wenn er alle vier Grenzen
einhält:

```{figure} pics/chap06_schutzfeld_pruefung.svg
:alt: Das FTF und das gelbe Schutzfeld im Fahrzeugkoordinatensystem mit den Grenzen x_F = 0.6 und 1.6 sowie y_F = 0.5 und -0.5, ein roter Messpunkt im Schutzfeld mit eingezeichnetem Längsabstand x_F - 0.6 zur Vorderkante, ein Punkt zu weit vorne und ein Punkt seitlich daneben
:align: center

Prüfung des Schutzfelds im Fahrzeugkoordinatensystem.
(Quelle: eigene Abbildung; Lizenz [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0))
```

Gehen Sie in einer Schleife durch `punkte_F` aus Teil 2. Zählen Sie die Punkte
im Schutzfeld und sammeln Sie für sie den **Längsabstand** $x_F - 0.6$, also
wie weit der Punkt in Fahrtrichtung vor der Vorderkante liegt. Wie groß ist
der kleinste Längsabstand? Muss das FTF anhalten?

Rechnen Sie zum Schluss die Ecken des Schutzfelds in die Halle um und zeichnen
Sie sie über den Parameter `schutzfeld` in das Bild aus Teil 2 ein. Warum
liegen nicht alle Punkte des unbekannten Gegenstands im Schutzfeld?
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 3
:class: tip
:class: dropdown
In Teil 3 von Kapitel 5.2 haben wir mit vier Vergleichen und `and` geprüft, ob
ein Deckel in der Kiste liegt. Hier prüfen wir dasselbe für jeden Messpunkt
einzeln und zählen wie in Kapitel 1.4 mit einer Variablen `anzahl` mit. Die
Ecken des Schutzfelds sind wie der Umriss fest mit dem Fahrzeug verbunden.
```python
# Messpunkte im Schutzfeld zählen, Prüfung im Fahrzeugkoordinatensystem
anzahl = 0
abstaende = []
for k in range(len(punkte_F)):
    x_F = punkte_F[k, 0]
    y_F = punkte_F[k, 1]
    if x_F >= 0.6 and x_F <= ___ and ___:
        anzahl = anzahl + 1
        abstaende.append(___)   # Längsabstand zur Vorderkante

print('Messpunkte im Schutzfeld:', anzahl)
print(f'Kleinster Längsabstand zur Vorderkante: {np.min(abstaende):.2f} m')
print('FTF muss anhalten:', ___)

# Schutzfeld in die Halle umrechnen und einzeichnen
schutzfeld_W = ___
zeichne_halle(umriss_W, messpunkte=punkte_W, scanner=T_WS[0:2, 2],
              schutzfeld=schutzfeld_W, umgebung=halle,
              titel='Hindernis im Schutzfeld')
```
````

````{admonition} Lösung Teil 3
:class: tip
:class: dropdown
```python
# Messpunkte im Schutzfeld zählen, Prüfung im Fahrzeugkoordinatensystem
anzahl = 0
abstaende = []
for k in range(len(punkte_F)):
    x_F = punkte_F[k, 0]
    y_F = punkte_F[k, 1]
    if x_F >= 0.6 and x_F <= 1.6 and y_F >= -0.5 and y_F <= 0.5:
        anzahl = anzahl + 1
        abstaende.append(x_F - 0.6)   # Längsabstand zur Vorderkante

print('Messpunkte im Schutzfeld:', anzahl)
print(f'Kleinster Längsabstand zur Vorderkante: {np.min(abstaende):.2f} m')
print('FTF muss anhalten:', anzahl > 0)

# Schutzfeld in die Halle umrechnen und einzeichnen
schutzfeld_W = transformiere(T_WF, schutzfeld)
zeichne_halle(umriss_W, messpunkte=punkte_W, scanner=T_WS[0:2, 2],
              schutzfeld=schutzfeld_W, umgebung=halle,
              titel='Hindernis im Schutzfeld')
```
Ausgabe:

```text
Messpunkte im Schutzfeld: 4
Kleinster Längsabstand zur Vorderkante: 0.75 m
FTF muss anhalten: True
```
Vier Messpunkte liegen im Schutzfeld, der nächste nur $0.75\,\text{m}$ vor der
Vorderkante. Das FTF muss anhalten. Alle vier gehören zu dem unbekannten
Gegenstand aus Teil 2. Seine drei übrigen Punkte liegen außerhalb des
Schutzfelds: Die Punkte unter $-10°$ und $-5°$ haben $x_F = 1.64\,\text{m}$
und $x_F = 1.78\,\text{m}$ und liegen damit hinter dem Ende des Schutzfelds.
Der Punkt unter $-35°$ hat $y_F = -0.60\,\text{m}$ und liegt rechts neben dem
Schutzfeld. Im Bild ist das Schutzfeld wie das Fahrzeug um $30°$ gedreht, denn
$\mathbf{T}_{WF}$ bringt seine Ecken genauso in die Halle wie die Ecken des
Umrisses.
````

```{admonition} Abschlussfrage
:class: tip
1. Warum prüfen wir das Schutzfeld in Fahrzeugkoordinaten und nicht in
   Hallenkoordinaten? Welche der Matrizen $\mathbf{T}_{FS}$ und
   $\mathbf{T}_{WF}$ braucht die Prüfung, und was folgt daraus, wenn die
   Steuerung die Pose des Fahrzeugs falsch kennt?
2. Zwei benachbarte Strahlen liegen $5°$ auseinander. Wie weit liegen ihre
   Messpunkte in $1\,\text{m}$ Entfernung auseinander? Würde der Scanner ein
   Stuhlbein mit $4\,\text{cm}$ Durchmesser in dieser Entfernung sicher
   erkennen?
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
1. Im Fahrzeugkoordinatensystem hat das Schutzfeld immer dieselben Grenzen,
   egal wo das Fahrzeug steht und wohin es zeigt. Seine Kanten liegen parallel
   zu den Achsen, deshalb genügen vier einfache Vergleiche wie beim Deckel in
   der Kiste aus Kapitel 5.2. In Hallenkoordinaten ist das Schutzfeld um
   $\psi$ gedreht, seine Kanten liegen schräg, und bei jeder neuen Pose liegt
   es woanders. Für die Prüfung in Fahrzeugkoordinaten brauchen wir nur
   $\mathbf{T}_{FS}$, also die feste Lage des Scanners am Fahrzeug.
   $\mathbf{T}_{WF}$ und damit die Pose brauchen wir dafür nicht. In unserem
   vereinfachten Modell ist die Prüfung also unabhängig von der Pose in der
   Halle, und eine falsch bekannte Pose ändert nichts an der Entscheidung, ob
   das FTF anhält. Echte Sicherheitsscanner werten ihr Schutzfeld deshalb
   selbst aus, unabhängig von der Navigation des Fahrzeugs. Ob das FTF in der
   Realität rechtzeitig zum Stehen kommt, ist damit aber noch nicht gesagt.
   Dafür müssen unter anderem Scanner und Montage, die Größe des Schutzfelds,
   Geschwindigkeit und Bremsweg, die Auswertezeit und der Umgang mit Fehlern
   sicherheitstechnisch ausgelegt sein.
2. Benachbarte Strahlen liegen $5°$ auseinander, in Radiant sind das
   $5 \cdot \pi / 180 \approx 0.087$. Wie beim Kreisbogen in Kapitel 5.2 liegen
   ihre Messpunkte in $1\,\text{m}$ Entfernung deshalb etwa
   $1\,\text{m} \cdot 0.087 = 8.7\,\text{cm}$ auseinander. Ein Stuhlbein mit
   $4\,\text{cm}$ Durchmesser kann genau zwischen zwei Strahlen stehen und wird
   dann nicht erkannt. Je weiter ein Gegenstand entfernt ist, desto größer
   werden die Lücken. Reale Sicherheitsscanner verwenden deshalb deutlich
   kleinere Winkelabstände. Welche Gegenstände sie sicher erkennen, hängt
   außerdem vom Durchmesser des Laserstrahls, von der Reflexion und von der
   Auswertung mehrerer aufeinanderfolgender Scans ab. Unser Scan mit $5°$ dient
   nur der Übersicht.
````

```{admonition} Zusatzaufgabe: Den Gierwinkel mit einem Reflektor bestimmen (✩✩✩)
:class: tip
Die Steuerung des FTF berechnet die Pose laufend aus den Drehungen der Räder.
Kleine Messfehler summieren sich dabei auf, besonders beim Gierwinkel. Zur
Korrektur hängt an der Säule ein **Reflektor**, laut Hallenkarte bei
$(6.3, 0.7)$. Für die Lokalisierung wertet die Steuerung die Messdaten des
Scanners vorne zusätzlich aus, und zwar mit einer Winkelauflösung von $1°$ statt
der $5°$ unseres Schutzfeldscans. In der Praxis übernimmt das oft ein eigener
Navigationsscanner, denn nicht jeder Sicherheitsscanner liefert solche Daten.
Den Reflektor erkennt die Auswertung an seinem besonders hellen Echo. Er liegt
in der Entfernung $r = 2.39\,\text{m}$ unter dem Winkel $\varphi = -72°$, beide
Werte gerundet. Nehmen Sie an, dass die Fahrzeugmitte bei $(4, 2)$ steht und
nur der Gierwinkel unbekannt ist.

1. Probieren Sie alle Gierwinkel von $0°$ bis $359°$ in Schritten von $1°$
   aus. Berechnen Sie für jeden Winkel mit
   $\mathbf{T}_{WF} \cdot \mathbf{T}_{FS}$, wo der Reflektor in der Halle läge,
   und seinen Abstand zur Lage laut Hallenkarte.
2. Bestimmen Sie mit `np.argmin` den Gierwinkel mit dem kleinsten Abstand.
   Zeichnen Sie den Abstand über dem Gierwinkel.
3. Beantworten Sie ohne Code: Warum genügt ein einziger Reflektor nicht mehr,
   wenn auch die Position des Fahrzeugs unbekannt ist?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Zusatzaufgabe
:class: tip
:class: dropdown
```python
# Reflektor: Messung in Scannerkoordinaten und Lage laut Hallenkarte
r_reflektor = 2.39
phi_reflektor = np.deg2rad(-72)
reflektor_S = np.array([[r_reflektor * np.cos(phi_reflektor),
                         r_reflektor * np.sin(phi_reflektor)]])
reflektor_karte = np.array([6.3, 0.7])

# Teilaufgabe 1: alle Gierwinkel von 0 bis 359 Grad ausprobieren
psi_grad = np.linspace(0, 359, 360)
abstaende_reflektor = np.zeros(len(psi_grad))
for k in range(len(psi_grad)):
    T_WF_k = transformationsmatrix(np.deg2rad(psi_grad[k]), 4, 2)
    reflektor_W = transformiere(T_WF_k @ T_FS, reflektor_S)
    differenz = reflektor_W[0] - reflektor_karte
    abstaende_reflektor[k] = np.sqrt(differenz[0]**2 + differenz[1]**2)

# Teilaufgabe 2: Gierwinkel mit dem kleinsten Abstand
k_best = np.argmin(abstaende_reflektor)
print(f'Gierwinkel: {psi_grad[k_best]:.0f}°')
print(f'Restabstand: {abstaende_reflektor[k_best] * 1000:.1f} mm')

fig, ax = plt.subplots(figsize=(7, 4))
ax.plot(psi_grad, abstaende_reflektor)
ax.set_xlabel('Gierwinkel in Grad')
ax.set_ylabel('Abstand zum Reflektor in m')
ax.grid(True)
plt.show()
```
Ausgabe:

```text
Gierwinkel: 30°
Restabstand: 4.3 mm
```
Der kleinste Abstand ergibt sich bei $\psi = 30°$, dem Gierwinkel aus Kapitel
6.1. Der Restabstand von wenigen Millimetern kommt daher, dass Entfernung und
Winkel auf Zentimeter und Grad gerundet sind. Die Kurve hat genau ein Minimum.
Bei jedem anderen Gierwinkel läge der Reflektor woanders in der Halle, bei
$210°$ sogar mehr als $5\,\text{m}$ von seiner Lage laut Karte entfernt. Eine
Funktion, deren kleinster Wert die gesuchte Größe liefert, heißt
**Kostenfunktion**. In Kapitel 8.1 lernen wir mit Gradient Descent ein
Verfahren kennen, das ihr Minimum findet, ohne alle Werte auszuprobieren.

Ist auch die Position unbekannt, suchen wir drei Größen $x$, $y$ und $\psi$.
Ein Reflektor legt nur fest, dass der Scanner im Abstand $r$ vom Reflektor
steht und ihn unter dem Winkel $\varphi$ sieht. Das gilt für jede Lage des
Scanners auf einem Kreis mit Radius $r$ um den Reflektor, wenn das Fahrzeug
jeweils passend gedreht ist. Zwei unterscheidbare und richtig zugeordnete
Reflektoren legen die Pose eindeutig fest. Navigationsscanner nutzen deshalb
mehrere Reflektoren gleichzeitig und müssen dabei klären, welcher gemessene
Reflektor zu welchem Eintrag der Hallenkarte gehört. Die Rastersuche würde mit drei Unbekannten außerdem sehr teuer:
Mit Schritten von $1\,\text{cm}$ und $1°$ wären es in unserem
Hallenausschnitt $1000 \cdot 800 \cdot 360 \approx 2.9 \cdot 10^8$ Posen.
````
