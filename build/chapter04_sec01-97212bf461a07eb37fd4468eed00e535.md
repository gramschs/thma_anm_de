---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.1 Vom Fachwerk zur Steifigkeitsmatrix

In Kapitel 3.2 haben wir die Auflagerkräfte eines Trägers berechnet. Die drei
Gleichgewichtsbedingungen haben wir dafür von Hand aufgestellt. Das klappt
bei einem Träger mit zwei Lagern gut. Bei einem Kran, einer Brücke oder einem
Dachstuhl mit Dutzenden Stäben wird das schnell unübersichtlich. *Wie bringen
wir den Rechner dazu, das Gleichungssystem für ein ganzes Tragwerk selbst
aufzustellen?*

Wir betrachten dafür **ideale ebene Fachwerke**. Sie bestehen aus geraden
Stäben, die an ihren Enden gelenkig verbunden sind. Die Verbindungsstellen
heißen **Knoten**. Äußere Kräfte greifen nur an den Knoten an. Deshalb werden
die Stäbe nur längs ihrer Achse auf Zug oder Druck beansprucht, nicht auf
Biegung. In diesem Kapitel beschreiben wir ein Fachwerk im Rechner und bauen
daraus Schritt für Schritt die Matrix des Gleichungssystems auf.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können ein ebenes Fachwerk durch Knotenkoordinaten, Lagerknoten,
  eine **Stabliste** und einen **Kraftvektor** im Rechner beschreiben.
* [ ] Sie können die **Stabsteifigkeit** $k = EA/L$ berechnen und erklären,
  warum ein Stab nur in Richtung seiner Achse Kräfte überträgt.
* [ ] Sie können die **Steifigkeitsmatrix** eines Fachwerks mit einer Funktion
  zusammensetzen und begründen, warum sie ohne Lager singulär ist.
```

## Wie beschreiben wir ein Fachwerk im Rechner?

Unser Beispiel ist ein einfacher Kranausleger. Zwei Stahlstäbe treffen sich
oben in Knoten 1, an dem eine Last von $5000\,\text{N}$ hängt. Die beiden
unteren Knoten 0 und 2 sind fest gelagert.

```{figure} pics/chap04_lastkran.svg
:alt: Kranausleger aus zwei Stäben mit drei Knoten, zwei Lagern und einer Last am oberen Knoten
:align: center

Der Kranausleger mit drei Knoten, zwei Stäben und einer Last an Knoten 1.
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

Für die Rechnung brauchen wir vier Angaben: Wo liegen die Knoten? Welche
Knoten sind gelagert? Welche Knoten sind durch einen Stab verbunden? Welche
Kräfte wirken? Das übersetzen wir direkt in Python.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Knotenkoordinaten in m: Zeile n enthält [x, y] von Knoten n
knoten_pos = np.array([
    [0.0, 0.0],   # Knoten 0: linkes Lager
    [1.0, 1.0],   # Knoten 1: Spitze, hier hängt die Last
    [2.0, 0.0],   # Knoten 2: rechtes Lager
])
anzahl_knoten = len(knoten_pos)

# Lagerknoten: diese Knoten können sich nicht bewegen
lager_indizes = [0, 2]

# Stabliste: Zeile s enthält die Nummern der beiden Knoten von Stab s
staebe = np.array([
    [0, 1],   # Stab 0 verbindet Knoten 0 und Knoten 1
    [1, 2],   # Stab 1 verbindet Knoten 1 und Knoten 2
])

# Kraftvektor in N: [Fx_0, Fy_0, Fx_1, Fy_1, Fx_2, Fy_2]
kraft_vektor = np.zeros(2 * anzahl_knoten)
kraft_vektor[3] = -5000.0   # Fy an Knoten 1: 5000 N nach unten

