---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 6.1 Fahrzeug- und Weltkoordinaten

In einer Logistikhalle bringt ein **fahrerloses Transportsystem**, kurz FTS,
Kisten von den Regalen zu einer Übergabestation. Das Fahrzeug fährt ohne
Fahrer und orientiert sich mit einem Laserscanner. Der Scanner sitzt oben auf
dem Fahrzeug und misst, wie weit entfernt und in welcher Richtung Gegenstände
liegen. Meldet er ein Hindernis $1.5\,\text{m}$ voraus, weiß die Steuerung
zunächst nur, wo das Hindernis relativ zum Fahrzeug liegt. *Wo steht das
Hindernis in der Halle?*

In Kapitel 5.3 haben wir mit homogenen Transformationsmatrizen Punkte gedreht
und verschoben, alle in einem gemeinsamen Koordinatensystem. In diesem Kapitel
lernen wir eine zweite Sichtweise kennen: Die Halle und das Fahrzeug haben
jeweils ein eigenes Koordinatensystem, und dieselbe Matrix rechnet Koordinaten
vom einen in das andere um. Damit bestimmen wir, wo das Fahrzeug steht, wo die
Messpunkte des Scanners in der Halle liegen und wo ein Ziel aus Sicht des
Fahrzeugs liegt.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie wissen, was ein **Weltkoordinatensystem** und ein
  **Fahrzeugkoordinatensystem** sind, und können die Lage eines Fahrzeugs durch
  seine **Pose** beschreiben.
* [ ] Sie können die Transformationsmatrix $\mathbf{T}_{WF}$ aufbauen und mit
  ihr Punkte vom Fahrzeugkoordinatensystem in das Weltkoordinatensystem
  umrechnen.
* [ ] Sie können Messungen aus Entfernung und Winkel in Fahrzeugkoordinaten
  umrechnen.
* [ ] Sie können mit der inversen Matrix $\mathbf{T}_{WF}^{-1}$ Punkte vom
  Weltkoordinatensystem in das Fahrzeugkoordinatensystem umrechnen.
```

## Wo steht das Fahrzeug in der Halle?

Wie beim SCARA-Roboter schauen wir von oben auf die Halle. Für die Rechnung
brauchen wir zwei Koordinatensysteme:

* Das **Weltkoordinatensystem** W ist fest mit der Halle verbunden. Sein
  Ursprung liegt in der linken unteren Ecke unseres Hallenausschnitts, die
  $x_W$-Achse zeigt entlang der langen Hallenwand. In diesem Koordinatensystem
  sind alle Regale und Stationen der Hallenkarte angegeben.
* Das **Fahrzeugkoordinatensystem** F ist fest mit dem Fahrzeug verbunden und
  fährt mit ihm mit. Sein Ursprung liegt in der Fahrzeugmitte, die $x_F$-Achse
  zeigt nach vorne, die $y_F$-Achse nach links, so wie es die Norm DIN ISO 8855
  für Fahrzeuge festlegt.

```{figure} pics/chap06_koordinatensysteme.svg
:alt: Draufsicht auf eine Hallenecke mit dem Weltkoordinatensystem W, einem um den Winkel psi gedrehten Fahrzeug an der Stelle x, y mit seinen Achsen x_F nach vorne und y_F nach links sowie einem Laserstrahl der Länge r unter dem Winkel phi zu einem Hindernis
:align: center

