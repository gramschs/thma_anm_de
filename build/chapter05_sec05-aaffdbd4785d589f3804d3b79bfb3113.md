---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 5.5 Übungen

Diese Aufgaben sind für das Selbststudium zuhause gedacht und wiederholen den
Stoff der Kapitel 5.1 bis 5.4. Rechnen Sie mit rund 90 Minuten
Bearbeitungszeit. Führen Sie vor den Aufgaben die beiden folgenden Zellen aus.
Sie enthalten die Zeichenfunktion, die Funktionen aus den Kapiteln 5.1, 5.3
und 5.4 sowie Arm und Bauteil des SCARA-Roboters.

Der Schwierigkeitsgrad steht im Titel jeder Aufgabe:

* ✩ Verständnis: Code und Ausgaben vorhersagen und erklären (ca. 5 min)
* ✩✩ Anwendung: eigenen Code schreiben und Ergebnisse interpretieren (ca. 10 min)
* ✩✩✩ Mini-Projekt: mehrere Konzepte des Parts kombinieren (ca. 30 min)

```{code-cell} python
:tags: [hide-input]
# Vorgegebene Zeichenfunktion: einfach ausführen, sie muss nicht verstanden werden.
def zeichne_roboter(arme, bauteile=None, ziele=None, bahn=None, umgebung=None,
                    beschriftungen=None, titel='', grenze=0.8):
    """Zeichnet einen Roboterarm in der Draufsicht.

    arme: Punkte eines Arms als Array, Zeile = [x, y], vom ersten Gelenk bis
          zum Greifer, oder eine Liste solcher Arrays für mehrere Stellungen
    bauteile: Eckpunkte des Bauteils im Greifer als Array, oder eine Liste
              mit einem Array pro Stellung (optional)
    ziele: Zielpunkte als Array, Zeile = [x, y], werden als Kreuze markiert (optional)
    bahn: Greiferpositionen als Array, Zeile = [x, y], werden als gestrichelte
          Linie mit Punkten gezeichnet (optional)
    umgebung: Eckpunkte von Flächen wie Förderband oder Kiste als Array oder
              Liste von Arrays, werden grau gefüllt (optional)
    beschriftungen: Legendentexte, einer pro Stellung (optional)
    titel: Diagrammtitel
    grenze: sichtbarer Bereich von -grenze bis +grenze in m
    """
    blau, rot, orange, grau = '#005A94', '#E60000', '#E87846', '#484949'
    farben = [blau, rot, orange, grau]
    if isinstance(arme, np.ndarray):
        arme = [arme]
    if isinstance(bauteile, np.ndarray):
        bauteile = [bauteile]
    if isinstance(umgebung, np.ndarray):
        umgebung = [umgebung]
    fig, ax = plt.subplots(figsize=(6, 6))

    # Umgebung: Flächen wie Förderband oder Kiste
    if umgebung is not None:
        for flaeche in umgebung:
            ax.fill(flaeche[:, 0], flaeche[:, 1], facecolor='#DDDEDE',
                    edgecolor=grau, linewidth=1, zorder=0)

    # Sockel im Ursprung
    ax.add_patch(plt.Circle((0, 0), 0.06, facecolor='#CCDEE9', edgecolor=blau,
                            linewidth=1.5, zorder=1))

    # Greiferbahn als gestrichelte Linie mit Punkten
    if bahn is not None:
        ax.plot(bahn[:, 0], bahn[:, 1], color=grau, linestyle='--', linewidth=1,
                marker='o', markersize=3, label='Greiferbahn', zorder=1)

    # Arme: Gelenke als Kreise, Greifer als Quadrat, Bauteil gelb gefüllt
    for k in range(len(arme)):
        farbe = farben[k % len(farben)]
        punkte = arme[k]
        label = None if beschriftungen is None else beschriftungen[k]
        ax.plot(punkte[:, 0], punkte[:, 1], color=farbe, linewidth=6,
                solid_capstyle='round', label=label, zorder=2)
        ax.scatter(punkte[:-1, 0], punkte[:-1, 1], s=50, color='white',
                   edgecolor=farbe, linewidth=2, zorder=3)
        ax.scatter(punkte[-1, 0], punkte[-1, 1], s=60, marker='s', color=farbe,
                   zorder=3)
        if bauteile is not None:
            ecken = bauteile[k]
            ax.fill(ecken[:, 0], ecken[:, 1], facecolor='#FFEC7F',
                    edgecolor=farbe, linewidth=1.5, zorder=4)

    # Zielpunkte als Kreuze
    if ziele is not None:
        ziele = np.atleast_2d(ziele)
        ax.scatter(ziele[:, 0], ziele[:, 1], marker='x', s=80, color=grau,
                   linewidth=2, zorder=5, label='Ziel')

    if beschriftungen is not None or ziele is not None or bahn is not None:
        ax.legend(loc='lower left')
    ax.set_xlim(-grenze, grenze)
    ax.set_ylim(-grenze, grenze)
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

# Funktion aus Kapitel 5.1
def drehmatrix(winkel):
    """Drehmatrix für eine Drehung um den Ursprung, winkel in Radiant."""
    return np.array([
        [np.cos(winkel), -np.sin(winkel)],
        [np.sin(winkel),  np.cos(winkel)],
    ])

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

def drehung_um_punkt(winkel, zentrum):
    """Homogene Matrix für eine Drehung um den Punkt zentrum, winkel in Radiant."""
    T_hin = transformationsmatrix(0, -zentrum[0], -zentrum[1])
    T_dreh = transformationsmatrix(winkel, 0, 0)
    T_zurueck = transformationsmatrix(0, zentrum[0], zentrum[1])
    return T_zurueck @ T_dreh @ T_hin

# Punkte des Arms in Grundstellung in m, Zeile = [x, y]
arm = np.array([
    [0.0, 0.0],   # Gelenk 1
    [0.4, 0.0],   # Gelenk 2
    [0.7, 0.0],   # Greifer
])

# Ecken des Bauteils im Greifer in m, Grundstellung
bauteil = np.array([
    [0.66, -0.02],
    [0.74, -0.02],
    [0.74,  0.02],
    [0.66,  0.02],
])

# Funktion aus Kapitel 5.4
def stellung(alpha, beta):
    """Punkte des Arms und Ecken des Bauteils für die Gelenkwinkel in Radiant."""
    T_alpha = transformationsmatrix(alpha, 0, 0)
    T_arm2 = T_alpha @ drehung_um_punkt(beta, arm[1])
    # Arm 1 nur mit T_alpha, Arm 2 und Bauteil mit beiden Drehungen
    arm1_neu = transformiere(T_alpha, arm[0:2])
    arm2_neu = transformiere(T_arm2, arm[1:3])
    arm_neu = np.array([arm1_neu[0], arm1_neu[1], arm2_neu[1]])
    bauteil_neu = transformiere(T_arm2, bauteil)
    return arm_neu, bauteil_neu
```