print('Anzahl Knoten:', anzahl_knoten)
print('Anzahl Stäbe: ', len(staebe))
print('Kraftvektor:  ', kraft_vektor)
```

Die Knotenkoordinaten stehen in einem 2D-Array mit einer Zeile pro Knoten.
Mit `knoten_pos[1]` erhalten wir die ganze Zeile, also beide Koordinaten von
Knoten 1. Die **Stabliste** `staebe` funktioniert genauso: Jede Zeile ist ein
Stab und enthält die Nummern der beiden Knoten, die er verbindet.

Beim Kraftvektor müssen wir genauer hinschauen. Jeder Knoten kann sich in
$x$- und in $y$-Richtung bewegen. Diese beiden Bewegungsmöglichkeiten heißen
**Freiheitsgrade**. Drei Knoten haben also sechs Freiheitsgrade, und der
Kraftvektor hat sechs Einträge. Sie sind knotenweise sortiert: erst $x$ und
$y$ von Knoten 0, dann von Knoten 1 und so weiter. Allgemein gilt:

- Index $2n$ gehört zur $x$-Richtung von Knoten $n$,
- Index $2n+1$ gehört zur $y$-Richtung von Knoten $n$.

Die Last an Knoten 1 in $y$-Richtung steht deshalb an Index $2 \cdot 1 + 1 = 3$.
Das Minuszeichen bedeutet: Die Kraft zeigt nach unten, also gegen die
$y$-Achse.

Bevor wir rechnen, zeichnen wir das Fachwerk. So fallen Tippfehler in den
Koordinaten oder in der Stabliste sofort auf. Die Zeichenfunktion ist
vorgegeben. Wir führen die Zelle einfach aus und müssen den Code nicht im
Detail verstehen.

```{code-cell} python
:tags: [hide-input]
# Vorgegebene Zeichenfunktion: einfach ausführen, sie muss nicht verstanden werden.
def zeichne_fachwerk(knoten_pos, staebe, lager_indizes, loslager_indizes=None,
                     kraft_vektor=None, verschiebung=None, skalierung=1.0,
                     stabkraefte=None, titel=''):
    """Zeichnet ein ebenes Fachwerk.

    knoten_pos: Knotenkoordinaten in m, Zeile n = [x_n, y_n]
    staebe: Stabliste, Zeile s = [i, j]
    lager_indizes: Liste der Knoten mit Festlager
    loslager_indizes: Liste der Knoten mit Loslager (optional)
    kraft_vektor: äußere Knotenkräfte in N, als Pfeile (optional)
    verschiebung: Verschiebungsvektor in m, zeichnet die verformte Lage (optional)
    skalierung: Überhöhungsfaktor für die Verschiebungen
    stabkraefte: Stabkräfte in N, blau = Zug, rot = Druck, grau = kraftlos (optional)
    titel: Diagrammtitel
    """
    blau, rot, orange, grau = '#005A94', '#E60000', '#E87846', '#484949'
    anzahl_knoten = len(knoten_pos)
    spannweite = np.max(knoten_pos) - np.min(knoten_pos)
    fig, ax = plt.subplots(figsize=(8, 4.5))

    # Knotenpositionen: Ausgangslage oder überhöht verformte Lage
    pos = knoten_pos.copy()
    if verschiebung is not None:
        pos = knoten_pos + skalierung * verschiebung.reshape(anzahl_knoten, 2)
        for i, j in staebe:
            ax.plot(knoten_pos[[i, j], 0], knoten_pos[[i, j], 1],
                    color=grau, linestyle='--', linewidth=1)

    # Stäbe, bei Stabkräften eingefärbt und beschriftet
    for s in range(len(staebe)):
        i, j = staebe[s]
        farbe = blau
        if stabkraefte is not None:
            farbe = blau if stabkraefte[s] >= 0 else rot
            if abs(stabkraefte[s]) < 1e-6 * np.max(np.abs(stabkraefte)):
                farbe = '#A6A6A6'   # hellgrau: Stab ohne Kraft (Rundungsfehler ignorieren)
            mitte = 0.5 * (pos[i] + pos[j])
            ax.text(mitte[0], mitte[1], f' {stabkraefte[s] / 1000:.2f} kN',
                    color=farbe, fontsize=9)
        ax.plot(pos[[i, j], 0], pos[[i, j], 1], color=farbe, linewidth=3)

    # Lager als Dreiecke unter den Knoten, Loslager mit zusätzlichem Strich
    if loslager_indizes is None:
        loslager_indizes = []
    h = 0.06 * spannweite
    for n in list(lager_indizes) + list(loslager_indizes):
        x, y = knoten_pos[n]
        ax.fill([x, x - h, x + h], [y, y - 1.5 * h, y - 1.5 * h],
                color='#CCDEE9', edgecolor=grau, zorder=2)
        if n in loslager_indizes:
            ax.plot([x - h, x + h], [y - 2 * h, y - 2 * h], color=grau, linewidth=2)

    # Knoten mit Nummern
    ax.scatter(pos[:, 0], pos[:, 1], color=orange, s=60, zorder=3)
    for n in range(anzahl_knoten):
        ax.text(pos[n, 0], pos[n, 1], f'  K{n}', color=blau,
                fontsize=10, va='bottom')

    # äußere Kräfte als Pfeile, die auf den Knoten zeigen
    if kraft_vektor is not None:
        kraefte = kraft_vektor.reshape(anzahl_knoten, 2)
        for n in range(anzahl_knoten):
            betrag = np.sqrt(kraefte[n, 0]**2 + kraefte[n, 1]**2)
            if betrag > 0:
                richtung = kraefte[n] / betrag
                start = pos[n] - 0.15 * spannweite * richtung
                ax.annotate('', xy=pos[n], xytext=start,
                            arrowprops=dict(color=grau, width=2, headwidth=8))
                ax.text(start[0], start[1], f' {betrag:.0f} N', color=grau,
                        fontsize=9)
                ax.plot(start[0], start[1], alpha=0)   # Pfeil im Bildbereich halten

    if stabkraefte is not None:
        ax.plot([], [], color=blau, linewidth=3, label='Zug')
        ax.plot([], [], color=rot, linewidth=3, label='Druck')
        ax.legend(loc='upper right')

    ax.set_title(titel)
    ax.set_xlabel('x in m')
    ax.set_ylabel('y in m')
    ax.set_aspect('equal')
    ax.margins(0.15)
    ax.grid(True)
    plt.show()
