---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 5.1 Drehmatrizen am Roboterarm

In einer Montagezelle setzt ein **SCARA-Roboter** elektronische Bauteile auf
Platinen. SCARA ist die englische Abkürzung für einen Montagearm, der in der
Waagerechten nachgiebig und in der Senkrechten steif ist. Der Roboter besteht
aus einem Sockel und zwei waagerechten Armen, die sich in zwei Gelenken um
senkrechte Achsen drehen. Am Ende des zweiten Arms sitzt der Greifer, der sich
zusätzlich auf und ab bewegen kann. Die Steuerung stellt nur die Winkel der
Gelenke ein. Wo der Greifer dann steht, muss sie ausrechnen. *Wohin wandert der
Greifer, wenn sich das erste Gelenk um 30° dreht?*

In den Kapiteln 3 und 4 haben wir Matrizen genutzt, um Gleichungssysteme
aufzustellen und zu lösen. In diesem Kapitel lernen wir eine zweite Aufgabe
von Matrizen kennen: Eine **Drehmatrix** dreht Punkte um den Ursprung. Mit ihr
berechnen wir, wohin sich der Greifer und das Bauteil im Greifer bewegen, und
wir prüfen, warum der Arm dabei seine Länge behält.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können die **Drehmatrix** $\mathbf{R}(\varphi)$ aus den gedrehten
  Einheitsvektoren herleiten und einen Punkt damit um den Ursprung drehen.
* [ ] Sie können viele Punkte, die als Zeilen eines Arrays gespeichert sind,
  mit `(R @ punkte.T).T` in einem Schritt drehen.
* [ ] Sie wissen, was eine **orthogonale Matrix** ist, und können eine Drehung
  mit der transponierten Drehmatrix rückgängig machen.
```

## Wohin wandert der Greifer?

Für die Rechnung schauen wir von oben auf den Roboter. Beide Gelenke drehen
sich um senkrechte Achsen, deshalb bewegen sich die Arme nur in der
waagerechten Ebene. Die Hubbewegung des Greifers ändert nur die Höhe. Für die
Lage in der Ebene genügt also die Draufsicht.

```{figure} pics/chap05_scara.svg
:alt: Links ein SCARA-Roboter von der Seite mit Sockel, zwei waagerechten Armen, zwei senkrechten Drehachsen und einem Greifer mit Hubachse über einer Platine, rechts die Draufsicht mit den Armlängen l1 und l2 und den Gelenkwinkeln alpha und beta
:align: center
:label: fig_chap05_scara

Ein SCARA-Roboter von der Seite und seine Draufsicht mit den Armlängen und
den Gelenkwinkeln.
(Quelle: eigene Abbildung; Lizenz [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0))
```

Wir legen den Ursprung des Koordinatensystems in Gelenk 1. Arm 1 ist
$l_1 = 0.4\,\text{m}$ lang, Arm 2 ist $l_2 = 0.3\,\text{m}$ lang. Den Winkel
$\alpha$ von Gelenk 1 messen wir gegen die $x$-Achse, den Winkel $\beta$ von
Gelenk 2 gegen die Verlängerung von Arm 1. In diesem Kapitel halten wir
Gelenk 2 in gestreckter Stellung fest, also $\beta = 0$. Dann dreht sich der
ganze Arm wie ein starrer Stab um Gelenk 1. In der **Grundstellung**
$\alpha = 0$ zeigt der Arm entlang der $x$-Achse. Wie die Knotenkoordinaten in
Kapitel 4.1 speichern wir die Punkte des Arms als Zeilen eines 2D-Arrays.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Punkte des gestreckten Arms in Grundstellung in m, Zeile = [x, y]
arm = np.array([
    [0.0, 0.0],   # Gelenk 1: Drehachse am Sockel
    [0.4, 0.0],   # Gelenk 2: Ende von Arm 1
    [0.7, 0.0],   # Greifer: Ende von Arm 2
])

print(arm)
```