````{admonition} Aufgabe 5.1 (✩)
:class: tip
Bei $\alpha = 0$ und $\beta = 90°$ steht der Greifer des SCARA-Roboters wie in
Kapitel 5.3 am Punkt $(0.4, 0.3)$. Nun dreht sich Gelenk 1 um $90°$, und
Gelenk 2 bleibt abgeknickt. Gegeben ist folgender Code:

```python
import numpy as np

greifer = np.array([0.4, 0.3])
R = drehmatrix(np.deg2rad(90))

print(np.round(R, 4))
print(np.round(R @ greifer, 4))
print(np.round(R.T @ greifer, 4))
print(np.round(np.cos(90), 4))
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Welche Einträge hat die Matrix `R`?
2. Was gibt `R @ greifer` aus? Rechnen Sie das Produkt von Hand aus. Wie weit
   ist der Greifer vor und nach der Drehung vom Ursprung entfernt?
3. Welcher Drehung entspricht `R.T`, und wo landet der Greifer damit?
4. Warum gibt die letzte Zeile nicht 0 aus?
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

greifer = np.array([0.4, 0.3])
R = drehmatrix(np.deg2rad(90))

print(np.round(R, 4))
print(np.round(R @ greifer, 4))
print(np.round(R.T @ greifer, 4))
print(np.round(np.cos(90), 4))
```
Ausgabe:

```text
[[ 0. -1.]
 [ 1.  0.]]
[-0.3  0.4]
[ 0.3 -0.4]
-0.4481
```

In den Spalten von `R` stehen die gedrehten Einheitsvektoren: $\vec{e}_x$
wird zu $(0, 1)^\top$, $\vec{e}_y$ zu $(-1, 0)^\top$. Von Hand gilt

$$\begin{pmatrix} 0 & -1 \\ 1 & 0 \end{pmatrix}
\begin{pmatrix} 0.4 \\ 0.3 \end{pmatrix}
= \begin{pmatrix} 0 \cdot 0.4 + (-1) \cdot 0.3 \\ 1 \cdot 0.4 + 0 \cdot 0.3 \end{pmatrix}
= \begin{pmatrix} -0.3 \\ 0.4 \end{pmatrix}.$$

Vor und nach der Drehung ist der Greifer $\sqrt{0.4^2 + 0.3^2} = 0.5\,\text{m}$
vom Ursprung entfernt, denn eine Drehung ändert keine Längen. `R.T` ist die
Drehung um $-90°$. Sie bringt den Greifer nach $(0.3, -0.4)$, also in die
Stellung $\alpha = -90°$, $\beta = 90°$. Die letzte Zeile rechnet mit
90 Radiant statt mit $90°$. Ohne `np.deg2rad` gibt es keine Fehlermeldung,
sondern nur ein falsches Ergebnis.
````

````{admonition} Aufgabe 5.2 (✩)
:class: tip
Der Schwenkarm aus Kapitel 5.2 schwenkt den Deckel zuerst um $50°$ gegen den
Uhrzeigersinn und danach um $20°$ zurück. Gegeben ist folgender Code:

```python
import numpy as np

# Ecken des Deckels in Grundstellung in m
deckel = np.array([
    [0.55, -0.03],
    [0.65, -0.03],
    [0.65,  0.03],
    [0.55,  0.03],
])
R_1 = drehmatrix(np.deg2rad(50))
R_2 = drehmatrix(np.deg2rad(-20))
deckel_neu = (R_2 @ R_1 @ deckel.T).T

print(deckel.T.shape)
print(np.allclose(R_2 @ R_1, drehmatrix(np.deg2rad(30))))
print(np.allclose(R_2 @ R_1, R_1 @ R_2))

# Lange Kante des Deckels von Ecke 0 nach Ecke 1
kante = deckel_neu[1] - deckel_neu[0]
print(np.round(kante, 4))
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Welche Form hat `deckel.T`, und welche Form hat `deckel_neu`?
2. Was geben die beiden Zeilen mit `np.allclose` aus? Begründen Sie Ihre
   Antwort.
3. In der Grundstellung ist die lange Kante $0.1\,\text{m}$ lang und zeigt in
   $x$-Richtung. Welche Werte erwarten Sie für `kante` nach dem Schwenk? Tipp:
   $\cos 30° \approx 0.866$.
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

deckel = np.array([
    [0.55, -0.03],
    [0.65, -0.03],
    [0.65,  0.03],
    [0.55,  0.03],
])
R_1 = drehmatrix(np.deg2rad(50))
R_2 = drehmatrix(np.deg2rad(-20))
deckel_neu = (R_2 @ R_1 @ deckel.T).T

print(deckel.T.shape)
print(np.allclose(R_2 @ R_1, drehmatrix(np.deg2rad(30))))
print(np.allclose(R_2 @ R_1, R_1 @ R_2))

kante = deckel_neu[1] - deckel_neu[0]
print(np.round(kante, 4))
```
Ausgabe:

```text
(2, 4)
True
True
[0.0866 0.05  ]
```

`deckel` hat vier Ecken und die Form `(4, 2)`, `deckel.T` hat die Form
`(2, 4)`. Nach dem zweiten `.T` hat `deckel_neu` wieder die Form `(4, 2)`.
Beide Drehungen gehen um dasselbe Gelenk, deshalb addieren sich die Winkel zu
$50° - 20° = 30°$. Bei einer Summe kommt es nicht auf die Reihenfolge an,
deshalb sind hier auch die beiden Matrixprodukte gleich. Das gilt nur für
Drehungen um denselben Punkt. Die Kante ist die Differenz zweier Ecken, ihre
Lage spielt keine Rolle. Sie dreht sich nur mit: Aus $(0.1, 0)$ wird
$0.1 \cdot (\cos 30°, \sin 30°) = (0.0866, 0.05)$, die Länge bleibt
$0.1\,\text{m}$.
````