```

```{code-cell} python
zeichne_fachwerk(knoten_pos, staebe, lager_indizes, kraft_vektor=kraft_vektor,
                 titel='Kranausleger')
```

Der Plot stimmt mit der Skizze überein: zwei Stäbe, zwei Lager, eine Last
nach unten an Knoten 1. Wir vereinfachen dabei wie folgt: Beide Lager sind
**Festlager** wie das Lager A in Kapitel 3.2, sie halten den Knoten in $x$-
und in $y$-Richtung fest.

```{admonition} Mini-Übung (✩)
:class: tip
1. Beantworten Sie ohne Code: Was liefert `knoten_pos[2]`, und was liefert
   `staebe[1]`? Was bedeuten die Werte am Fachwerk?
2. Beantworten Sie ohne Code: An welchem Index von `kraft_vektor` steht die
   Kraft in $x$-Richtung an Knoten 2?
3. Legen Sie einen neuen Kraftvektor `kraft_vektor_neu` an, bei dem an Knoten 1
   zusätzlich eine Kraft von $1000\,\text{N}$ nach rechts wirkt. Zeichnen Sie
   das Fachwerk mit diesem Kraftvektor.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 3: Seitenkraft zusätzlich zur Last an Knoten 1
kraft_vektor_neu = np.zeros(2 * anzahl_knoten)
kraft_vektor_neu[2] = 1000.0    # Fx an Knoten 1: Index 2 * 1 = 2
kraft_vektor_neu[3] = -5000.0   # Fy an Knoten 1: Index 2 * 1 + 1 = 3
print('Neuer Kraftvektor:', kraft_vektor_neu)