Zum Zeichnen verwenden wir wie in Part 4 eine vorgegebene Funktion. Wir führen
die Zelle einfach aus und müssen den Code nicht im Detail verstehen.

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

    if beschriftungen is not None or ziele is not None:
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
zeichne_roboter(arm, titel='SCARA-Roboter in Grundstellung')
```

Dreht sich Gelenk 1 um den Winkel $\alpha$, wandert der Greifer auf einem
Kreis mit dem Radius $0.7\,\text{m}$ um den Ursprung. Nach der Definition von
Kosinus und Sinus am Kreis landet er bei $(0.7\cos\alpha,\ 0.7\sin\alpha)$.
Diese Überlegung klappt, weil der Greifer in Grundstellung auf der $x$-Achse
liegt. Das Bauteil im Greifer hat aber Ecken neben der $x$-Achse. Wir brauchen
deshalb eine Formel, die jeden beliebigen Punkt $(x, y)$ dreht.

Dazu zerlegen wir den Punkt in seine Anteile entlang der Achsen. Er liegt $x$
Schritte entlang des **Einheitsvektors** $\vec{e}_x = (1, 0)^\top$ und $y$
Schritte entlang $\vec{e}_y = (0, 1)^\top$. Einheitsvektoren haben die Länge 1
und zeigen in Richtung der Koordinatenachsen. Bei einer Drehung um den Winkel
$\varphi$ drehen sich beide Einheitsvektoren mit.

```{figure} pics/chap05_einheitsvektoren.svg
:alt: Links wird der Einheitsvektor e_x um den Winkel phi gedreht und hat danach die Komponenten cos phi und sin phi, rechts wird e_y um phi gedreht und hat danach die Komponenten minus sin phi und cos phi
:align: center
:label: fig_chap05_einheitsvektoren

Die beiden Einheitsvektoren vor und nach einer Drehung um den Winkel
$\varphi$, mit den Komponenten der gedrehten Vektoren.
(Quelle: eigene Abbildung; Lizenz [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0))
```

Die Abbildung zeigt: $\vec{e}_x$ geht über in $(\cos\varphi, \sin\varphi)^\top$
und $\vec{e}_y$ in $(-\sin\varphi, \cos\varphi)^\top$. Der gedrehte Punkt
besteht aus denselben Anteilen, jetzt aber entlang der gedrehten
Einheitsvektoren:

$$\vec{p}^{\,\prime} = x \begin{pmatrix} \cos\varphi \\ \sin\varphi \end{pmatrix}
+ y \begin{pmatrix} -\sin\varphi \\ \cos\varphi \end{pmatrix}
= \begin{pmatrix} x\cos\varphi - y\sin\varphi \\ x\sin\varphi + y\cos\varphi \end{pmatrix}.$$

Rechts steht genau das, was beim Produkt einer Matrix mit dem Vektor
$(x, y)^\top$ herauskommt. Dieses Produkt haben wir in Kapitel 3.1 Zeile für
Zeile ausgerechnet:

$$\vec{p}^{\,\prime} =
\underbrace{\begin{pmatrix} \cos\varphi & -\sin\varphi \\ \sin\varphi & \cos\varphi \end{pmatrix}}_{\mathbf{R}(\varphi)}
\begin{pmatrix} x \\ y \end{pmatrix}.$$

Die Matrix $\mathbf{R}(\varphi)$ heißt **Drehmatrix**. In ihren Spalten stehen
die gedrehten Einheitsvektoren. Ein positiver Winkel dreht gegen den
Uhrzeigersinn, ein negativer im Uhrzeigersinn.

Wir prüfen die Formel an einem Fall, den wir sofort im Kopf überblicken. Bei
$\varphi = 90°$ gilt $\cos 90° = 0$ und $\sin 90° = 1$. Der Greifer bei
$(0.7, 0)$ landet dann bei

$$\mathbf{R}(90°) \begin{pmatrix} 0.7 \\ 0 \end{pmatrix}
= \begin{pmatrix} 0 & -1 \\ 1 & 0 \end{pmatrix} \begin{pmatrix} 0.7 \\ 0 \end{pmatrix}
= \begin{pmatrix} 0 \cdot 0.7 + (-1) \cdot 0 \\ 1 \cdot 0.7 + 0 \cdot 0 \end{pmatrix}
= \begin{pmatrix} 0 \\ 0.7 \end{pmatrix}.$$

Nach einer Vierteldrehung zeigt der Arm also entlang der $y$-Achse, wie
erwartet.

```{admonition} Drehungen im Raum
:class: note
Die Gelenke des SCARA-Roboters drehen sich um senkrechte Achsen, also um die
$z$-Achse. Im Raum lautet die Drehmatrix dafür

$$\mathbf{R}_z(\varphi) = \begin{pmatrix} \cos\varphi & -\sin\varphi & 0 \\
\sin\varphi & \cos\varphi & 0 \\ 0 & 0 & 1 \end{pmatrix}.$$

