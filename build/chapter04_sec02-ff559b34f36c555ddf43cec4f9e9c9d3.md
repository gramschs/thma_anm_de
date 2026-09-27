---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.2 Steifigkeitsmatrix einer Wandkonsole

In Kapitel 4.1 haben wir den Kranausleger im Rechner beschrieben und seine
Steifigkeitsmatrix aufgebaut. In diesem Kapitel wenden wir dasselbe Vorgehen
auf ein größeres Fachwerk mit fünf Knoten und sechs Stäben an. Bearbeiten Sie
die Teilaufgaben möglichst zu zweit und der Reihe nach.

````{admonition} Projekt: Steifigkeitsmatrix einer Wandkonsole (✩✩)
:class: tip
An einer Hallenwand ist eine Konsole aus Stahlstäben befestigt. An ihrer Spitze
hängt eine Rohrleitung, die mit $3000\,\text{N}$ nach unten zieht. Die beiden
Knoten an der Wand sind Festlager.

```{figure} pics/chap04_wandkonsole.svg
:alt: Wandkonsole aus sechs Stäben mit fünf Knoten, zwei Festlagern an der Wand und einer Last an der Spitze
:align: center

Die Wandkonsole mit Knotennummern, Stabnummern und der Last an Knoten 4.
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

| Knoten | $x$ in m | $y$ in m | Bemerkung |
| --- | --- | --- | --- |
| 0 | 0.0 | 0.0 | Festlager |
| 1 | 0.0 | 1.0 | Festlager |
| 2 | 1.0 | 0.0 | frei |
| 3 | 1.0 | 1.0 | frei |
| 4 | 2.0 | 0.0 | frei, hier hängt die Rohrleitung |

| Stab | verbindet Knoten | Bauteil |
| --- | --- | --- |
| 0 | 0 und 2 | Untergurt innen |
| 1 | 2 und 4 | Untergurt außen |
| 2 | 1 und 3 | Obergurt |
| 3 | 2 und 3 | Pfosten |
| 4 | 1 und 2 | Diagonale |
| 5 | 3 und 4 | Schräge |

Alle Stäbe sind aus Stahl ($E = 2.1 \cdot 10^{11}\,\text{N/m}^2$) und haben
einen runden Querschnitt mit $1\,\text{cm}$ Durchmesser.
````

Führen Sie zuerst die beiden folgenden Zellen aus. Die erste enthält die
vorgegebene Zeichenfunktion, die zweite die Importe und die Funktion
`baue_steifigkeitsmatrix` aus Kapitel 4.1.

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
import numpy as np
import matplotlib.pyplot as plt

# Funktion aus Kapitel 4.1
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
```

```{admonition} Teil 1: Das Fachwerk beschreiben und zeichnen
:class: tip
Legen Sie `knoten_pos`, `lager_indizes`, `staebe` und `kraft_vektor` an.
Halten Sie bei der Stabliste die Reihenfolge aus der Tabelle ein. Zeichnen Sie
das Fachwerk mit `zeichne_fachwerk` und vergleichen Sie den Plot mit der
Skizze.

Beantworten Sie außerdem: Wie viele Freiheitsgrade hat die Konsole, und an
welchem Index von `kraft_vektor` steht die Last?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
```python
# Knotenkoordinaten in m: Zeile n enthält [x, y] von Knoten n
knoten_pos = np.array([
    [0.0, 0.0],   # Knoten 0: Festlager unten an der Wand
    [0.0, 1.0],   # Knoten 1: Festlager oben an der Wand
    [1.0, 0.0],   # Knoten 2
    [1.0, 1.0],   # Knoten 3
    [2.0, 0.0],   # Knoten 4: Spitze mit Rohrleitung
])
anzahl_knoten = len(knoten_pos)

lager_indizes = [0, 1]