zeichne_fachwerk(knoten_pos, staebe, lager_indizes,
                 kraft_vektor=kraft_vektor_neu, titel='Kranausleger mit Seitenkraft')
```
`knoten_pos[2]` liefert `[2. 0.]`, also die Koordinaten von Knoten 2:
$x = 2\,\text{m}$, $y = 0\,\text{m}$. `staebe[1]` liefert `[1 2]`: Stab 1
verbindet Knoten 1 mit Knoten 2. Die $x$-Kraft an Knoten 2 steht an Index
$2 \cdot 2 = 4$. Die Seitenkraft an Knoten 1 gehört an Index $2 \cdot 1 = 2$.
Im Plot zeigt der Kraftpfeil jetzt schräg nach rechts unten, denn er stellt
die Summe beider Kräfte dar. Die Beschriftung nennt deshalb den Betrag
$\sqrt{1000^2 + 5000^2}\,\text{N} \approx 5099\,\text{N}$.
````

## Ein Stab wirkt wie eine Feder

Jetzt brauchen wir das Material. Die Stäbe sind aus Stahl und haben einen
runden Querschnitt mit $1\,\text{cm}$ Durchmesser. Wir greifen uns Stab 0
heraus und berechnen seine Länge und seine Steifigkeit.

```{code-cell} python
# Material und Querschnitt, für alle Stäbe gleich
elastizitaetsmodul = 2.1e11                      # Stahl in N/m²
durchmesser = 0.01                               # in m
querschnitt = np.pi * durchmesser**2 / 4         # Kreisfläche in m²

# Stab 0: Knotennummern aus der Stabliste holen
i, j = staebe[0]

# Differenzvektor von Knoten i nach Knoten j und Stablänge (Pythagoras)
differenz = knoten_pos[j] - knoten_pos[i]
stablaenge = np.sqrt(differenz[0]**2 + differenz[1]**2)

# Stabsteifigkeit k = E * A / L
k = elastizitaetsmodul * querschnitt / stablaenge

print(f'Stab 0 verbindet Knoten {i} und {j}')
print(f'Differenzvektor: {differenz} m')
print(f'Stablänge:       {stablaenge:.4f} m')
print(f'Steifigkeit k:   {k:.4e} N/m = {k * 1e-6:.2f} kN/mm')
```

Die Zeile `i, j = staebe[0]` entpackt die beiden Knotennummern in zwei
Variablen, so wie wir in Kapitel 3.3 die Lösung in `T_AB, T_BC, Q` entpackt
haben.

Warum heißt $k$ Steifigkeit? Ein Stab verhält sich unter Zug oder Druck wie
eine Feder. Aus dem Hookeschen Gesetz $\sigma = E\,\varepsilon$ mit der
Spannung $\sigma = F/A$ und der Dehnung $\varepsilon = \Delta L/L$ folgt

$$F = \frac{E\,A}{L} \cdot \Delta L = k \cdot \Delta L.$$

Die **Stabsteifigkeit** $k = EA/L$ ist also die Federkonstante des Stabs. Ein
dicker, kurzer Stab aus einem steifen Material ist eine harte Feder. Unser
Stahlstab hat $k \approx 11.66\,\text{kN/mm}$: Um ihn um einen Millimeter zu
dehnen, brauchen wir eine Kraft von rund $11.7\,\text{kN}$.

```{figure} pics/chap04_federanalogie.svg
:alt: Ein Stab mit Länge L, Querschnitt A und Elastizitätsmodul E neben einer Feder mit Federkonstante k gleich EA durch L
:align: center