````{admonition} Aufgabe 5.3 (✩)
:class: tip
Gegeben ist folgender Code mit einer homogenen Transformationsmatrix und drei
Punkten auf dem gestreckten Arm des SCARA-Roboters:

```python
import numpy as np

T = transformationsmatrix(np.deg2rad(180), 0.7, 0.0)

# Punkte auf dem Arm in homogenen Koordinaten
gelenk1 = np.array([0.0, 0.0, 1.0])
greifer = np.array([0.7, 0.0, 1.0])
mitte = np.array([0.35, 0.0, 1.0])
# Richtung des Arms in Grundstellung
richtung = np.array([1.0, 0.0, 0.0])

print(np.round(T @ gelenk1, 4))
print(np.round(T @ greifer, 4))
print(np.round(T @ mitte, 4))
print(np.round(T @ richtung, 4))
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Was gibt `T @ gelenk1` aus? Welcher Teil von `T` steckt in diesem Ergebnis?
2. Wohin bringt `T` den Greifer? Denken Sie daran, dass `T` zuerst dreht und
   danach verschiebt.
3. Was gibt `T @ mitte` aus? Welche Bewegung beschreibt `T` also insgesamt?
4. `richtung` hat als dritte Koordinate eine 0. Was gibt `T @ richtung` aus,
   und warum wird die Richtung nicht verschoben?
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

T = transformationsmatrix(np.deg2rad(180), 0.7, 0.0)

gelenk1 = np.array([0.0, 0.0, 1.0])
greifer = np.array([0.7, 0.0, 1.0])
mitte = np.array([0.35, 0.0, 1.0])
richtung = np.array([1.0, 0.0, 0.0])

print(np.round(T @ gelenk1, 4))
print(np.round(T @ greifer, 4))
print(np.round(T @ mitte, 4))
print(np.round(T @ richtung, 4))
```
Ausgabe:

```text
[0.7 0.  1. ]
[0. 0. 1.]
[0.35 0.   1.  ]
[-1.  0.  0.]
```

Die Drehung lässt den Ursprung an seinem Platz, für Gelenk 1 bleibt also nur
die Verschiebung übrig. `T @ gelenk1` liefert deshalb die rechte Spalte von
`T`, den Verschiebungsvektor $(0.7, 0)$. Den Greifer dreht `T` zuerst nach
$(-0.7, 0)$ und verschiebt ihn dann nach $(0, 0)$. Gelenk 1 und Greifer
tauschen also die Plätze, nur die Mitte des Arms bleibt bei $(0.35, 0)$
stehen. Insgesamt klappt `T` den Arm um: Es ist eine Drehung um $180°$ um die
Armmitte, dieselbe Matrix liefert `drehung_um_punkt(np.deg2rad(180), [0.35, 0])`.
Bei `richtung` wird die rechte Spalte von `T` mit der 0 multipliziert. Die
Richtung wird nur gedreht und zeigt danach in $-x$-Richtung, verschoben wird
sie nicht. Eine Richtung hat keinen Ort, deshalb ist das genau richtig.
````

````{admonition} Aufgabe 5.4 (✩)
:class: tip
Ein Werkstück liegt bei $(0.1, 0)$ auf dem Tisch einer Werkzeugmaschine. Der
Tisch kann sich um den Ursprung drehen und in $x$-Richtung verschoben werden.
Gegeben ist folgender Code:

```python
import numpy as np

T_dreh = transformationsmatrix(np.deg2rad(90), 0, 0)   # Drehung um 90 Grad
T_schieb = transformationsmatrix(0, 0.2, 0)            # Verschiebung um 0.2 m in x
werkstueck = np.array([[0.1, 0.0]])

print(np.round(transformiere(T_schieb @ T_dreh, werkstueck), 4))
print(np.round(transformiere(T_dreh @ T_schieb, werkstueck), 4))
print(np.round(T_schieb @ T_dreh, 4))
print(np.round(T_dreh @ T_schieb, 4))
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Welche Bewegung wirkt bei `T_schieb @ T_dreh` zuerst? Wo landet das
   Werkstück?
2. Wo landet das Werkstück bei `T_dreh @ T_schieb`?
3. Eines der beiden Produkte ist gleich
   `transformationsmatrix(np.deg2rad(90), 0.2, 0)`. Welches, und warum? Welche
   Verschiebung steht in der rechten Spalte des anderen Produkts?
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

T_dreh = transformationsmatrix(np.deg2rad(90), 0, 0)
T_schieb = transformationsmatrix(0, 0.2, 0)
werkstueck = np.array([[0.1, 0.0]])

print(np.round(transformiere(T_schieb @ T_dreh, werkstueck), 4))
print(np.round(transformiere(T_dreh @ T_schieb, werkstueck), 4))
print(np.round(T_schieb @ T_dreh, 4))
print(np.round(T_dreh @ T_schieb, 4))
```
Ausgabe:

```text
[[0.2 0.1]]
[[0.  0.3]]
[[ 0.  -1.   0.2]
 [ 1.   0.   0. ]
 [ 0.   0.   1. ]]
[[ 0.  -1.   0. ]
 [ 1.   0.   0.2]
 [ 0.   0.   1. ]]
```

In einem Matrixprodukt wirkt die rechte Matrix zuerst. Bei
`T_schieb @ T_dreh` dreht der Tisch das Werkstück zuerst nach $(0, 0.1)$ und
verschiebt es dann nach $(0.2, 0.1)$. Bei `T_dreh @ T_schieb` wird es zuerst
nach $(0.3, 0)$ verschoben und dann um den Ursprung nach $(0, 0.3)$ gedreht.
`transformationsmatrix` dreht nach ihrer Definition zuerst und verschiebt
danach. Deshalb ist `T_schieb @ T_dreh` gleich
`transformationsmatrix(np.deg2rad(90), 0.2, 0)`. Im anderen Produkt steht die
Verschiebung $(0, 0.2)$, denn die Drehung hat den Verschiebungsvektor
mitgedreht. Sobald Verschiebungen beteiligt sind, ist das Matrixprodukt also
nicht mehr vertauschbar. Das Werkstück ist ein Array der Form `(1, 2)`, weil
`transformiere` die Punkte zeilenweise erwartet.
````