Das Weltkoordinatensystem W, das Fahrzeugkoordinatensystem F mit der Pose
$(x, y, \psi)$ und eine Messung des Laserscanners mit Entfernung $r$ und
Winkel $\varphi$.
(Quelle: eigene Abbildung; Lizenz [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0))
```

Wo das Fahrzeug steht und wohin es zeigt, also seine Position und seine
Orientierung aus Kapitel 5.1, beschreiben drei Zahlen: die Lage
$(x, y)$ der Fahrzeugmitte in Hallenkoordinaten und der **Gierwinkel** $\psi$
(psi) zwischen der $x_W$-Achse und der $x_F$-Achse. Diese drei Zahlen zusammen
heißen **Pose** des Fahrzeugs. Ein positiver Gierwinkel bedeutet wie in
Kapitel 5.1 eine Drehung gegen den Uhrzeigersinn.

Unser FTS ist $1.2\,\text{m}$ lang und $0.8\,\text{m}$ breit. Seine Ecken liegen
im Fahrzeugkoordinatensystem immer an denselben Stellen, egal wo das Fahrzeug
gerade fährt. Wir legen sie deshalb einmal als Array `umriss` an. Die ersten
beiden Ecken bilden die Vorderkante.

Zuerst führen wir zwei Zellen aus. Die erste enthält die vorgegebene
Zeichenfunktion `zeichne_halle`, die wie `zeichne_roboter` aus Part 5 nicht
im Detail verstanden werden muss. Die zweite enthält die Funktionen
`transformationsmatrix` und `transformiere` aus Kapitel 5.3, den Umriss und die
Hallenkarte mit einem Regal, einer Palette und einer Säule.

```{code-cell} python
:tags: [hide-input]
# Vorgegebene Zeichenfunktion: einfach ausführen, sie muss nicht verstanden werden.
def zeichne_halle(fahrzeuge=None, koordinatensysteme=None, messpunkte=None,
                  scanner=None, ziele=None, bahn=None, umgebung=None,
                  beschriftungen=None, titel='', bereich=(0, 10, 0, 8)):
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
    ziele: Zielpunkte als Array, Zeile = [x, y], werden als Kreuze markiert (optional)
    bahn: Bahnpunkte als Array, Zeile = [x, y], werden als gestrichelte Linie
          mit Punkten gezeichnet (optional)
    umgebung: Ecken von Flächen wie Regalen oder Paletten als Array oder Liste
              von Arrays, werden grau gefüllt (optional)
    beschriftungen: Legendentexte, einer pro Fahrzeug (optional)
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

    # Bahn als gestrichelte Linie mit Punkten
    if bahn is not None:
        ax.plot(bahn[:, 0], bahn[:, 1], color=grau, linestyle='--', linewidth=1,
                marker='o', markersize=3, label='Bahn', zorder=1)

    # Fahrzeuge: Umriss hellblau gefüllt, Vorderkante (Ecke 0 bis Ecke 1) orange
    if fahrzeuge is not None:
        for k in range(len(fahrzeuge)):
            farbe = farben[k % len(farben)]
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
            or bahn is not None):
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

# Ecken des FTS im Fahrzeugkoordinatensystem in m, Zeile = [x, y]
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
```

Wie kommen die Ecken in Hallenkoordinaten? Stellen wir uns das Fahrzeug
zunächst so vor, dass seine Mitte im Ursprung der Halle steht und es in
$x_W$-Richtung zeigt. Dann stimmen Fahrzeug- und Hallenkoordinaten überein. Von
dort bringen wir es wie das Bauteil in Kapitel 5.3 in seine Pose: erst um
$\psi$ drehen, dann um $(x, y)$ verschieben. Das erledigt eine einzige homogene
Transformationsmatrix

$$\mathbf{T}_{WF} = \begin{pmatrix} \cos\psi & -\sin\psi & x \\
\sin\psi & \cos\psi & y \\ 0 & 0 & 1 \end{pmatrix}.$$

```{admonition} Indexschreibweise
:class: note
Die Indizes von $\mathbf{T}_{WF}$ lesen wir von rechts nach links: Der rechte
Index F nennt das System, aus dem die Koordinaten kommen, der linke Index W
das System, in das sie umgerechnet werden. Hat ein Punkt in homogenen
Koordinaten aus Kapitel 5.3 die Koordinaten $\tilde{p}_F$ im System F und
$\tilde{p}_W$ im System W, gilt also