Der linke obere 2×2-Block ist unsere Drehmatrix, die $z$-Koordinate bleibt
unverändert. Das bestätigt, dass für den SCARA-Roboter die Draufsicht genügt.
Knickarmroboter, die in der Automobilfertigung schweißen und lackieren, drehen
sich zusätzlich um waagerechte Achsen. Für sie gibt es entsprechende
Drehmatrizen um die $x$- und die $y$-Achse.
```

In Python schreiben wir die Drehmatrix als Funktion `drehmatrix(winkel)`, die
ein 2×2-Array zurückgibt. `np.cos` und `np.sin` erwarten den Winkel in
**Radiant**, nicht in Grad. Die Umrechnung übernimmt `np.deg2rad`. Den Punkt
drehen wir mit dem Operator `@` für die Matrixmultiplikation aus Kapitel 3.1.

```{code-cell} python
def drehmatrix(winkel):
    """Drehmatrix für eine Drehung um den Ursprung, winkel in Radiant."""
    return np.array([
        [np.cos(winkel), -np.sin(winkel)],
        [np.sin(winkel),  np.cos(winkel)],
    ])

# Gelenk 1 dreht um 30 Grad, np.deg2rad rechnet Grad in Radiant um
alpha = np.deg2rad(30)
R = drehmatrix(alpha)

# Greifer drehen: Drehmatrix mal Punkt
greifer = arm[2]
greifer_neu = R @ greifer

print('Drehmatrix R(30°):')
print(np.round(R, 4))
print('Greifer nach der Drehung in m:', np.round(greifer_neu, 4))
```

Der Greifer steht nach der Drehung bei $(0.606, 0.350)$. Das passt zu unserer
Überlegung am Kreis: $0.7\cos 30° = 0.606$ und $0.7\sin 30° = 0.350$. In der
ersten Spalte der Drehmatrix steht $(0.866, 0.5)$, also der gedrehte
Einheitsvektor $\vec{e}_x$.

```{admonition} Winkel in Radiant
:class: warning
Ein häufiger Fehler ist `np.cos(30)`. NumPy rechnet dann mit 30 Radiant, das
sind rund 1719°, und liefert 0.154 statt 0.866. Python meldet dabei keinen
Fehler. Grad rechnen wir deshalb immer zuerst mit `np.deg2rad` um. Ein voller
Kreis entspricht $2\pi \approx 6.28$ Radiant.
```

```{admonition} Mini-Übung (✩)
:class: tip
1. Beantworten Sie ohne Code: Wo steht der Greifer bei $\alpha = 180°$, wo bei
   $\alpha = -90°$?
2. Berechnen Sie mit `drehmatrix` die Position des Greifers für
   $\alpha = 135°$.
3. Beantworten Sie ohne Code: Warum haben die beiden Koordinaten aus
   Teilaufgabe 2 denselben Betrag?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 2: Greiferposition bei 135 Grad
R_135 = drehmatrix(np.deg2rad(135))
greifer_135 = R_135 @ arm[2]
print('Greifer bei 135° in m:', np.round(greifer_135, 4))
```
Bei $\alpha = 180°$ zeigt der Arm entgegen der $x$-Achse, der Greifer steht bei
$(-0.7, 0)$. Bei $\alpha = -90°$ dreht der Arm im Uhrzeigersinn, der Greifer
steht bei $(0, -0.7)$. Bei $\alpha = 135°$ steht er bei $(-0.495, 0.495)$. Der
Winkel $135°$ liegt genau in der Mitte zwischen $90°$ und $180°$, also auf der
Winkelhalbierenden zwischen negativer $x$-Achse und positiver $y$-Achse. Dort
sind Kosinus und Sinus dem Betrag nach gleich, beide etwa $0.707$.
````

## Den ganzen Arm auf einmal drehen

Bisher haben wir nur den Greifer gedreht. Die Steuerung braucht aber mehr
Punkte. Aus der Lage von Gelenk 2 erkennt sie, ob der Arm irgendwo anstößt.
Aus den Ecken des Bauteils erkennt sie, ob das Bauteil passend auf der Platine
landet. Das Bauteil ist $8\,\text{cm}$ lang und $4\,\text{cm}$ breit. Es sitzt
mittig unter dem Greifer und liegt in Grundstellung längs zum Arm.

*Müssen wir jeden Punkt einzeln mit $\mathbf{R}$ multiplizieren?* Nein, das
geht in einem Schritt. Das Produkt $\mathbf{R} \cdot \vec{p}$ erwartet den
Punkt als Spalte mit zwei Einträgen. Unsere Arrays speichern dagegen jeden
Punkt als Zeile und haben die Form `(n, 2)`. Die Transponierte `.T` aus
Kapitel 4.2 vertauscht Zeilen und Spalten. `bauteil.T` hat die Form `(2, 4)`,
jede Spalte ist eine Ecke. `R @ bauteil.T` multipliziert $\mathbf{R}$ mit jeder
Spalte auf einmal und liefert wieder die Form `(2, 4)`. Ein abschließendes `.T`
bringt das Ergebnis zurück in die Form mit einem Punkt pro Zeile.

```{code-cell} python
# Ecken des Bauteils im Greifer in m, Grundstellung, Zeile = [x, y]
bauteil = np.array([
    [0.66, -0.02],
    [0.74, -0.02],
    [0.74,  0.02],
    [0.66,  0.02],
])