```{admonition} Aufgabe 5.5 (✩✩)
:class: tip
Eine CNC-Bohrmaschine soll sechs Bohrungen auf dem **Lochkreis** eines
Flanschs setzen, also auf einem Kreis um die Flanschmitte. Die Flanschmitte
liegt auf dem Maschinentisch bei $M = (120, 80)$ in mm, der Lochkreis hat den
Radius $50\,\text{mm}$. Die erste Bohrung liegt rechts neben der Mitte bei
$(170, 80)$, die übrigen folgen gegen den Uhrzeigersinn im Abstand von $60°$.

1. Berechnen Sie mit `drehung_um_punkt` und `transformiere` die Positionen
   aller sechs Bohrungen und speichern Sie sie in einem Array `bohrungen` der
   Form `(6, 2)`. Bohrung $k$ entsteht, indem Sie die erste Bohrung um
   $k \cdot 60°$ um $M$ drehen.
2. Prüfen Sie mit Pythagoras, dass alle Bohrungen $50\,\text{mm}$ von $M$
   entfernt sind. Wie weit liegen zwei benachbarte Bohrungen auseinander?
   Begründen Sie das Ergebnis geometrisch.
3. Der Flansch wurde versehentlich um $15°$ verdreht eingespannt. Berechnen
   Sie die tatsächlichen Positionen mit einer einzigen Transformation und
   stellen Sie Soll- und Ist-Positionen mit `ax.scatter` in einem Diagramm
   dar. Damit der Lochkreis rund aussieht, ergänzen Sie
   `ax.set_aspect('equal')`. Um wie viele Millimeter liegt jede Bohrung neben
   ihrer Sollposition?

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
mitte = np.array([120.0, 80.0])            # Flanschmitte in mm
erste_bohrung = np.array([[170.0, 80.0]])  # als Zeile für transformiere

# Verarbeitung: Bohrung k durch Drehung um k * 60 Grad um die Mitte
bohrungen = np.zeros((6, 2))
for k in range(6):
    T = drehung_um_punkt(np.deg2rad(60 * k), mitte)
    bohrungen[k] = transformiere(T, erste_bohrung)[0]

# Abstände aller Bohrungen zur Mitte
differenz = bohrungen - mitte
abstand_mitte = np.sqrt(differenz[:, 0]**2 + differenz[:, 1]**2)

# Abstand der benachbarten Bohrungen 0 und 1
differenz = bohrungen[1] - bohrungen[0]
abstand_nachbar = np.sqrt(differenz[0]**2 + differenz[1]**2)

# Flansch um 15 Grad verdreht: eine Drehung um die Mitte für alle Bohrungen
T_verdreht = drehung_um_punkt(np.deg2rad(15), mitte)
bohrungen_ist = transformiere(T_verdreht, bohrungen)
differenz = bohrungen_ist - bohrungen
versatz = np.sqrt(differenz[:, 0]**2 + differenz[:, 1]**2)

# Ausgabe
print('Bohrungen in mm:')
print(np.round(bohrungen, 2))
print('Abstand zur Mitte in mm:', np.round(abstand_mitte, 2))
print(f'Abstand benachbarter Bohrungen: {abstand_nachbar:.2f} mm')
print('Versatz durch die Verdrehung in mm:', np.round(versatz, 2))

fig, ax = plt.subplots(figsize=(5, 5))
ax.scatter(bohrungen[:, 0], bohrungen[:, 1], label='Soll')
ax.scatter(bohrungen_ist[:, 0], bohrungen_ist[:, 1], marker='x',
           label='Ist, um 15° verdreht')
ax.scatter(mitte[0], mitte[1], marker='+', color='black', label='Flanschmitte')
ax.set_aspect('equal')
ax.set_xlabel('x in mm')
ax.set_ylabel('y in mm')
ax.set_title('Lochkreis des Flanschs')
ax.legend()
ax.grid(True)
plt.show()
```
Ausgabe:

```text
Bohrungen in mm:
[[170.    80.  ]
 [145.   123.3 ]
 [ 95.   123.3 ]
 [ 70.    80.  ]
 [ 95.    36.7 ]
 [145.    36.7 ]]
Abstand zur Mitte in mm: [50. 50. 50. 50. 50. 50.]
Abstand benachbarter Bohrungen: 50.00 mm
Versatz durch die Verdrehung in mm: [13.05 13.05 13.05 13.05 13.05 13.05]
```

Die Funktionen rechnen in jeder Längeneinheit, hier in mm. Alle Bohrungen
liegen $50\,\text{mm}$ von der Mitte entfernt, denn eine Drehung um $M$ ändert
keine Abstände zu $M$. Zwei benachbarte Bohrungen bilden mit der Mitte ein
Dreieck mit zwei Seiten von $50\,\text{mm}$ und einem Winkel von $60°$
dazwischen. Dieses Dreieck ist gleichseitig, deshalb liegen benachbarte
Bohrungen ebenfalls $50\,\text{mm}$ auseinander. Durch die Verdrehung wandert
jede Bohrung auf ihrem Kreisbogen um $13.05\,\text{mm}$, alle gleich weit,
weil alle gleich weit von $M$ entfernt sind. Damit wäre der Flansch Ausschuss.
CNC-Steuerungen messen deshalb vor dem Bohren die Lage des Werkstücks und
drehen das Bohrbild mit genau dieser Transformation mit.
````