$$\tilde{p}_W = \mathbf{T}_{WF} \cdot \tilde{p}_F.$$
```

Wir prüfen das an einem Fall, den wir im Kopf überblicken. Das Fahrzeug stehe
bei $(4, 2)$ und zeige mit $\psi = 90°$ in $y_W$-Richtung. Die Ecke vorne links
hat die Fahrzeugkoordinaten $(0.6, 0.4)$. Wie in Kapitel 5.3 rechnen wir
Zeile mal Spalte:

$$\begin{pmatrix} 0 & -1 & 4 \\ 1 & 0 & 2 \\ 0 & 0 & 1 \end{pmatrix}
\begin{pmatrix} 0.6 \\ 0.4 \\ 1 \end{pmatrix}
= \begin{pmatrix} 3.6 \\ 2.6 \\ 1 \end{pmatrix}.$$

Die Ecke liegt $0.6\,\text{m}$ weiter in $y_W$-Richtung als die Mitte, also
vorne, und $0.4\,\text{m}$ weiter in negativer $x_W$-Richtung. Für ein
Fahrzeug, das in $y_W$-Richtung zeigt, ist das links.

In unserer Halle steht das FTS bei $(4, 2)$ mit dem Gierwinkel $\psi = 30°$.
Wir bauen $\mathbf{T}_{WF}$ mit `transformationsmatrix` auf und rechnen den
Umriss mit `transformiere` in Hallenkoordinaten um. Der Parameter
`koordinatensysteme` von `zeichne_halle` zeichnet zusätzlich die Achsen des
Fahrzeugkoordinatensystems ein.

```{code-cell} python
# Pose des FTS: Mitte bei (4, 2) m, Gierwinkel 30 Grad
T_WF = transformationsmatrix(np.deg2rad(30), 4, 2)
print('T_WF:')
print(np.round(T_WF, 4))

# Ecken des Fahrzeugs von Fahrzeug- in Hallenkoordinaten umrechnen
umriss_W = transformiere(T_WF, umriss)
print('Ecken in Hallenkoordinaten in m:')
print(np.round(umriss_W, 3))

zeichne_halle(umriss_W, koordinatensysteme=T_WF, umgebung=halle,
              titel='FTS in der Halle')
```

Im Bild zeigt die rote Achse $x_F$ nach vorne und die blaue Achse $y_F$ nach
links. *Wo finden wir diese Achsen in der Matrix wieder?* In der ersten Spalte
von $\mathbf{T}_{WF}$ steht $(0.866, 0.5)$. Das ist die Richtung der
$x_F$-Achse in Hallenkoordinaten, also ein Einheitsvektor unter $30°$. Die
zweite Spalte $(-0.5, 0.866)$ ist die Richtung der $y_F$-Achse, und die dritte
Spalte $(4, 2)$ ist der Ursprung des Fahrzeugkoordinatensystems, also die
Fahrzeugmitte. Die Matrix enthält damit die ganze Pose: wo die Mitte liegt und
wohin die Achsen zeigen.

Wir können $\mathbf{T}_{WF}$ also auf zwei Arten lesen. Als Bewegung bringt
sie das Fahrzeug aus dem Ursprung in seine Pose, so wie wir in Kapitel 5.3 den
Roboterarm bewegt haben. Als Umrechnung beschreibt sie dieselben Ecken einmal
in Fahrzeug- und einmal in Hallenkoordinaten, ohne dass sich etwas bewegt.
Beide Sichtweisen liefern dieselben Zahlen.

```{admonition} Mini-Übung (✩)
:class: tip
1. Beantworten Sie ohne Code: Ein zweites FTS steht bei $(2, 5)$ mit
   $\psi = 180°$. In welche Richtung der Halle zeigt seine $x_F$-Achse? Wo
   liegt seine Ecke vorne links?
2. Prüfen Sie Ihre Antwort mit `transformationsmatrix` und `transformiere`.
   Zeichnen Sie beide Fahrzeuge mit ihren Koordinatensystemen in ein Bild.
3. Beantworten Sie ohne Code: Das erste FTS fährt $1\,\text{m}$ geradeaus.
   Welche Spalten von $\mathbf{T}_{WF}$ ändern sich, und welche bleiben gleich?
   Was ändert sich, wenn es sich stattdessen auf der Stelle dreht?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 2: zweites FTS bei (2, 5) m mit Gierwinkel 180 Grad
T_WF2 = transformationsmatrix(np.deg2rad(180), 2, 5)
umriss_W2 = transformiere(T_WF2, umriss)
print('Ecken des zweiten FTS in m:')
print(np.round(umriss_W2, 3))

zeichne_halle([umriss_W, umriss_W2], koordinatensysteme=[T_WF, T_WF2],
              umgebung=halle, beschriftungen=['FTS 1', 'FTS 2'],
              titel='Zwei Fahrzeuge in der Halle')