Ein Stab unter Zug oder Druck verhält sich wie eine lineare Feder mit der
Federkonstanten $k = EA/L$.
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

Anders als eine Feder auf dem Tisch liegt unser Stab schräg im Raum. Er
überträgt nur Kräfte in Richtung seiner Achse. Diese Richtung beschreiben wir
durch den **Einheitsvektor** $\vec{e}$, den Differenzvektor geteilt durch die
Stablänge. Was passiert, wenn sich Knoten 1 um $1\,\text{mm}$ nach unten
verschiebt?

```{code-cell} python
# Einheitsvektor in Richtung der Stabachse (Länge 1)
e = differenz / stablaenge

# Knoten 1 verschiebt sich um 1 mm nach unten, Knoten 0 bleibt stehen
verschiebung_knoten1 = np.array([0.0, -0.001])   # in m

# Längenänderung: nur der Anteil der Verschiebung in Stabrichtung zählt
delta_l = e[0] * verschiebung_knoten1[0] + e[1] * verschiebung_knoten1[1]

# Kraft an Knoten 1, die diese Verschiebung erzeugt: Betrag k * delta_l, Richtung e
kraft_knoten1 = k * delta_l * e

print(f'Einheitsvektor e:  {e}')
print(f'Längenänderung:    {delta_l * 1000:.4f} mm')
print(f'Kraft an Knoten 1: {kraft_knoten1} N')
```

Der Stab wird nur um rund $0.71\,\text{mm}$ kürzer, nicht um den vollen
Millimeter. Der Grund: Die Verschiebung zeigt senkrecht nach unten, der Stab
aber schräg nach oben. Nur der Anteil der Verschiebung in Stabrichtung ändert
die Länge. Die Kraft zeigt wieder in Stabrichtung, und zwar nach links unten:
Sie drückt den Stab zusammen. Deshalb hat sie auch eine $x$-Komponente, obwohl
sich der Knoten gar nicht seitlich bewegt.

```{figure} pics/chap04_stabgeometrie.svg
:alt: Stab 0 des Kranauslegers mit Länge L, Winkel phi und den Komponenten Delta x und Delta y
:align: center

Der Differenzvektor von Knoten 0 nach Knoten 1 hat die Komponenten $\Delta x$
und $\Delta y$, die Stablänge $L$ und den Winkel $\varphi$ zur $x$-Achse.
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

Diese Rechnung können wir für jede beliebige Verschiebung wiederholen.
Einfacher geht es mit einer $2 \times 2$-Matrix, die den ganzen Zusammenhang
auf einmal enthält.

```{code-cell} python
# Steifigkeitsblock des Stabs: rechnet eine Verschiebung in eine Kraft um
block = k * np.array([
    [e[0] * e[0], e[0] * e[1]],
    [e[1] * e[0], e[1] * e[1]],
])

print('Steifigkeitsblock in kN/mm:')
print(np.round(block * 1e-6, 2))
print('block @ verschiebung:', block @ verschiebung_knoten1, 'N')
```

Das Produkt `block @ verschiebung_knoten1` liefert genau die Kraft von oben.
Wir nennen diese Matrix den **Steifigkeitsblock** des Stabs. Mit
$\vec{e} = (\cos\varphi, \sin\varphi)^\top$ lautet er

$$\mathbf{b} = k \begin{pmatrix}
\cos^2\varphi & \cos\varphi\,\sin\varphi \\
\cos\varphi\,\sin\varphi & \sin^2\varphi
\end{pmatrix}.$$

Für unseren Stab mit $\varphi = 45°$ sind alle vier Einträge gleich, nämlich
$k/2 \approx 5.83\,\text{kN/mm}$. Den Winkel selbst müssen wir nie ausrechnen,
denn die Einträge von `e` sind bereits $\cos\varphi$ und $\sin\varphi$.

```{admonition} Mini-Übung (✩)
:class: tip
1. Berechnen Sie Länge, Steifigkeit $k$ und Steifigkeitsblock für Stab 1.
2. Beantworten Sie ohne Code: Warum hat der Block von Stab 1 auf der
   Nebendiagonalen ein anderes Vorzeichen als der Block von Stab 0?