# Formen vor und nach dem Transponieren
print('Form von bauteil:      ', bauteil.shape)
print('Form von bauteil.T:    ', bauteil.T.shape)
print('Form von R @ bauteil.T:', (R @ bauteil.T).shape)
```

Die Drehmatrix `R` ist noch die Drehung um 30° aus dem letzten Abschnitt. Wir
drehen damit den Arm und das Bauteil und zeichnen beide Stellungen.

```{code-cell} python
# Alle Punkte auf einmal drehen: transponieren, multiplizieren, zurücktransponieren
arm_gedreht = (R @ arm.T).T
bauteil_gedreht = (R @ bauteil.T).T

print('Arm nach der Drehung um 30° in m:')
print(np.round(arm_gedreht, 4))

zeichne_roboter([arm, arm_gedreht], bauteile=[bauteil, bauteil_gedreht],
                beschriftungen=['Grundstellung', 'α = 30°'],
                titel='Gelenk 1 dreht um 30°')
```

Gelenk 1 bleibt im Ursprung stehen, denn eine Drehung um den Ursprung bewegt
den Ursprung nicht. Gelenk 2 landet bei $(0.346, 0.200)$, der Greifer wie
vorher bei $(0.606, 0.350)$. Beide wandern auf Kreisen um Gelenk 1. Das Bauteil
wandert aber nicht nur mit, es dreht sich auch: Seine lange Kante liegt jetzt
um 30° schräg zur $x$-Achse. Eine Drehung ändert also neben der Position auch
die **Orientierung**, also die Ausrichtung eines Körpers.

```{admonition} Mini-Übung (✩)
:class: tip
1. Drehen Sie Arm und Bauteil um $\alpha = -60°$ und zeichnen Sie
   Grundstellung und gedrehte Stellung mit `zeichne_roboter`.
2. Beantworten Sie ohne Code: Welche Form haben `arm.T` und `R @ arm.T`?
3. Beantworten Sie ohne Code: Das Bauteil soll längs zur $x$-Achse auf der
   Platine landen. Was passiert, wenn der Roboter es bei $\alpha = -60°$
   absetzt? Was müsste der Roboter zusätzlich können?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 1: Arm und Bauteil um -60 Grad drehen und zeichnen
R_minus60 = drehmatrix(np.deg2rad(-60))
arm_minus60 = (R_minus60 @ arm.T).T
bauteil_minus60 = (R_minus60 @ bauteil.T).T
print(np.round(arm_minus60, 4))

zeichne_roboter([arm, arm_minus60], bauteile=[bauteil, bauteil_minus60],
                beschriftungen=['Grundstellung', 'α = -60°'],
                titel='Gelenk 1 dreht um -60°')
```
Der Arm zeigt nach rechts unten, der Greifer steht bei $(0.350, -0.606)$.
`arm` hat drei Punkte und die Form `(3, 2)`, deshalb haben `arm.T` und
`R @ arm.T` die Form `(2, 3)`. Das Bauteil ist um $60°$ im Uhrzeigersinn
gedreht und läge schräg auf der Platine. Damit es längs zur $x$-Achse landet,
muss es vor dem Absetzen um $60°$ zurückgedreht werden. Echte SCARA-Roboter
haben dafür eine weitere Drehachse direkt am Greifer.
````

## Warum bleibt der Arm gleich lang?

Ein starrer Arm darf sich beim Drehen weder strecken noch stauchen. Wir prüfen
zuerst, ob unsere Rechnung das einhält. Den Abstand des Greifers von Gelenk 1
berechnen wir wie die Stablänge in Kapitel 4.1 mit Pythagoras,
$\sqrt{x^2 + y^2}$.

```{code-cell} python
# Drehung um 30 Grad wie im letzten Abschnitt
R = drehmatrix(np.deg2rad(30))
arm_gedreht = (R @ arm.T).T