```
Bei $\psi = 180°$ zeigt die $x_F$-Achse in negative $x_W$-Richtung, das zweite
FTS fährt also nach links durch die Halle. Seine Ecke vorne links liegt bei
$(1.4, 4.6)$ und steht in der zweiten Zeile der Ausgabe. Vorne heißt hier
$0.6\,\text{m}$ in negativer $x_W$-Richtung, und links heißt $0.4\,\text{m}$ in
negativer $y_W$-Richtung. Fährt das erste FTS $1\,\text{m}$ geradeaus, wandert
seine Mitte um $1\,\text{m}$ in Richtung der ersten Spalte. Nur die dritte
Spalte ändert sich, sie wird zu $(4.866, 2.5)$. Die Richtungen der Achsen
bleiben gleich. Dreht sich das Fahrzeug auf der Stelle, ändern sich die ersten
beiden Spalten, und die dritte bleibt bei $(4, 2)$.
````

## Wo liegt das Hindernis in der Halle?

Der Laserscanner sitzt oben in der Mitte des Fahrzeugs, also im Ursprung des
Fahrzeugkoordinatensystems. Er sendet Laserstrahlen in verschiedene Richtungen
und misst für jeden Strahl, nach welcher Entfernung $r$ er auf einen
Gegenstand trifft. Die Richtung gibt er als Winkel $\varphi$ gegen die
$x_F$-Achse an, positiv nach links. Die Abbildung im ersten Abschnitt zeigt
eine solche Messung. Ein Messpunkt liegt also nicht als $(x_F, y_F)$ vor,
sondern als Entfernung und Winkel.

```{admonition} Aus der Mathematik: Polarkoordinaten
:class: note
Ein Punkt in der Ebene lässt sich durch seinen Abstand $r$ vom Ursprung und den
Winkel $\varphi$ gegen die $x$-Achse beschreiben. Diese **Polarkoordinaten**
rechnen wir mit

$$x = r \cos\varphi, \qquad y = r \sin\varphi$$

in kartesische Koordinaten um. Genauso haben wir in Kapitel 5.1 die Lage des
Greifers auf seinem Kreis berechnet. Der Punkt mit $r = 2$ und $\varphi = 90°$
liegt zum Beispiel bei $(0, 2)$.
```

Damit beantworten wir die Frage vom Anfang. Ein Hindernis $1.5\,\text{m}$
voraus hat $r = 1.5$ und $\varphi = 0$, im Fahrzeugkoordinatensystem also die
Koordinaten $(1.5, 0)$. In Hallenkoordinaten bringt es $\mathbf{T}_{WF}$.
`transformiere` erwartet ein Array mit einem Punkt pro Zeile. Den einzelnen
Punkt schreiben wir deshalb mit doppelten eckigen Klammern als Array der Form
`(1, 2)`.

```{code-cell} python
# Hindernis 1.5 m voraus: r = 1.5 m und phi = 0, also (1.5, 0) im Fahrzeug-KS
hindernis_F = np.array([[1.5, 0.0]])
hindernis_W = transformiere(T_WF, hindernis_F)
print('Hindernis in Hallenkoordinaten in m:', np.round(hindernis_W, 3))
```

Das Hindernis liegt bei $(5.299, 2.750)$. Das passt: $1.5\,\text{m}$ in
Fahrtrichtung unter $30°$ sind $1.5 \cdot \cos 30° = 1.299\,\text{m}$ in
$x_W$-Richtung und $1.5 \cdot \sin 30° = 0.75\,\text{m}$ in $y_W$-Richtung,
gerechnet ab der Fahrzeugmitte $(4, 2)$. Bei einer anderen Pose hätte dasselbe
Hindernis ganz andere Hallenkoordinaten. Die Messung allein verrät also nicht,
wo es in der Halle steht, dafür brauchen wir zusätzlich die Pose.

Ein Scanner misst nicht nur einen Punkt, sondern viele. Fünf seiner Strahlen
haben einen Gegenstand links vor dem Fahrzeug getroffen. Die Formeln für
Polarkoordinaten wenden wir wie in Kapitel 2.1 auf ganze Arrays auf einmal an.
Danach tragen wir $x_F$ und $y_F$ wie in der Funktion `transformiere` in die
Spalten eines Arrays der Form `(5, 2)` ein. Der Parameter `scanner` von
`zeichne_halle` zeichnet zusätzlich die Laserstrahlen ein. Er erwartet die
Lage des Scanners in Hallenkoordinaten, also die Fahrzeugmitte aus der dritten
Spalte von $\mathbf{T}_{WF}$.

```{code-cell} python
# Fünf Messungen des Scanners: Entfernung in m und Winkel in Grad
entfernungen = np.array([3.11, 2.83, 2.61, 2.79, 3.20])
winkel_grad = np.array([10, 15, 20, 25, 30])