```{admonition} Aufgabe 5.6 (✩✩)
:class: tip
In Kapitel 5.4 haben wir gesehen, dass Arm 2 immer um $\alpha + \beta$ gegen
die $x$-Achse gedreht ist. Daraus lässt sich die Greiferposition auch ohne
Matrizen berechnen.

1. Stellen Sie ohne Code Formeln für die Koordinaten $x$ und $y$ des Greifers
   auf. Gelenk 2 liegt auf einem Kreis mit dem Radius $l_1 = 0.4\,\text{m}$ um
   den Ursprung, wie der Greifer in Kapitel 5.1. Arm 2 hat die Länge
   $l_2 = 0.3\,\text{m}$ und zeigt von Gelenk 2 aus in Richtung des Winkels
   $\alpha + \beta$.
2. Schreiben Sie eine Funktion `greifer_formel(alpha, beta)` mit Docstring, die
   die Greiferposition für Winkel in Radiant als Array zurückgibt.
3. Vergleichen Sie Ihre Funktion für die Winkelpaare $(30°, 60°)$,
   $(9°, 99°)$, $(120°, 45°)$ und $(180°, 150°)$ mit `np.allclose` mit dem
   Greifer aus `stellung`.
4. Die Formel ist viel kürzer als die Kette aus Transformationsmatrizen.
   Warum rechnen Robotersteuerungen trotzdem mit Matrizen?

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
def greifer_formel(alpha, beta):
    """Greiferposition des SCARA-Roboters in m, Winkel in Radiant.

    Gelenk 2 liegt bei l1 * (cos alpha, sin alpha), Arm 2 zeigt in Richtung
    alpha + beta.
    """
    l1 = 0.4
    l2 = 0.3
    x = l1 * np.cos(alpha) + l2 * np.cos(alpha + beta)
    y = l1 * np.sin(alpha) + l2 * np.sin(alpha + beta)
    return np.array([x, y])

winkelpaare = np.array([[30, 60], [9, 99], [120, 45], [180, 150]])

# Verarbeitung und Ausgabe
for k in range(len(winkelpaare)):
    alpha = np.deg2rad(winkelpaare[k, 0])
    beta = np.deg2rad(winkelpaare[k, 1])
    greifer = greifer_formel(alpha, beta)
    arm_neu, bauteil_neu = stellung(alpha, beta)
    print(f'({winkelpaare[k, 0]:3d}°, {winkelpaare[k, 1]:3d}°): '
          f'Greifer bei {np.round(greifer, 4)} m, '
          f'gleich wie stellung: {np.allclose(greifer, arm_neu[2])}')
```
Ausgabe:

```text
( 30°,  60°): Greifer bei [0.3464 0.5   ] m, gleich wie stellung: True
(  9°,  99°): Greifer bei [0.3017 0.3479] m, gleich wie stellung: True
(120°,  45°): Greifer bei [-0.4898  0.4241] m, gleich wie stellung: True
(180°, 150°): Greifer bei [-0.1402 -0.15  ] m, gleich wie stellung: True
```

Gelenk 2 liegt bei $l_1 (\cos\alpha, \sin\alpha)$. Von dort aus liegt der
Greifer noch $l_2$ weiter in Richtung $\alpha + \beta$:

$$x = l_1 \cos\alpha + l_2 \cos(\alpha + \beta), \qquad
y = l_1 \sin\alpha + l_2 \sin(\alpha + \beta).$$

Für alle vier Winkelpaare stimmt die Formel mit `stellung` überein. Für den
Greifer allein ist sie tatsächlich kürzer. Die Steuerung braucht aber auch
die Ecken des Bauteils, und viele Roboter haben mehr als zwei Gelenke, die
sich im Raum drehen. Jede weitere Größe bräuchte eine eigene, immer längere
Formel. Mit Matrizen bekommt jedes Gelenk eine eigene Matrix, und die Kette
aus Matrixprodukten sieht für jeden Roboter gleich aus. Deshalb beschreiben
Robotersteuerungen ihre Arme mit homogenen Transformationsmatrizen.
````

```{admonition} Aufgabe 5.7 (✩✩)
:class: tip
Welche Punkte kann der Greifer des SCARA-Roboters überhaupt erreichen? Die
Menge dieser Punkte heißt **Arbeitsraum** des Roboters. Gelenk 1 dreht wie in
Kapitel 5.4 zwischen $0°$ und $180°$, Gelenk 2 zwischen $0°$ und $150°$.

1. Berechnen Sie mit `stellung` die Greiferposition für alle Winkel $\alpha$
   von $0°$ bis $180°$ und $\beta$ von $0°$ bis $150°$ in Schritten von $5°$.
   Speichern Sie die Positionen in einem Array `punkte` mit
   $37 \cdot 31 = 1147$ Zeilen und zwei Spalten. Legen Sie es vorher mit
   `np.zeros` an und zählen Sie in den Schleifen einen Zeilenindex mit.
2. Stellen Sie die Punkte mit `ax.scatter(..., s=2)` dar. Ergänzen Sie
   `ax.set_aspect('equal')`, damit Kreise rund aussehen.
3. Berechnen Sie den kleinsten und den größten Abstand der Punkte vom
   Ursprung.
4. Beantworten Sie ohne Code: Warum hat der Arbeitsraum in der Mitte ein
   Loch? Kann der Roboter ein Bauteil bei $(0.10, 0.15)$ absetzen?

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
alphas_grad = np.linspace(0, 180, 37)
betas_grad = np.linspace(0, 150, 31)

# Verarbeitung: Greiferposition für jede Winkelkombination
punkte = np.zeros((len(alphas_grad) * len(betas_grad), 2))
zeile = 0
for alpha_grad in alphas_grad:
    for beta_grad in betas_grad:
        arm_neu, bauteil_neu = stellung(np.deg2rad(alpha_grad), np.deg2rad(beta_grad))
        punkte[zeile] = arm_neu[2]
        zeile += 1

abstaende = np.sqrt(punkte[:, 0]**2 + punkte[:, 1]**2)

# Ausgabe
print('Form von punkte:', punkte.shape)
print(f'Kleinster Abstand vom Ursprung: {np.min(abstaende):.3f} m')
print(f'Größter Abstand vom Ursprung:   {np.max(abstaende):.3f} m')

fig, ax = plt.subplots(figsize=(6, 6))
ax.scatter(punkte[:, 0], punkte[:, 1], s=2)
ax.set_aspect('equal')
ax.set_xlabel('x in m')
ax.set_ylabel('y in m')
ax.set_title('Arbeitsraum des SCARA-Roboters')
ax.grid(True)
plt.show()
```
Ausgabe:

```text
Form von punkte: (1147, 2)
Kleinster Abstand vom Ursprung: 0.205 m
Größter Abstand vom Ursprung:   0.700 m
```

Der Arbeitsraum ist ein Halbring mit dem Außenradius
$l_1 + l_2 = 0.7\,\text{m}$, den der gestreckte Arm erreicht. Der Abstand des
Greifers vom Ursprung hängt nur vom Knickwinkel $\beta$ ab, denn $\alpha$
dreht den ganzen Arm um den Ursprung. Je stärker Gelenk 2 abknickt, desto
näher kommt der Greifer dem Sockel. Weil $\beta$ höchstens $150°$ beträgt,
kann Arm 2 nicht ganz auf Arm 1 zurückklappen, und in der Mitte bleibt ein
Loch mit dem Radius $0.205\,\text{m}$. Der Punkt $(0.10, 0.15)$ liegt
$\sqrt{0.10^2 + 0.15^2} \approx 0.180\,\text{m}$ vom Ursprung entfernt, also
in diesem Loch. Dort kann der Roboter kein Bauteil absetzen. Weil Arm 2 gegen
den Uhrzeigersinn abknickt, ist der Halbring etwas gedreht: Rechts reicht er
nicht ganz bis zur $x$-Achse herunter, links ein Stück darunter.
````

```{admonition} Aufgabe 5.8 (✩✩)
:class: tip
In Kapitel 5.4 hat die Rastersuche mit Schritten von $1°$ insgesamt
$27\,331$ Stellungen berechnet und den Bestückplatz $P_1 = (0.30, 0.35)$
trotzdem um $3.2\,\text{mm}$ verfehlt. Ein feineres Raster über den ganzen
Winkelbereich wäre zu teuer. Wir suchen deshalb in zwei Stufen: erst grob über
den ganzen Bereich, dann fein in der Umgebung des groben Ergebnisses.

1. Schreiben Sie eine Funktion `suche(ziel, alphas_grad, betas_grad)`, die
   alle Kombinationen der Winkel aus den beiden Arrays durchprobiert. Sie soll
   die besten Winkel in Grad und den Restabstand zurückgeben.
2. Stufe 1: Suchen Sie für $P_1$ mit Schritten von $5°$ über den ganzen
   Bereich, $\alpha$ von $0°$ bis $180°$ und $\beta$ von $0°$ bis $150°$.
3. Stufe 2: Suchen Sie mit Schritten von $0.1°$ in einem Fenster von
   $\pm 5°$ um die Winkel aus Stufe 1. Wie viele Stellungen berechnen beide
   Stufen zusammen, und wie groß ist der Restabstand? Vergleichen Sie mit der
   Suche aus Kapitel 5.4.
4. Wiederholen Sie beide Stufen für $P_2 = (0.40, 0.40)$, einmal mit einem
   Fenster von $\pm 5°$ und einmal mit $\pm 1°$. Warum versagt das kleine
   Fenster?

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
def suche(ziel, alphas_grad, betas_grad):
    """Probiert alle Winkelkombinationen in Grad durch.

    Rückgabe: bestes alpha, bestes beta in Grad und Restabstand in m
    """
    bester_abstand = 10.0
    for alpha_grad in alphas_grad:
        for beta_grad in betas_grad:
            arm_neu, bauteil_neu = stellung(np.deg2rad(alpha_grad), np.deg2rad(beta_grad))
            differenz = arm_neu[2] - ziel
            abstand = np.sqrt(differenz[0]**2 + differenz[1]**2)
            if abstand < bester_abstand:
                bester_abstand = abstand
                bestes_alpha = alpha_grad
                bestes_beta = beta_grad
    return bestes_alpha, bestes_beta, bester_abstand

p1 = np.array([0.30, 0.35])
p2 = np.array([0.40, 0.40])

# Verarbeitung und Ausgabe, Stufe 1: grobes Raster über den ganzen Bereich
alpha_grob, beta_grob, abstand_grob = suche(p1, np.linspace(0, 180, 37),
                                            np.linspace(0, 150, 31))
print(f'P1 grob: α = {alpha_grob:.1f}°, β = {beta_grob:.1f}°, '
      f'Rest {abstand_grob * 1000:.2f} mm')

# Stufe 2: feines Raster mit 0.1 Grad im Fenster von +-5 Grad
alpha_fein, beta_fein, abstand_fein = suche(
    p1, np.linspace(alpha_grob - 5, alpha_grob + 5, 101),
    np.linspace(beta_grob - 5, beta_grob + 5, 101))
print(f'P1 fein: α = {alpha_fein:.1f}°, β = {beta_fein:.1f}°, '
      f'Rest {abstand_fein * 1000:.2f} mm')
print('Stellungen insgesamt:', 37 * 31 + 101 * 101)

# Teilaufgabe 4: P2 mit großem und kleinem Fenster
alpha_grob, beta_grob, abstand_grob = suche(p2, np.linspace(0, 180, 37),
                                            np.linspace(0, 150, 31))
print(f'P2 grob: α = {alpha_grob:.1f}°, β = {beta_grob:.1f}°, '
      f'Rest {abstand_grob * 1000:.2f} mm')
for fenster in [5, 1]:
    anzahl = int(20 * fenster + 1)   # Schritte von 0.1 Grad
    alpha_fein, beta_fein, abstand_fein = suche(
        p2, np.linspace(alpha_grob - fenster, alpha_grob + fenster, anzahl),
        np.linspace(beta_grob - fenster, beta_grob + fenster, anzahl))
    print(f'P2 fein, Fenster ±{fenster}°: α = {alpha_fein:.1f}°, '
          f'β = {beta_fein:.1f}°, Rest {abstand_fein * 1000:.2f} mm')
```
Ausgabe:

```text
P1 grob: α = 10.0°, β = 100.0°, Rest 8.79 mm
P1 fein: α = 9.4°, β = 99.0°, Rest 0.07 mm
Stellungen insgesamt: 11348
P2 grob: α = 15.0°, β = 70.0°, Rest 12.74 mm
P2 fein, Fenster ±5°: α = 14.5°, β = 73.1°, Rest 0.21 mm
P2 fein, Fenster ±1°: α = 14.0°, β = 71.0°, Rest 6.02 mm
```