# Stabliste in der Reihenfolge der Tabelle
staebe = np.array([
    [0, 2],   # Stab 0: Untergurt innen
    [2, 4],   # Stab 1: Untergurt außen
    [1, 3],   # Stab 2: Obergurt
    [2, 3],   # Stab 3: Pfosten
    [1, 2],   # Stab 4: Diagonale
    [3, 4],   # Stab 5: Schräge
])

# Last an Knoten 4 in y-Richtung: Index 2 * 4 + 1 = 9
kraft_vektor = np.zeros(2 * anzahl_knoten)
kraft_vektor[9] = -3000.0

print('Anzahl Freiheitsgrade:', 2 * anzahl_knoten)
zeichne_fachwerk(knoten_pos, staebe, lager_indizes, kraft_vektor=kraft_vektor,
                 titel='Wandkonsole')
```
Die Konsole hat fünf Knoten mit je zwei Freiheitsgraden, also zehn
Freiheitsgrade. Die Last wirkt an Knoten 4 in $y$-Richtung und steht deshalb
an Index $2 \cdot 4 + 1 = 9$. Das Minuszeichen bedeutet, dass die Kraft nach
unten zeigt.
````

```{admonition} Teil 2: Eine Stabtabelle ausgeben
:class: tip
Legen Sie Elastizitätsmodul und Querschnitt an. Schreiben Sie dann eine
Schleife über alle Stäbe, die für jeden Stab die Stabnummer, die beiden
Knoten, die Länge in m und die Steifigkeit $k$ in kN/mm ausgibt. Eine
Schleife der Form `for s in range(len(staebe)):` liefert die Stabnummer `s`,
mit `i, j = staebe[s]` erhalten Sie die beiden Knoten.

Welche Stäbe sind am weichsten, und woran liegt das?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 2
:class: tip
:class: dropdown
```python
elastizitaetsmodul = 2.1e11                    # Stahl in N/m²
durchmesser = 0.01                             # in m
querschnitt = np.pi * durchmesser**2 / 4       # Kreisfläche in m²

print('Stab  Knoten  Länge in m  k in kN/mm')
for s in range(len(staebe)):
    i, j = staebe[s]
    differenz = knoten_pos[j] - knoten_pos[i]
    stablaenge = np.sqrt(differenz[0]**2 + differenz[1]**2)
    k = elastizitaetsmodul * querschnitt / stablaenge
    print(f'{s:4d}   {i} - {j}  {stablaenge:10.4f}  {k * 1e-6:10.2f}')
```
Ausgabe:

```text
Stab  Knoten  Länge in m  k in kN/mm
   0   0 - 2      1.0000       16.49
   1   2 - 4      1.0000       16.49
   2   1 - 3      1.0000       16.49
   3   2 - 3      1.0000       16.49
   4   1 - 2      1.4142       11.66
   5   3 - 4      1.4142       11.66
```

Die beiden schrägen Stäbe 4 und 5 sind am weichsten. Alle Stäbe haben dasselbe
Material und denselben Querschnitt, die Steifigkeit $k = EA/L$ hängt also nur
noch von der Länge ab. Die schrägen Stäbe sind mit $\sqrt{2}\,\text{m}$ die
längsten.
````

```{admonition} Teil 3: Die Steifigkeitsmatrix aufbauen
:class: tip
Bauen Sie die Steifigkeitsmatrix `K` mit der Funktion
`baue_steifigkeitsmatrix` auf. Geben Sie ihre Form und die Matrix in kN/mm
aus, gerundet auf eine Nachkommastelle.

Prüfen Sie anschließend mit `np.allclose(K, K.T)`, ob `K` symmetrisch ist.
`K.T` ist die **transponierte** Matrix, bei der Zeilen und Spalten vertauscht
sind. Warum muss die Steifigkeitsmatrix symmetrisch sein?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 3
:class: tip
:class: dropdown
```python
K = baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt)