# Polarkoordinaten in Fahrzeugkoordinaten umrechnen, alle Messungen auf einmal
winkel = np.deg2rad(winkel_grad)
messpunkte_F = np.zeros((len(entfernungen), 2))
messpunkte_F[:, 0] = entfernungen * np.cos(winkel)
messpunkte_F[:, 1] = entfernungen * np.sin(winkel)

# Von Fahrzeug- in Hallenkoordinaten umrechnen
messpunkte_W = transformiere(T_WF, messpunkte_F)
print('Messpunkte in Hallenkoordinaten in m:')
print(np.round(messpunkte_W, 3))

zeichne_halle(umriss_W, messpunkte=messpunkte_W, scanner=T_WF[0:2, 2],
              umgebung=halle, titel='Was der Scanner sieht')
```

Alle fünf Messpunkte liegen auf dem Rand der Palette, drei auf ihrer unteren
Kante bei $y = 4$ und zwei auf ihrer linken Kante bei $x = 5.6$. Dass einige
Werte wie $3.999$ statt $4$ lauten, liegt nicht an der Rechnung: Der Scanner
gibt die Entfernungen auf Zentimeter gerundet aus.

```{admonition} Mini-Übung (✩)
:class: tip
Vier weitere Strahlen treffen einen Gegenstand rechts vor dem Fahrzeug:

| Winkel $\varphi$ in Grad | Entfernung $r$ in m |
| --- | --- |
| $-66$ | $2.84$ |
| $-63$ | $2.74$ |
| $-60$ | $2.66$ |
| $-57$ | $2.86$ |

1. Beantworten Sie ohne Code: Welches Vorzeichen hat $y_F$ bei diesen
   Messpunkten, und warum?
2. Rechnen Sie die Messungen in Hallenkoordinaten um und zeichnen Sie sie
   zusammen mit der Hallenkarte. Welchen Gegenstand hat der Scanner getroffen?
3. Beantworten Sie ohne Code: Warum liegen die Messpunkte der Palette nur auf
   zwei ihrer vier Kanten?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 2: vier Messungen rechts vor dem Fahrzeug
entfernungen_2 = np.array([2.84, 2.74, 2.66, 2.86])
winkel_2 = np.deg2rad(np.array([-66, -63, -60, -57]))

messpunkte_F2 = np.zeros((len(entfernungen_2), 2))
messpunkte_F2[:, 0] = entfernungen_2 * np.cos(winkel_2)
messpunkte_F2[:, 1] = entfernungen_2 * np.sin(winkel_2)
messpunkte_W2 = transformiere(T_WF, messpunkte_F2)
print('Messpunkte in Hallenkoordinaten in m:')
print(np.round(messpunkte_W2, 3))

zeichne_halle(umriss_W, messpunkte=messpunkte_W2, scanner=T_WF[0:2, 2],
              umgebung=halle, titel='Ein Gegenstand rechts vor dem Fahrzeug')