Das grobe Raster verfehlt $P_1$ um fast $9\,\text{mm}$, zeigt aber, wo sich
das Suchen lohnt. Beide Stufen zusammen berechnen $11\,348$ Stellungen, weniger
als halb so viele wie die $1°$-Suche aus Kapitel 5.4. Der Restabstand sinkt
dabei von $3.2\,\text{mm}$ auf $0.07\,\text{mm}$. Ein feines Raster über den
ganzen Bereich bräuchte für dieselbe Genauigkeit rund 2.7 Millionen
Stellungen. Bei $P_2$ liegt der beste Winkel $\beta \approx 73.1°$ gut $3°$
neben dem groben Ergebnis $70°$. Das Fenster von $\pm 1°$ reicht nur bis
$71°$, und die feine Suche endet am Rand des Fensters mit $6\,\text{mm}$
Rest. Das Fenster muss deshalb mindestens so groß sein wie eine Schrittweite
des groben Rasters. Ein Hinweis auf ein zu kleines Fenster ist ein Ergebnis
genau am Fensterrand.
````

`````{admonition} Aufgabe 5.9 (✩✩✩) Mini-Projekt: Eine Drehachse am Greifer
:class: tip
In Kapitel 5.4 lagen die Bauteile schräg auf der Platine, weil das Bauteil
starr an Arm 2 sitzt und immer um $\alpha + \beta$ gedreht ist. Wir rüsten den
SCARA-Roboter deshalb mit einer dritten Drehachse direkt am Greifer aus. Sie
dreht das Bauteil um den Winkel $\gamma$ um seine eigene Mitte, also um den
Greifer. In der Grundstellung $\alpha = \beta = \gamma = 0$ steht der Greifer
bei $(0.7, 0)$. Der Bestückplan legt für jeden Platz fest, unter welchem
Winkel gegen die $x$-Achse die lange Kante des Bauteils liegen soll:

| Platz | Lage in m | Winkel aus Kapitel 5.4 | Sollwinkel des Bauteils |
| --- | --- | --- | --- |
| $P_1$ | $(0.30, 0.35)$ | $\alpha = 9°$, $\beta = 99°$ | $0°$ |
| $P_2$ | $(0.40, 0.40)$ | $\alpha = 14°$, $\beta = 74°$ | $90°$ |
| $P_3$ | $(0.45, 0.30)$ | $\alpha = 1°$, $\beta = 79°$ | $45°$ |

Zuführung, Platine und Bestückplätze aus Kapitel 5.4:

```python
zufuehrung = np.array([[0.62, -0.06], [0.78, -0.06], [0.78, 0.06], [0.62, 0.06]])
platine = np.array([[0.25, 0.25], [0.50, 0.25], [0.50, 0.45], [0.25, 0.45]])
bestueckplaetze = np.array([[0.30, 0.35], [0.40, 0.40], [0.45, 0.30]])
```

**Teil 1:** Schreiben Sie eine Funktion
`stellung_mit_greiferachse(alpha, beta, gamma)`, die wie `stellung` die Punkte
des Arms und die Ecken des Bauteils zurückgibt. Das Bauteil transformieren
Sie mit $\mathbf{T}_\alpha \cdot \mathbf{T}_\beta \cdot \mathbf{T}_\gamma$,
wobei $\mathbf{T}_\gamma$ um die Lage des Greifers in der Grundstellung dreht.
Prüfen Sie für $\alpha = 30°$ und $\beta = 60°$: Mit $\gamma = 0$ liefert die
Funktion dasselbe wie `stellung`. Mit $\gamma = 90°$ bleibt die Mitte des
Bauteils beim Greifer. Die Mitte ist der Mittelpunkt der Diagonale, also
`(ecken[0] + ecken[2]) / 2`.

**Teil 2:** Berechnen Sie für jeden Platz den Winkel $\gamma$, der das Bauteil
vom Winkel $\alpha + \beta$ auf den Sollwinkel dreht. Prüfen Sie mit
`np.allclose`, dass die lange Kante `ecken[1] - ecken[0]` danach gleich
$0.08 \cdot (\cos\varphi_\text{soll}, \sin\varphi_\text{soll})$ ist.

**Teil 3:** Die Greiferachse kann nur zwischen $-90°$ und $+90°$ drehen, damit
sich die Kabel zum Greifer nicht aufwickeln. Prüfen Sie mit einer
`if`-Abfrage, welcher Platz außerhalb dieses Bereichs liegt. Das Bauteil ist
ein Rechteck und sieht nach einer halben Umdrehung genauso aus wie vorher.
Korrigieren Sie $\gamma$ für diesen Platz um $180°$. Zeichnen Sie dann alle
drei Stellungen mit ihren Bauteilen, der Zuführung und der Platine.

**Abschlussfrage:** Warum muss $\mathbf{T}_\gamma$ in der Kette ganz rechts
stehen? Was würde mit dem Bauteil passieren, wenn wir
$\mathbf{T}_\gamma \cdot \mathbf{T}_\alpha \cdot \mathbf{T}_\beta$
verwenden? Und warum ist die Korrektur aus Teil 3 bei gepolten Bauteilen wie
Dioden nicht erlaubt?

Strukturieren Sie Ihren Code mit EVA-Kommentaren.
`````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Eingabe
zufuehrung = np.array([[0.62, -0.06], [0.78, -0.06], [0.78, 0.06], [0.62, 0.06]])
platine = np.array([[0.25, 0.25], [0.50, 0.25], [0.50, 0.45], [0.25, 0.45]])
bestueckplaetze = np.array([[0.30, 0.35], [0.40, 0.40], [0.45, 0.30]])

# Winkel aus Kapitel 5.4 und Sollwinkel des Bauteils in Grad, Zeile = Platz
bestueckplan = np.array([
    [9, 99, 0],
    [14, 74, 90],
    [1, 79, 45],
])