3. Beantworten Sie ohne Code: Ein waagerechter Stab hat den Einheitsvektor
   $\vec{e} = (1, 0)^\top$. Welche Einträge seines Steifigkeitsblocks sind
   null? Was bedeutet das für eine Verschiebung senkrecht zum Stab?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 1: Länge, Steifigkeit und Steifigkeitsblock von Stab 1
i1, j1 = staebe[1]
differenz1 = knoten_pos[j1] - knoten_pos[i1]
stablaenge1 = np.sqrt(differenz1[0]**2 + differenz1[1]**2)
k1 = elastizitaetsmodul * querschnitt / stablaenge1
e1 = differenz1 / stablaenge1

block1 = k1 * np.array([
    [e1[0] * e1[0], e1[0] * e1[1]],
    [e1[1] * e1[0], e1[1] * e1[1]],
])

print(f'Stablänge:     {stablaenge1:.4f} m')
print(f'Steifigkeit k: {k1 * 1e-6:.2f} kN/mm')
print(f'Einheitsvektor: {e1}')
print('Steifigkeitsblock in kN/mm:')
print(np.round(block1 * 1e-6, 2))
```
Stab 1 ist genauso lang wie Stab 0 und hat deshalb dieselbe Steifigkeit.
Er zeigt aber von Knoten 1 schräg nach rechts unten, also ist
$\vec{e} = (0.71, -0.71)^\top$. Das Produkt $e_x e_y$ auf der Nebendiagonalen
wird dadurch negativ, die Diagonale bleibt positiv, weil dort Quadrate stehen.

Beim waagerechten Stab ist $e_y = 0$. Damit sind alle Einträge null bis auf
den oberen linken, der Block lautet $k \begin{pmatrix} 1 & 0 \\ 0 & 0
\end{pmatrix}$. Eine kleine Verschiebung senkrecht zum Stab ändert seine Länge
nicht und erzeugt deshalb keine Kraft.
````

## Aus Stäben wird die Steifigkeitsmatrix

Jeder Stab liefert einen Steifigkeitsblock. Jetzt fügen wir alle Blöcke zu
einer großen Matrix zusammen, die das ganze Fachwerk beschreibt. Da wir das
für jedes Fachwerk brauchen, schreiben wir gleich eine Funktion. Im Inneren
steht eine Schleife über die Stabliste, und in der Schleife rechnen wir genau
das, was wir eben für Stab 0 von Hand gemacht haben.

```{code-cell} python
def baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt):
    """Setzt die Steifigkeitsmatrix eines ebenen Fachwerks zusammen.

    knoten_pos: Knotenkoordinaten in m, Zeile n = [x_n, y_n]
    staebe: Stabliste, Zeile s = [i, j]
    elastizitaetsmodul: E in N/m², für alle Stäbe gleich
    querschnitt: A in m², für alle Stäbe gleich
    Rückgabe: Steifigkeitsmatrix K in N/m, Form (2 * Knotenanzahl, 2 * Knotenanzahl)
    """
    anzahl_freiheitsgrade = 2 * len(knoten_pos)
    K = np.zeros((anzahl_freiheitsgrade, anzahl_freiheitsgrade))

    for i, j in staebe:
        # Geometrie und Steifigkeit des Stabs
        differenz = knoten_pos[j] - knoten_pos[i]
        stablaenge = np.sqrt(differenz[0]**2 + differenz[1]**2)
        k = elastizitaetsmodul * querschnitt / stablaenge
        e = differenz / stablaenge

        # Steifigkeitsblock des Stabs
        block = k * np.array([
            [e[0] * e[0], e[0] * e[1]],
            [e[1] * e[0], e[1] * e[1]],
        ])

        # Block in K eintragen: +block bei i-i und j-j, -block bei i-j und j-i
        K[2*i : 2*i + 2, 2*i : 2*i + 2] += block
        K[2*j : 2*j + 2, 2*j : 2*j + 2] += block
        K[2*i : 2*i + 2, 2*j : 2*j + 2] -= block
        K[2*j : 2*j + 2, 2*i : 2*i + 2] -= block

    return K

K = baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt)

print('Steifigkeitsmatrix K in kN/mm:')
print(np.round(K * 1e-6, 2))
```

Die **Steifigkeitsmatrix** $\mathbf{K}$ hat eine Zeile und eine Spalte für
jeden Freiheitsgrad, hier also $6 \times 6$ Einträge. Sie verknüpft die
Verschiebungen aller Knoten mit den Kräften an allen Knoten:

$$\mathbf{K} \cdot \vec{u} = \vec{F}.$$

Die Einträge schreibt die Funktion mit **Slicing** in die Matrix. Der
Ausdruck `K[2:4, 2:4]` wählt die Zeilen 2 und 3 und die Spalten 2 und 3 aus.
Wie bei `range` gehört die obere Grenze nicht mehr dazu. Für Knoten $i$ sind
das die Zeilen und Spalten $2i$ und $2i+1$, also genau seine beiden
Freiheitsgrade.

```{code-cell} python
# Der 2x2-Block von Knoten 1: Zeilen 2 und 3, Spalten 2 und 3
print('K[2:4, 2:4] in kN/mm:')
print(np.round(K[2:4, 2:4] * 1e-6, 2))
```

Im Block von Knoten 1 addieren sich die Beiträge beider Stäbe. Die
Nebendiagonalen heben sich auf, weil die beiden Stäbe spiegelbildlich
liegen. Übrig bleibt eine Steifigkeit von $11.66\,\text{kN/mm}$ in $x$- und in
$y$-Richtung.

Woher kommen die Vorzeichen? Die Kraft in einem Stab hängt nur davon ab, wie
weit sich seine beiden Endknoten *gegeneinander* verschieben. Für die Kräfte
an den Knoten $i$ und $j$ gilt deshalb

$$\vec{F}_i = \mathbf{b}\,(\vec{u}_i - \vec{u}_j), \qquad
\vec{F}_j = \mathbf{b}\,(\vec{u}_j - \vec{u}_i).$$

Daraus entsteht das Muster in der Funktion: $+\mathbf{b}$ auf den beiden
**Diagonalblöcken** und $-\mathbf{b}$ auf den beiden **Nebendiagonalblöcken**.
Wo kein Stab zwei Knoten verbindet, bleibt ein Nullblock stehen.

```{figure} pics/chap04_blockstruktur.svg
:alt: Die Steifigkeitsmatrix als 3 mal 3 Raster aus 2 mal 2 Blöcken mit Diagonalblöcken, Nebendiagonalblöcken und zwei Nullblöcken
:align: center

Blockstruktur der Steifigkeitsmatrix des Kranauslegers: Zwischen Knoten 0 und
Knoten 2 gibt es keinen Stab, deshalb sind diese Blöcke null.
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

Können wir $\mathbf{K} \cdot \vec{u} = \vec{F}$ jetzt einfach mit
`np.linalg.solve` lösen? Wir prüfen die Lösbarkeit wie in Kapitel 3.1 mit der
Determinante. Außerdem schieben wir das ganze Fachwerk probeweise um
$1\,\text{mm}$ nach rechts.

```{code-cell} python
print(f'Determinante von K: {np.linalg.det(K):.4e}')

# Alle drei Knoten verschieben sich um 1 mm nach rechts
u_starr = np.array([0.001, 0.0, 0.001, 0.0, 0.001, 0.0])
print('K @ u_starr:', K @ u_starr, 'N')
```

