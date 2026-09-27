---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.4 Dachbinder einer Stahlhalle unter Schneelast

In Kapitel 4.3 haben wir die Lager eingebaut, Verschiebungen, Lagerkräfte und
Stabkräfte berechnet und den Kranausleger auf Spannung und Knicken geprüft.
In diesem Kapitel setzen wir die drei Funktionen ein, um einen Dachbinder zu
berechnen und zu bewerten. Bearbeiten Sie die Teilaufgaben möglichst zu
zweit und der Reihe nach.

````{admonition} Projekt: Dachbinder einer Stahlhalle (✩✩)
:class: tip
Das Dach einer kleinen Stahlhalle ruht auf Dachbindern mit $6\,\text{m}$
Spannweite und $2\,\text{m}$ Firsthöhe. Nach starkem Schneefall drückt auf
jeden der drei oberen Knoten eine Last von $2000\,\text{N}$ nach unten. Der
Binder liegt links auf einem Festlager und rechts auf einem Loslager, wie der
Träger in Kapitel 3.2.

```{figure} pics/chap04_dachbinder.svg
:alt: Dachbinder aus elf Stäben mit sieben Knoten, Festlager links, Loslager rechts und drei Schneelasten auf den oberen Knoten
:align: center

Der Dachbinder mit Knotennummern, Stabnummern und der Schneelast.
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

| Knoten | $x$ in m | $y$ in m | Bemerkung |
| --- | --- | --- | --- |
| 0 | 0.0 | 0.0 | Festlager |
| 1 | 2.0 | 0.0 | frei |
| 2 | 4.0 | 0.0 | frei |
| 3 | 6.0 | 0.0 | Loslager |
| 4 | 2.0 | 4/3 | frei, Schneelast |
| 5 | 3.0 | 2.0 | frei, Schneelast (First) |
| 6 | 4.0 | 4/3 | frei, Schneelast |

| Stab | verbindet Knoten | Bauteil |
| --- | --- | --- |
| 0, 1, 2 | 0 und 1, 1 und 2, 2 und 3 | Untergurt |
| 3, 4, 5, 6 | 0 und 4, 4 und 5, 5 und 6, 6 und 3 | Obergurt |
| 7, 8 | 1 und 4, 2 und 6 | Pfosten |
| 9, 10 | 1 und 5, 2 und 5 | Diagonalen |

Alle Stäbe sind Rundstäbe aus Baustahl S235 mit $2\,\text{cm}$ Durchmesser,
$E = 2.1 \cdot 10^{11}\,\text{N/m}^2$ und der Streckgrenze
$R_e = 235\,\text{N/mm}^2$.
````

Führen Sie zuerst die beiden folgenden Zellen aus. Die erste enthält die
vorgegebene Zeichenfunktion, die zweite die Importe und die drei Funktionen
aus den Kapiteln 4.1 und 4.3.

Die Funktion `berechne_verschiebungen` hat einen zusätzlichen Parameter
`loslager_indizes`. Ein Loslager wie das Lager B in Kapitel 3.2 hält den
Knoten nur in senkrechter Richtung fest, waagerecht kann er sich frei
bewegen. Deshalb ersetzt die Funktion an einem Loslager nur die Gleichung für
$u_y$. Auch `zeichne_fachwerk` kennt diesen Parameter und zeichnet Loslager
mit einem zusätzlichen Strich.

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


# Funktion aus Kapitel 4.3, erweitert um Loslager
def berechne_verschiebungen(K, kraft_vektor, lager_indizes, loslager_indizes=[]):
    """Löst K * u = F für ein Fachwerk mit Fest- und Loslagern.

    K: Steifigkeitsmatrix in N/m
    kraft_vektor: äußere Knotenkräfte in N
    lager_indizes: Liste der Knoten mit Festlager, dort gilt ux = uy = 0
    loslager_indizes: Liste der Knoten mit Loslager, dort gilt nur uy = 0
    Rückgabe: Verschiebungsvektor u in m
    """
    K_lager = K.copy()
    kraft_lager = kraft_vektor.copy()
    for n in lager_indizes:
        for d in [2 * n, 2 * n + 1]:
            K_lager[d, :] = 0.0
            K_lager[d, d] = 1.0
            kraft_lager[d] = 0.0
    # neu: am Loslager nur die Gleichung für die y-Richtung ersetzen
    for n in loslager_indizes:
        d = 2 * n + 1
        K_lager[d, :] = 0.0
        K_lager[d, d] = 1.0
        kraft_lager[d] = 0.0
    return np.linalg.solve(K_lager, kraft_lager)


# Funktion aus Kapitel 4.3
def berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul, querschnitt, u):
    """Berechnet die Stabkräfte eines Fachwerks aus den Verschiebungen.

    knoten_pos: Knotenkoordinaten in m, Zeile n = [x_n, y_n]
    staebe: Stabliste, Zeile s = [i, j]
    elastizitaetsmodul: E in N/m², für alle Stäbe gleich
    querschnitt: A in m², für alle Stäbe gleich
    u: Verschiebungsvektor in m
    Rückgabe: Stabkräfte in N, positiv = Zug, negativ = Druck
    """
    stabkraefte = np.zeros(len(staebe))
    for s in range(len(staebe)):
        i, j = staebe[s]
        # Geometrie und Steifigkeit wie in baue_steifigkeitsmatrix
        differenz = knoten_pos[j] - knoten_pos[i]
        stablaenge = np.sqrt(differenz[0]**2 + differenz[1]**2)
        k = elastizitaetsmodul * querschnitt / stablaenge
        e = differenz / stablaenge

        # Verschiebung von Knoten j gegenüber Knoten i
        u_relativ = u[2*j : 2*j + 2] - u[2*i : 2*i + 2]

        # Längenänderung = Anteil in Stabrichtung, Stabkraft N = k * delta_l
        delta_l = e[0] * u_relativ[0] + e[1] * u_relativ[1]
        stabkraefte[s] = k * delta_l
    return stabkraefte
```