print('Form von K:', K.shape)
print('K in kN/mm:')
print(np.round(K * 1e-6, 1))
print('K ist symmetrisch:', np.allclose(K, K.T))
```
`K` hat die Form `(10, 10)`, eine Zeile und eine Spalte für jeden
Freiheitsgrad. Die Matrix ist symmetrisch, weil jeder Stab zwei
Eigenschaften mitbringt. Erstens ist sein Steifigkeitsblock selbst
symmetrisch, denn oben rechts und unten links steht jeweils $k\,e_x e_y$.
Zweitens trägt die Funktion den Block spiegelbildlich ein: $-\mathbf{b}$
steht sowohl bei Knoten $i$ und $j$ als auch bei Knoten $j$ und $i$.
````

```{admonition} Teil 4: Die Blockstruktur lesen
:class: tip
1. Sagen Sie zuerst ohne Code vorher, welche der folgenden Blöcke null sind:
   `K[0:2, 8:10]`, `K[4:6, 8:10]` und `K[2:4, 4:6]`. Welche Knoten gehören
   jeweils zu dem Block? Geben Sie die drei Blöcke danach in kN/mm aus und
   überprüfen Sie Ihre Vorhersage.
2. Geben Sie den Diagonalblock von Knoten 4 aus. Der Eintrag für die
   $x$-Richtung ist deutlich größer als der für die $y$-Richtung. Welche
   Stäbe tragen zu den beiden Einträgen bei? Was bedeutet das für die
   Rohrleitung?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 4
:class: tip
:class: dropdown
```python
print('K[0:2, 8:10], Knoten 0 und 4, in kN/mm:')
print(np.round(K[0:2, 8:10] * 1e-6, 2))
print('K[4:6, 8:10], Knoten 2 und 4, in kN/mm:')
print(np.round(K[4:6, 8:10] * 1e-6, 2))
print('K[2:4, 4:6], Knoten 1 und 2, in kN/mm:')
print(np.round(K[2:4, 4:6] * 1e-6, 2))

# Diagonalblock von Knoten 4: Zeilen und Spalten 8 und 9
print('K[8:10, 8:10], Knoten 4, in kN/mm:')
print(np.round(K[8:10, 8:10] * 1e-6, 2))
```
Ausgabe:

```text
K[0:2, 8:10], Knoten 0 und 4, in kN/mm:
[[0. 0.]
 [0. 0.]]
K[4:6, 8:10], Knoten 2 und 4, in kN/mm:
[[-16.49   0.  ]
 [  0.     0.  ]]
K[2:4, 4:6], Knoten 1 und 2, in kN/mm:
[[-5.83  5.83]
 [ 5.83 -5.83]]
K[8:10, 8:10], Knoten 4, in kN/mm:
[[22.32 -5.83]
 [-5.83  5.83]]
```

Nur der Block von Knoten 0 und Knoten 4 ist null, denn zwischen diesen beiden
Knoten gibt es keinen Stab. Knoten 2 und 4 sind durch den waagerechten Stab 1
verbunden, deshalb steht nur links oben ein Eintrag. Knoten 1 und 2 sind durch
die Diagonale verbunden, deshalb sind alle vier Einträge belegt.

Im Diagonalblock von Knoten 4 addieren sich die Beiträge der Stäbe 1 und 5.
In $x$-Richtung wirken beide: Stab 1 mit $16.49\,\text{kN/mm}$ und die Schräge
mit $5.83\,\text{kN/mm}$. In $y$-Richtung wirkt nur die Schräge, denn der
waagerechte Stab 1 kann keine senkrechte Kraft aufnehmen. Die Spitze gibt
deshalb in $y$-Richtung am leichtesten nach, also genau in der Richtung, in
der die Rohrleitung zieht.
````