def stellung_mit_greiferachse(alpha, beta, gamma):
    """Punkte des Arms und Ecken des Bauteils mit drehbarem Greifer, Winkel in Radiant."""
    T_alpha = transformationsmatrix(alpha, 0, 0)
    T_beta = drehung_um_punkt(beta, arm[1])
    # Greiferachse dreht um die Lage des Greifers in der Grundstellung
    T_gamma = drehung_um_punkt(gamma, arm[2])
    arm1_neu = transformiere(T_alpha, arm[0:2])
    arm2_neu = transformiere(T_alpha @ T_beta, arm[1:3])
    arm_neu = np.array([arm1_neu[0], arm1_neu[1], arm2_neu[1]])
    # T_gamma steht rechts, weil es um die Grundstellung des Greifers dreht
    bauteil_neu = transformiere(T_alpha @ T_beta @ T_gamma, bauteil)
    return arm_neu, bauteil_neu

# Verarbeitung Teil 1: Funktion prüfen
alpha = np.deg2rad(30)
beta = np.deg2rad(60)
arm_ohne, bauteil_ohne = stellung(alpha, beta)
arm_0, bauteil_0 = stellung_mit_greiferachse(alpha, beta, 0)
arm_90, bauteil_90 = stellung_mit_greiferachse(alpha, beta, np.deg2rad(90))
mitte_90 = (bauteil_90[0] + bauteil_90[2]) / 2

print('Teil 1')
print('gamma = 0 wie stellung:', np.allclose(arm_0, arm_ohne)
      and np.allclose(bauteil_0, bauteil_ohne))
print('gamma = 90°, Mitte beim Greifer:', np.allclose(mitte_90, arm_90[2]))

# Verarbeitung Teil 2 und 3: gamma für jeden Platz, Achsbereich prüfen
print('Teil 2 und 3')
arme = []
bauteile = []
for k in range(len(bestueckplan)):
    alpha_grad = bestueckplan[k, 0]
    beta_grad = bestueckplan[k, 1]
    soll_grad = bestueckplan[k, 2]
    gamma_grad = soll_grad - (alpha_grad + beta_grad)
    print(f'P{k + 1}: γ = {gamma_grad}°')
    if gamma_grad < -90 or gamma_grad > 90:
        # Rechteck sieht nach einer halben Umdrehung gleich aus
        gamma_grad = gamma_grad + 180
        print(f'    außerhalb von ±90°, korrigiert auf γ = {gamma_grad}°')

    arm_neu, bauteil_neu = stellung_mit_greiferachse(
        np.deg2rad(alpha_grad), np.deg2rad(beta_grad), np.deg2rad(gamma_grad))
    arme.append(arm_neu)
    bauteile.append(bauteil_neu)

    # Lange Kante mit der Sollrichtung vergleichen
    kante = bauteil_neu[1] - bauteil_neu[0]
    soll = 0.08 * np.array([np.cos(np.deg2rad(soll_grad)),
                            np.sin(np.deg2rad(soll_grad))])
    print(f'    Kante in Sollrichtung: {np.allclose(kante, soll)}, '
          f'in Gegenrichtung: {np.allclose(kante, -soll)}')

# Ausgabe Teil 3
zeichne_roboter(arme, bauteile=bauteile, ziele=bestueckplaetze,
                umgebung=[zufuehrung, platine], beschriftungen=['P1', 'P2', 'P3'],
                titel='Bestücken mit drehbarem Greifer')
```
Ausgabe:

```text
Teil 1
gamma = 0 wie stellung: True
gamma = 90°, Mitte beim Greifer: True
Teil 2 und 3
P1: γ = -108°
    außerhalb von ±90°, korrigiert auf γ = 72°
    Kante in Sollrichtung: False, in Gegenrichtung: True
P2: γ = 2°
    Kante in Sollrichtung: True, in Gegenrichtung: False
P3: γ = -35°
    Kante in Sollrichtung: True, in Gegenrichtung: False
```

Mit $\gamma = 0$ verhält sich der Roboter wie bisher, und bei jedem $\gamma$
bleibt die Mitte des Bauteils beim Greifer. Die Greiferachse ändert also nur
die Orientierung, nicht die Lage. Das Bauteil ist jetzt um
$\alpha + \beta + \gamma$ gedreht. Für den Sollwinkel muss deshalb
$\gamma = \varphi_\text{soll} - (\alpha + \beta)$ gelten. Bei $P_1$ ergibt das
$-108°$, und die Achse kommt nicht so weit. Mit $\gamma = 72°$ ist das Bauteil
um $180°$ weitergedreht. Die Kante zeigt dann in die Gegenrichtung, das
Rechteck bedeckt aber dieselbe Fläche. Im Bild liegen alle drei Bauteile wie
im Bestückplan auf der Platine: längs, quer und unter $45°$.

**Abschlussfrage:** $\mathbf{T}_\gamma$ dreht um den Punkt $(0.7, 0)$, an dem
der Greifer nur in der Grundstellung steht. Die Matrix muss deshalb wirken,
solange Arm 1 und Arm 2 noch in der Grundstellung liegen, also ganz rechts in
der Kette. Genauso dreht $\mathbf{T}_\beta$ um die Lage von Gelenk 2 in der
Grundstellung und steht rechts von $\mathbf{T}_\alpha$. Bei
$\mathbf{T}_\gamma \cdot \mathbf{T}_\alpha \cdot \mathbf{T}_\beta$ würde der
Arm zuerst in seine Stellung fahren, und erst danach würde das Bauteil um
$(0.7, 0)$ gedreht. Dort ist aber kein Greifer mehr, und das Bauteil würde vom
Greifer weggerissen, so wie Arm 2 bei vertauschter Reihenfolge in Kapitel 5.3.

Bei gepolten Bauteilen wie Dioden oder Elektrolytkondensatoren sind die beiden
Enden nicht gleichwertig. Eine halbe Umdrehung vertauscht Plus und Minus, und
das Bauteil wäre falsch herum eingebaut. Die Anlage müsste dann zum Beispiel
das Bauteil schon um $180°$ gedreht in der Zuführung bereitstellen oder eine
Greiferachse mit größerem Drehbereich verwenden. Dürfte Gelenk 2 auch im
Uhrzeigersinn abknicken, ginge es auch mit der gespiegelten Armstellung aus der
Abschlussfrage von Kapitel 5.4. Dort ist $\alpha + \beta \approx -10°$, und für
$P_1$ genügt $\gamma \approx 10°$.
````