```{admonition} Teil 1: Den Dachbinder beschreiben und zeichnen
:class: tip
Legen Sie die Materialwerte, `knoten_pos`, `lager_indizes` für das Festlager,
`loslager_indizes` für das Loslager, `staebe` und `kraft_vektor` an. Halten
Sie bei der Stabliste die Reihenfolge der Stabnummern ein. Zeichnen Sie den
Binder mit

`zeichne_fachwerk(knoten_pos, staebe, lager_indizes, loslager_indizes=loslager_indizes, kraft_vektor=kraft_vektor)`

und vergleichen Sie den Plot mit der Skizze.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
```python
# Material: Rundstäbe aus S235 mit 2 cm Durchmesser
elastizitaetsmodul = 2.1e11                  # in N/m²
durchmesser = 0.02                           # in m
querschnitt = np.pi * durchmesser**2 / 4     # in m²
streckgrenze = 235.0                         # in N/mm²

# Knotenkoordinaten in m
knoten_pos = np.array([
    [0.0, 0.0],       # Knoten 0: Festlager
    [2.0, 0.0],       # Knoten 1
    [4.0, 0.0],       # Knoten 2
    [6.0, 0.0],       # Knoten 3: Loslager
    [2.0, 4 / 3],     # Knoten 4
    [3.0, 2.0],       # Knoten 5: First
    [4.0, 4 / 3],     # Knoten 6
])
anzahl_knoten = len(knoten_pos)

lager_indizes = [0]      # Festlager
loslager_indizes = [3]   # Loslager

staebe = np.array([
    [0, 1], [1, 2], [2, 3],           # Stäbe 0 bis 2: Untergurt
    [0, 4], [4, 5], [5, 6], [6, 3],   # Stäbe 3 bis 6: Obergurt
    [1, 4], [2, 6],                   # Stäbe 7 und 8: Pfosten
    [1, 5], [2, 5],                   # Stäbe 9 und 10: Diagonalen
])