Die Determinante ist null oder im Rahmen der Rundungsfehler winzig. Die Matrix
ist also **singulär**. Die zweite Rechnung zeigt den Grund: Verschieben wir
das ganze Fachwerk, ändert sich keine Stablänge, und es entsteht keine Kraft.
Die ausgegebenen Werte in der Größenordnung von $10^{-13}\,\text{N}$ sind
Rundungsfehler, physikalisch ist das null.
Ohne Lager könnte das Fachwerk wegrutschen, und zu einer Last gäbe es keine
eindeutige Verschiebung. Die Lager fehlen in $\mathbf{K}$ noch. Wir bauen sie
in Kapitel 4.3 ein.

```{admonition} Mini-Übung (✩)
:class: tip
1. Beantworten Sie ohne Code: Warum ist der Block `K[0:2, 4:6]` null?
2. Beantworten Sie ohne Code: Warum stimmt der Block `K[0:2, 0:2]` genau mit
   dem Steifigkeitsblock von Stab 0 überein?
3. Wir schließen das Dreieck mit einem dritten Stab zwischen Knoten 0 und
   Knoten 2. Legen Sie dafür eine neue Stabliste `staebe_neu` an, bauen Sie
   die Steifigkeitsmatrix neu auf und geben Sie `K_neu[0:2, 4:6]` aus. Was
   hat sich geändert?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 3: dritter Stab zwischen Knoten 0 und Knoten 2
staebe_neu = np.array([
    [0, 1],
    [1, 2],
    [0, 2],   # neuer Stab zwischen den beiden Lagerknoten
])
K_neu = baue_steifigkeitsmatrix(knoten_pos, staebe_neu,
                                elastizitaetsmodul, querschnitt)

print('K_neu[0:2, 4:6] in kN/mm:')
print(np.round(K_neu[0:2, 4:6] * 1e-6, 2))
```
Der Block `K[0:2, 4:6]` verknüpft die Freiheitsgrade von Knoten 0 mit denen
von Knoten 2. Zwischen diesen Knoten gibt es im Kranausleger keinen Stab,
deshalb trägt die Funktion dort nichts ein. An Knoten 0 hängt nur Stab 0,
deshalb enthält sein Diagonalblock nur dessen Steifigkeitsblock.

Mit dem neuen Stab ist `K_neu[0:2, 4:6]` nicht mehr null. Der Stab ist
waagerecht und $2\,\text{m}$ lang, seine Steifigkeit beträgt
$k = 8.25\,\text{kN/mm}$. Im Block steht deshalb nur links oben ein Eintrag,
nämlich $-8.25\,\text{kN/mm}$. Das Minuszeichen stammt vom
Nebendiagonalblock.
````

## Zusammenfassung und Ausblick

Ein ebenes Fachwerk beschreiben wir durch vier Datenstrukturen: die
Knotenkoordinaten, die Lagerknoten, die Stabliste und den Kraftvektor. Jeder
Knoten hat zwei Freiheitsgrade, Knoten $n$ belegt die Indizes $2n$ und $2n+1$.
Jeder Stab wirkt wie eine Feder mit der Steifigkeit $k = EA/L$ und liefert
einen $2 \times 2$-Steifigkeitsblock. Die Funktion `baue_steifigkeitsmatrix`
setzt alle Blöcke zur Steifigkeitsmatrix $\mathbf{K}$ zusammen. Ohne Lager ist
$\mathbf{K}$ singulär, weil das Fachwerk als Ganzes verschoben werden könnte.

Im nächsten Kapitel wenden wir die Funktion in Partnerarbeit auf ein größeres
Fachwerk an, eine Wandkonsole für eine Rohrleitung. In Kapitel 4.3 bauen wir anschließend die Lager ein,
lösen das Gleichungssystem und berechnen, wie weit sich die Spitze des
Kranauslegers absenkt.