```
Negative Winkel zeigen nach rechts. Der Sinus ist für Winkel zwischen $-90°$
und $0°$ negativ, deshalb sind alle $y_F$ negativ, und die Punkte liegen rechts
von der Fahrtrichtung. In Hallenkoordinaten liegen sie auf der linken und der
oberen Kante der Säule. Ein Laserstrahl endet an der ersten Kante, auf die er
trifft. Der Scanner sieht deshalb nur die Kanten, die ihm zugewandt sind. Bei
der Palette sind das die untere und die linke Kante, die beiden anderen liegen
im Schatten.
````

## Wo liegt die Übergabestation aus Sicht des Fahrzeugs?

Bisher haben wir Fahrzeugkoordinaten in Hallenkoordinaten umgerechnet. Für die
Fahrt zur Übergabestation braucht die Steuerung die umgekehrte Richtung. Die
Station steht laut Hallenkarte bei $(8, 2.5)$. Um sie anzufahren, muss das
Fahrzeug wissen, ob sie vor oder hinter ihm liegt und ob es nach links oder
rechts lenken muss. Genau das verraten die Fahrzeugkoordinaten der Station:
$x_F > 0$ heißt vorne, $y_F > 0$ heißt links.

*Welche Matrix rechnet von W nach F?* Sie muss die Wirkung von
$\mathbf{T}_{WF}$ genau aufheben. Eine solche Matrix haben wir in Kapitel 5.1
kennengelernt.

```{admonition} Aus der Mathematik: inverse Matrix
:class: note
Die **inverse Matrix** $\mathbf{A}^{-1}$ einer quadratischen Matrix
$\mathbf{A}$ hebt deren Wirkung auf, es gilt
$\mathbf{A}^{-1} \mathbf{A} = \mathbf{I}$. Sie existiert genau dann, wenn
$\det(\mathbf{A}) \neq 0$ gilt. Für Drehmatrizen haben wir in Kapitel 5.1
gesehen, dass die Inverse die Transponierte ist, $\mathbf{R}^{-1} = \mathbf{R}^\top$.
Unsere homogenen Transformationsmatrizen bestehen nur aus einer Drehung und
einer Verschiebung. Ihre Determinante ist wie bei der Drehmatrix immer 1,
deshalb lassen sie sich immer umkehren.
```

Die Umrechnung von W nach F leistet also

$$\mathbf{T}_{FW} = \mathbf{T}_{WF}^{-1}.$$

Auch hier lesen wir die Indizes von rechts nach links: $\mathbf{T}_{FW}$ nimmt
Hallenkoordinaten entgegen und liefert Fahrzeugkoordinaten. NumPy berechnet
die inverse Matrix mit `np.linalg.inv`. Zur Probe prüfen wir mit `np.allclose`
und `np.eye` aus Kapitel 5.1, ob $\mathbf{T}_{FW} \cdot \mathbf{T}_{WF}$ die
Einheitsmatrix ergibt.

```{code-cell} python
# Übergabestation in Hallenkoordinaten in m, als Array der Form (1, 2)
station = np.array([[8.0, 2.5]])

# Von Hallen- in Fahrzeugkoordinaten: mit der inversen Matrix
T_FW = np.linalg.inv(T_WF)
print('T_FW:')
print(np.round(T_FW, 4))
print('T_FW @ T_WF gleich Einheitsmatrix:', np.allclose(T_FW @ T_WF, np.eye(3)))

station_F = transformiere(T_FW, station)
print('Station in Fahrzeugkoordinaten in m:', np.round(station_F, 3))

zeichne_halle(umriss_W, koordinatensysteme=T_WF, ziele=station, umgebung=halle,
              titel='Wo liegt die Übergabestation?')
```

Die Station liegt $3.714\,\text{m}$ vor der Fahrzeugmitte. Ihre
$y_F$-Koordinate ist negativ, sie liegt also $1.567\,\text{m}$ rechts. Das
Fahrzeug muss nach rechts lenken.

In $\mathbf{T}_{FW}$ steht links oben $\mathbf{R}(-30°)$, die Rückdrehung aus
Kapitel 5.1. Die rechte Spalte ist aber nicht einfach $(-4, -2)$, sondern
$(-4.464, 0.268)$. *Warum?* $\mathbf{T}_{WF}$ dreht zuerst und verschiebt dann.
Die Umkehrung muss die beiden Schritte in umgekehrter Reihenfolge rückgängig
machen: zuerst zurückverschieben, dann zurückdrehen. Die Rückdrehung erfasst
dabei auch den Verschiebungsvektor. `np.linalg.inv` nimmt uns diese
Überlegung ab.

```{admonition} Mini-Übung (✩)
:class: tip
1. Beantworten Sie ohne Code: Das Fahrzeug steht bei $(4, 2)$ und zeigt mit
   $\psi = 90°$ in $y_W$-Richtung. Liegt die Station bei $(8, 2.5)$ vor oder
   hinter dem Fahrzeug, links oder rechts? Wie weit jeweils?