# Schneelast: Fy an den Knoten 4, 5 und 6 steht an den Indizes 9, 11 und 13
kraft_vektor = np.zeros(2 * anzahl_knoten)
kraft_vektor[9] = -2000.0
kraft_vektor[11] = -2000.0
kraft_vektor[13] = -2000.0

zeichne_fachwerk(knoten_pos, staebe, lager_indizes,
                 loslager_indizes=loslager_indizes, kraft_vektor=kraft_vektor,
                 titel='Dachbinder unter Schneelast')
```
Die Koordinaten der Knoten 4 und 6 schreiben wir als `4 / 3`, dann rechnet
Python den Wert exakt genug aus. Die Schneelast an Knoten $n$ steht an Index
$2n + 1$, also an den Indizes 9, 11 und 13.
````

```{admonition} Teil 2: Vorhersagen, dann rechnen
:class: tip
1. Überlegen Sie ohne Code: Steht der Untergurt unter Zug oder unter Druck?
   Und der Obergurt? Notieren Sie Ihre Vorhersage.
2. Berechnen Sie mit den drei Funktionen die Steifigkeitsmatrix, die
   Verschiebungen und die Stabkräfte. Geben Sie die Stabkräfte mit Zug oder
   Druck aus und zeichnen Sie den Binder mit Stabkräften und 200-fach
   überhöhten Verschiebungen. Stimmt Ihre Vorhersage?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 2
:class: tip
:class: dropdown
```python
K = baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt)
u = berechne_verschiebungen(K, kraft_vektor, lager_indizes, loslager_indizes)
stabkraefte = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                   querschnitt, u)

for s in range(len(staebe)):
    art = 'Zug' if stabkraefte[s] > 0 else 'Druck'
    print(f'Stab {s:2d}: N = {stabkraefte[s]:8.1f} N  ({art})')
print(f'Absenkung des Firsts: {u[11] * 1000:.3f} mm')

zeichne_fachwerk(knoten_pos, staebe, lager_indizes,
                 loslager_indizes=loslager_indizes, kraft_vektor=kraft_vektor,
                 verschiebung=u, skalierung=200, stabkraefte=stabkraefte,
                 titel='Dachbinder, Verschiebungen 200-fach überhöht')
```
Ausgabe:

```text
Stab  0: N =   4500.0 N  (Zug)
Stab  1: N =   3500.0 N  (Zug)
Stab  2: N =   4500.0 N  (Zug)
Stab  3: N =  -5408.3 N  (Druck)
Stab  4: N =  -5408.3 N  (Druck)
Stab  5: N =  -5408.3 N  (Druck)
Stab  6: N =  -5408.3 N  (Druck)
Stab  7: N =  -2000.0 N  (Druck)
Stab  8: N =  -2000.0 N  (Druck)
Stab  9: N =   2236.1 N  (Zug)
Stab 10: N =   2236.1 N  (Zug)
Absenkung des Firsts: -0.817 mm
```

Der Untergurt steht unter Zug, der Obergurt unter Druck. Die Schneelast
drückt die schrägen Obergurtstäbe zusammen, und diese wollen die beiden
Auflager nach außen schieben. Der Untergurt hält die Auflager zusammen wie
ein Zugband. Die Pfosten leiten die Last der Knoten 4 und 6 auf Druck nach
unten in den Untergurt, die Diagonalen führen sie auf Zug wieder hinauf zum
First. Der First senkt sich um rund $0.8\,\text{mm}$ ab.
````

```{admonition} Teil 3: Lagerkräfte und Gleichgewicht
:class: tip
Berechnen Sie die Knotenkräfte $\mathbf{K} \cdot \vec{u}$ und geben Sie die
Kräfte an den Lagerknoten 0 und 3 aus.

1. Warum trägt jedes Lager genau die Hälfte der gesamten Schneelast?
2. Warum ist die waagerechte Lagerkraft am Festlager null?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 3
:class: tip
:class: dropdown
```python
knotenkraefte = K @ u

for n in [0, 3]:
    print(f'Knoten {n}: Fx = {knotenkraefte[2*n]:8.1f} N,  '
          f'Fy = {knotenkraefte[2*n + 1]:8.1f} N')