# Abstand des Greifers von Gelenk 1 mit Pythagoras, vor und nach der Drehung
abstand_vorher = np.sqrt(arm[2, 0]**2 + arm[2, 1]**2)
abstand_nachher = np.sqrt(arm_gedreht[2, 0]**2 + arm_gedreht[2, 1]**2)

print(f'Abstand vorher:  {abstand_vorher:.4f} m')
print(f'Abstand nachher: {abstand_nachher:.4f} m')
```

Der Abstand beträgt vor und nach der Drehung $0.7\,\text{m}$. *Liegt das an
den gewählten Zahlen, oder gilt es für jeden Winkel?* Den Grund finden wir in
der Drehmatrix selbst. Wir multiplizieren ihre Transponierte mit ihr:

$$\mathbf{R}^\top \mathbf{R}
= \begin{pmatrix} \cos\varphi & \sin\varphi \\ -\sin\varphi & \cos\varphi \end{pmatrix}
  \begin{pmatrix} \cos\varphi & -\sin\varphi \\ \sin\varphi & \cos\varphi \end{pmatrix}
= \begin{pmatrix} \cos^2\varphi + \sin^2\varphi & 0 \\ 0 & \sin^2\varphi + \cos^2\varphi \end{pmatrix}
= \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix}.$$

Neben der Diagonalen heben sich die Terme weg, oben rechts zum Beispiel
$\cos\varphi \cdot (-\sin\varphi) + \sin\varphi \cdot \cos\varphi = 0$. Auf der
Diagonalen steht $\cos^2\varphi + \sin^2\varphi = 1$, das ist der Satz des
Pythagoras am Einheitskreis. Das Ergebnis ist die **Einheitsmatrix**
$\mathbf{I}$ mit Einsen auf der Diagonalen und Nullen sonst. Eine Matrix mit
$\mathbf{R}^\top \mathbf{R} = \mathbf{I}$ heißt **orthogonale Matrix**.
Orthogonale Matrizen ändern weder Längen noch Winkel. Deshalb behalten Arm und
Bauteil bei jeder Drehung ihre Form.

In Python erzeugt `np.eye(2)` die 2×2-Einheitsmatrix. Wir vergleichen mit
`np.allclose` aus Kapitel 3.1, weil Kosinus und Sinus kleine Rundungsfehler
haben. Zusätzlich berechnen wir die Determinante mit `np.linalg.det`.

```{code-cell} python
# R transponiert mal R soll die Einheitsmatrix ergeben
produkt = R.T @ R
print(np.round(produkt, 4))
print('R^T R gleich Einheitsmatrix:', np.allclose(produkt, np.eye(2)))

# Determinante der Drehmatrix
print(f'Determinante von R: {np.linalg.det(R):.4f}')
```

Die Rechnung bestätigt $\mathbf{R}^\top \mathbf{R} = \mathbf{I}$. Die
Determinante ist 1. Sie ist also ungleich null, und die Drehung lässt sich nach
Kapitel 3.1 eindeutig umkehren. Wie das geht, verrät uns
$\mathbf{R}^\top \mathbf{R} = \mathbf{I}$ direkt: Die Transponierte macht die
Drehung rückgängig. Eine Matrix, die die Wirkung einer anderen Matrix
aufhebt, heißt **inverse Matrix** $\mathbf{R}^{-1}$. Bei Drehmatrizen ist die
Inverse einfach die Transponierte, $\mathbf{R}^{-1} = \mathbf{R}^\top$. Für den
Roboter heißt das: Multiplizieren wir den gedrehten Arm mit
$\mathbf{R}^\top$, fährt er zurück in die Grundstellung.

```{code-cell} python
# Zurück in die Grundstellung: mit der transponierten Drehmatrix drehen
arm_zurueck = (R.T @ arm_gedreht.T).T