2. Prüfen Sie Ihre Antwort mit `np.linalg.inv` und `transformiere`.
3. Zurück zur Ausgangspose bei $(4, 2)$ mit $\psi = 30°$: Es liegt nahe, die
   Umkehrung von $\mathbf{T}_{WF}$ einfach als
   `transformationsmatrix(np.deg2rad(-30), -4, -2)` zu schreiben. Prüfen Sie
   mit `np.allclose`, ob das stimmt. Begründen Sie das Ergebnis ohne Code.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 2: Station aus Sicht des Fahrzeugs bei psi = 90 Grad
T_WF_90 = transformationsmatrix(np.deg2rad(90), 4, 2)
station_F_90 = transformiere(np.linalg.inv(T_WF_90), station)
print('Station in Fahrzeugkoordinaten in m:', np.round(station_F_90, 3))

# Teilaufgabe 3: naheliegende, aber falsche Umkehrung
T_naiv = transformationsmatrix(np.deg2rad(-30), -4, -2)
print('T_naiv @ T_WF gleich Einheitsmatrix:', np.allclose(T_naiv @ T_WF, np.eye(3)))
```
Bei $\psi = 90°$ zeigt das Fahrzeug in $y_W$-Richtung. Die Station liegt
$0.5\,\text{m}$ weiter in $y_W$-Richtung, also $0.5\,\text{m}$ vorne, und
$4\,\text{m}$ weiter in $x_W$-Richtung. Für ein Fahrzeug, das in
$y_W$-Richtung zeigt, ist die positive $x_W$-Richtung rechts. Die Station liegt
also $4\,\text{m}$ rechts, und die Rechnung liefert $(0.5, -4)$. Die
naheliegende Umkehrung ist falsch. `transformationsmatrix` dreht immer zuerst
und verschiebt danach. `T_naiv` macht also zuerst die Drehung rückgängig und
dann die Verschiebung. $\mathbf{T}_{WF}$ hat aber zuletzt verschoben, deshalb
muss die Umkehrung die Verschiebung als Erstes rückgängig machen. Wie in
Kapitel 5.3 spielt die Reihenfolge eine Rolle.
````

## Zusammenfassung und Ausblick

Die Halle und das Fahrzeug haben jeweils ein eigenes Koordinatensystem. Die
Pose $(x, y, \psi)$ legt fest, wo das Fahrzeugkoordinatensystem in der Halle
liegt und wohin es zeigt. Die homogene Transformationsmatrix
$\mathbf{T}_{WF}$ aus Kapitel 5.3 rechnet Fahrzeugkoordinaten in
Hallenkoordinaten um. In ihren Spalten stehen die Richtungen der
Fahrzeugachsen und die Lage der Fahrzeugmitte. Messungen des Laserscanners aus
Entfernung und Winkel rechnen wir mit $x = r\cos\varphi$ und
$y = r\sin\varphi$ in Fahrzeugkoordinaten um und bringen sie dann mit
$\mathbf{T}_{WF}$ in die Halle. Für die umgekehrte Richtung verwenden wir die
inverse Matrix $\mathbf{T}_{FW} = \mathbf{T}_{WF}^{-1}$, die NumPy mit
`np.linalg.inv` berechnet. Die Fahrzeugkoordinaten eines Ziels verraten, ob es
vor oder hinter dem Fahrzeug liegt und ob es links oder rechts liegt.

Im Projekt in Kapitel 6.2 werten wir einen ganzen Scan aus. Dort sitzt der
Scanner nicht in der Fahrzeugmitte, sondern vorne am Fahrzeug, und bringt
damit ein drittes Koordinatensystem mit. Wie beim Roboterarm in Kapitel 5.3
verketten wir deshalb zwei Transformationen. Außerdem prüfen wir, ob ein
Hindernis so nah vor dem Fahrzeug liegt, dass es anhalten muss.