```
Ausgabe:

```text
Knoten 0: Fx =      0.0 N,  Fy =   3000.0 N
Knoten 3: Fx =      0.0 N,  Fy =   3000.0 N
```

Die gesamte Schneelast beträgt $3 \cdot 2000\,\text{N} = 6000\,\text{N}$.
Binder und Last sind spiegelbildlich zur Mitte, deshalb trägt jedes Lager
$3000\,\text{N}$ nach oben. Waagerecht wirken nur Lagerkräfte, denn die
Schneelast zeigt senkrecht nach unten. Das Loslager kann keine waagerechte
Kraft aufnehmen. Damit die Summe der waagerechten Kräfte null ist, muss auch
die waagerechte Kraft am Festlager null sein, genau wie beim Träger in
Kapitel 3.2.
````

```{admonition} Teil 4: Hält der Dachbinder?
:class: tip
Prüfen Sie jeden Stab mit einer Schleife:

- Berechnen Sie die Spannung $\sigma = N/A$ in N/mm² und die Auslastung
  $|\sigma| / R_e$.
- Für jeden Druckstab berechnen Sie zusätzlich die Euler-Knicklast
  $F_\text{krit} = \pi^2 E I / L^2$ mit $I = \pi d^4 / 64$ und vergleichen
  sie mit dem Betrag der Druckkraft.

Welche Stäbe bestehen einen der beiden Nachweise nicht?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 4
:class: tip
:class: dropdown
```python
traegheitsmoment = np.pi * durchmesser**4 / 64   # in m^4

print('Stab   N in N    sigma in N/mm²  Auslastung  Knicklast in N  Knicken?')
for s in range(len(staebe)):
    i, j = staebe[s]
    differenz = knoten_pos[j] - knoten_pos[i]
    stablaenge = np.sqrt(differenz[0]**2 + differenz[1]**2)

    # Spannungsnachweis für alle Stäbe
    spannung = stabkraefte[s] / querschnitt * 1e-6
    auslastung = abs(spannung) / streckgrenze

    # Knicknachweis nur für Druckstäbe
    if stabkraefte[s] < 0:
        knicklast = np.pi**2 * elastizitaetsmodul * traegheitsmoment / stablaenge**2
        knickt = 'ja' if abs(stabkraefte[s]) > knicklast else 'nein'
        print(f'{s:4d}  {stabkraefte[s]:8.1f}  {spannung:10.1f}  {auslastung * 100:9.1f} %'
              f'  {knicklast:12.1f}    {knickt}')
    else:
        print(f'{s:4d}  {stabkraefte[s]:8.1f}  {spannung:10.1f}  {auslastung * 100:9.1f} %'
              f'  {"(Zug)":>12}')
```
Ausgabe:

```text
Stab   N in N    sigma in N/mm²  Auslastung  Knicklast in N  Knicken?
   0    4500.0        14.3        6.1 %         (Zug)
   1    3500.0        11.1        4.7 %         (Zug)
   2    4500.0        14.3        6.1 %         (Zug)
   3   -5408.3       -17.2        7.3 %        2817.4    ja
   4   -5408.3       -17.2        7.3 %       11269.6    nein
   5   -5408.3       -17.2        7.3 %       11269.6    nein
   6   -5408.3       -17.2        7.3 %        2817.4    ja
   7   -2000.0        -6.4        2.7 %        9156.5    nein
   8   -2000.0        -6.4        2.7 %        9156.5    nein
   9    2236.1         7.1        3.0 %         (Zug)
  10    2236.1         7.1        3.0 %         (Zug)
```

Den Spannungsnachweis bestehen alle Stäbe mit großem Abstand, die höchste
Auslastung liegt unter zehn Prozent. Beim Knicken fallen aber die beiden
äußeren Obergurtstäbe 3 und 6 durch: Ihre Knicklast ist nur halb so groß wie
die Druckkraft. Der Dachbinder hält die Schneelast also nicht.
````