```{admonition} Abschlussfrage
:class: tip
Die Steifigkeitsmatrix der Konsole ist singulär, obwohl wir die Lager in
`lager_indizes` festgelegt haben. Warum? Nennen Sie Bewegungen der ganzen
Konsole, bei denen keine Kraft entsteht.
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
Die Funktion `baue_steifigkeitsmatrix` bekommt `lager_indizes` gar nicht
übergeben. Die Matrix `K` beschreibt deshalb die Konsole, als wäre sie nicht
an der Wand befestigt. Eine solche freie Konsole können wir als Ganzes nach
rechts oder links schieben, nach oben oder unten schieben oder ein kleines
Stück drehen. Bei all diesen Bewegungen ändert sich keine Stablänge, also
entsteht auch keine Kraft. Zu einer gegebenen Last gibt es dann keine
eindeutige Verschiebung, und genau das zeigt die Determinante null an. In
Kapitel 4.3 bauen wir die Lager in das Gleichungssystem ein.
````

```{admonition} Zusatzaufgabe: Drehen, ohne dass eine Kraft entsteht (✩✩✩)
:class: tip
Wir drehen die freie Konsole um einen kleinen Winkel $\alpha = 0.001$ um
Knoten 0. Für eine kleine Drehung verschiebt sich Knoten $n$ mit den
Koordinaten $(x_n, y_n)$ um

$$u_{x,n} = -\alpha\, y_n, \qquad u_{y,n} = \alpha\, x_n.$$

1. Legen Sie mit einer Schleife über alle Knoten den Verschiebungsvektor
   `u_drehung` an. Berechnen Sie `K @ u_drehung`.
2. Zum Vergleich verschieben Sie nur Knoten 4 um $1\,\text{mm}$ nach unten
   und berechnen ebenfalls die Kräfte. Was fällt Ihnen auf?
3. Zeichnen Sie die gedrehte Konsole mit
   `zeichne_fachwerk(..., verschiebung=u_drehung, skalierung=100)`.
   Die Verschiebungen sind dabei 100-fach überhöht dargestellt.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Zusatzaufgabe
:class: tip
:class: dropdown
```python
# Teilaufgabe 1: Verschiebungsvektor der kleinen Drehung um Knoten 0
alpha = 0.001   # Drehwinkel in rad
u_drehung = np.zeros(2 * anzahl_knoten)
for n in range(anzahl_knoten):
    u_drehung[2 * n] = -alpha * knoten_pos[n, 1]      # x-Verschiebung
    u_drehung[2 * n + 1] = alpha * knoten_pos[n, 0]   # y-Verschiebung

print('Kräfte bei Drehung in N:')
print(np.round(K @ u_drehung, 6))

# Teilaufgabe 2: nur Knoten 4 um 1 mm nach unten
u_knoten4 = np.zeros(2 * anzahl_knoten)
u_knoten4[9] = -0.001
print('Kräfte bei Verschiebung von Knoten 4 in N:')
print(np.round(K @ u_knoten4, 1))

# Teilaufgabe 3: gedrehte Konsole, 100-fach überhöht
zeichne_fachwerk(knoten_pos, staebe, lager_indizes, verschiebung=u_drehung,
                 skalierung=100, titel='Kleine Drehung der freien Konsole')
```
Bei der Drehung sind alle Kräfte null. Die Ausgabe `-0.` ist ein winziger
negativer Rundungsfehler, der auf null gerundet wurde. Bei der
Verschiebung von Knoten 4 entstehen dagegen Kräfte von mehreren tausend
Newton an den Knoten 3 und 4. Der Grund: Bei der Drehung verschieben sich die
beiden Enden jedes Stabs gegeneinander nur senkrecht zur Stabachse. Deshalb
bleiben alle Stablängen gleich, und kein Stab wird gedehnt oder gestaucht. Die Drehung ist neben den beiden Verschiebungen in $x$ und $y$ die
dritte Bewegung, die ein ebenes Fachwerk ohne Lager kraftfrei ausführen kann.
Im Plot sieht man außerdem, wie sich die Konsole von ihren Lagern löst: Die
Matrix `K` weiß noch nichts von der Wand.
````