print(np.round(arm_zurueck, 4))
print('Grundstellung erreicht:', np.allclose(arm_zurueck, arm))
```

Der Arm steht wieder in der Grundstellung. In der Ausgabe erscheint dabei
`-0.`. Das sind winzige negative Rundungsreste in der Größenordnung $10^{-18}$,
die `np.round` als $-0$ anzeigt. Auch hier vergleichen wir deshalb mit
`np.allclose` und nicht mit `==`.

```{admonition} Mini-Übung (✩)
:class: tip
1. Berechnen Sie die Länge der langen Bauteilkante, also den Abstand der Ecken
   0 und 1, vor und nach der Drehung um $30°$.
2. Prüfen Sie mit `np.allclose`, dass $\mathbf{R}^\top \mathbf{R}$ auch für
   $\varphi = 135°$ die Einheitsmatrix ergibt.
3. Beantworten Sie ohne Code: $\mathbf{R}(30°)^\top$ dreht den Arm zurück in
   die Grundstellung. Welcher Drehung entspricht $\mathbf{R}(30°)^\top$? Tipp:
   Es gilt $\cos(-\varphi) = \cos\varphi$ und $\sin(-\varphi) = -\sin\varphi$.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 1: lange Bauteilkante vor und nach der Drehung um 30 Grad
bauteil_gedreht = (R @ bauteil.T).T
kante_vorher = bauteil[1] - bauteil[0]
kante_nachher = bauteil_gedreht[1] - bauteil_gedreht[0]
laenge_vorher = np.sqrt(kante_vorher[0]**2 + kante_vorher[1]**2)
laenge_nachher = np.sqrt(kante_nachher[0]**2 + kante_nachher[1]**2)
print(f'Kante vorher:  {laenge_vorher:.4f} m')
print(f'Kante nachher: {laenge_nachher:.4f} m')

# Teilaufgabe 2: Orthogonalität für 135 Grad
R_135 = drehmatrix(np.deg2rad(135))
print('R^T R gleich Einheitsmatrix:', np.allclose(R_135.T @ R_135, np.eye(2)))
```
Die Kante ist vor und nach der Drehung $0.08\,\text{m}$ lang. Auch für
$135°$ ergibt $\mathbf{R}^\top \mathbf{R}$ die Einheitsmatrix, denn die
Rechnung oben gilt für jeden Winkel. Zu Teilaufgabe 3: Die Transponierte
lautet

$$\mathbf{R}(30°)^\top = \begin{pmatrix} \cos 30° & \sin 30° \\ -\sin 30° & \cos 30° \end{pmatrix}
= \begin{pmatrix} \cos(-30°) & -\sin(-30°) \\ \sin(-30°) & \cos(-30°) \end{pmatrix}
= \mathbf{R}(-30°).$$

Die Rückdrehung ist also eine Drehung um $30°$ im Uhrzeigersinn. Allgemein gilt
$\mathbf{R}(\varphi)^\top = \mathbf{R}(-\varphi)$.
````

## Zusammenfassung und Ausblick

Eine Drehmatrix $\mathbf{R}(\varphi)$ dreht Punkte um den Ursprung. In ihren
Spalten stehen die gedrehten Einheitsvektoren, und ein positiver Winkel dreht
gegen den Uhrzeigersinn. Speichern wir viele Punkte als Zeilen eines Arrays,
drehen wir sie mit `(R @ punkte.T).T` alle in einem Schritt. Weil
$\mathbf{R}^\top \mathbf{R} = \mathbf{I}$ gilt, ist die Drehmatrix orthogonal:
Sie erhält Längen und Winkel, und die Transponierte macht die Drehung
rückgängig. Für unseren SCARA-Roboter heißt das: Aus dem Winkel von Gelenk 1
berechnen wir die Lage von Arm, Greifer und Bauteil.

Im nächsten Kapitel 5.2 setzen wir die Drehmatrix in einem Projekt ein: Ein
Schwenkarm legt Teile von einem Förderband in eine Kiste. Bisher bleibt bei
unserem SCARA-Roboter Gelenk 2 gestreckt. Dreht es sich auch, sitzt seine
Drehachse nicht mehr im Ursprung, sondern am Ende von Arm 1. Eine Drehmatrix
allein reicht dann nicht mehr. Wie wir Drehen und Verschieben in einer
einzigen Matrix zusammenfassen, zeigt Kapitel 5.3 mit den **homogenen
Koordinaten**.