```{admonition} Abschlussfrage
:class: tip
Alle vier Obergurtstäbe tragen dieselbe Druckkraft. Trotzdem knicken nur die
Stäbe 3 und 6. Warum? Und warum müssen wir den Untergurt gar nicht auf
Knicken prüfen, obwohl seine Stabkräfte ähnlich groß sind? Schlagen Sie eine
konstruktive Änderung vor, mit der der Binder hält.
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
Die Knicklast $F_\text{krit} = \pi^2 E I / L^2$ sinkt mit dem Quadrat der
Stablänge. Die Stäbe 3 und 6 sind mit $2.40\,\text{m}$ doppelt so lang wie
die Stäbe 4 und 5 mit $1.20\,\text{m}$. Ihre Knicklast ist deshalb nur ein
Viertel so groß, und das reicht für die Druckkraft von $5408\,\text{N}$ nicht
mehr aus.

Der Untergurt steht unter Zug. Ein gezogener Stab wird gestreckt und kann
nicht seitlich ausweichen, Knicken ist nur bei Druck möglich.

Eine Abhilfe ist ein zusätzlicher Knoten in der Mitte der Stäbe 3 und 6, der
über weitere Stäbe mit dem Untergurt verbunden ist. Dann halbiert sich die
Knicklänge, und die Knicklast vervierfacht sich. Alternativ nehmen wir für den
Obergurt dickere Stäbe oder Rohre. Ein Rohr hat bei gleichem Material ein viel
größeres Flächenträgheitsmoment $I$ als ein Vollstab.
````

```{admonition} Zusatzaufgabe: Warum braucht der Binder ein Loslager? (✩✩✩)
:class: tip
Wir ersetzen das Loslager an Knoten 3 durch ein zweites Festlager, also
`lager_indizes = [0, 3]` und keine Loslager. Berechnen Sie Verschiebungen,
Stabkräfte und Lagerkräfte neu und vergleichen Sie mit der bisherigen
Lagerung.

1. Wie ändern sich die Kräfte im Untergurt?
2. Welche waagerechten Kräfte treten jetzt an den Lagern auf? Was bedeutet
   das für die Hallenwände?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Zusatzaufgabe
:class: tip
:class: dropdown
```python
# Zwei Festlager, kein Loslager
u_fest = berechne_verschiebungen(K, kraft_vektor, [0, 3])
stabkraefte_fest = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                        querschnitt, u_fest)
knotenkraefte_fest = K @ u_fest

# Teilaufgabe 1: Untergurt im Vergleich
print('Untergurt       Fest + Los    Fest + Fest')
for s in [0, 1, 2]:
    print(f'Stab {s}:   {stabkraefte[s]:10.1f} N  {stabkraefte_fest[s]:10.1f} N')

# Teilaufgabe 2: Lagerkräfte bei zwei Festlagern
for n in [0, 3]:
    print(f'Knoten {n}: Fx = {knotenkraefte_fest[2*n]:8.1f} N,  '
          f'Fy = {knotenkraefte_fest[2*n + 1]:8.1f} N')
```
Ausgabe:

```text
Untergurt       Fest + Los    Fest + Fest
Stab 0:       4500.0 N       333.3 N
Stab 1:       3500.0 N      -666.7 N
Stab 2:       4500.0 N       333.3 N
Knoten 0: Fx =   4166.7 N,  Fy =   3000.0 N
Knoten 3: Fx =  -4166.7 N,  Fy =   3000.0 N
```

Mit zwei Festlagern ist der Untergurt fast kraftlos, der mittlere Stab steht
sogar leicht unter Druck. Die Aufgabe des Zugbands übernehmen jetzt die
beiden Lager: Sie drücken mit rund $4200\,\text{N}$ waagerecht gegeneinander.
Umgekehrt drückt der Binder die beiden Hallenwände mit dieser Kraft nach
außen. Mauerwerk verträgt solche Schubkräfte schlecht. Außerdem würde jede
Temperaturänderung zusätzliche Zwangskräfte erzeugen, weil sich der Binder
nicht mehr frei ausdehnen kann. Deshalb liegt ein Binder auf einer Seite auf
einem Loslager.
````
