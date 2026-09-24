---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.3 Verschiebungen und Kräfte im Fachwerk

In Kapitel 4.1 haben wir die Steifigkeitsmatrix des Kranauslegers aufgebaut,
in Kapitel 4.2 die einer Wandkonsole. Beide Male war die Matrix singulär,
weil die Lager noch fehlten. In diesem Kapitel bauen wir die Lager ein und
lösen das Gleichungssystem. *Wie weit senkt sich die Spitze des Kranauslegers
unter der Last ab, und hält die Konstruktion die Last überhaupt aus?*

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können Lager in das Gleichungssystem einbauen, indem Sie die
  Gleichungen der gesperrten Freiheitsgrade ersetzen, und die Verschiebungen
  mit `np.linalg.solve` berechnen.
* [ ] Sie können aus den Verschiebungen die **Lagerkräfte** und die
  **Stabkräfte** berechnen und Zug von Druck unterscheiden.
* [ ] Sie können die Spannung in einem Stab mit der Streckgrenze vergleichen
  und wissen, dass Druckstäbe zusätzlich auf **Knicken** geprüft werden
  müssen.
```

## Lager einbauen und das Gleichungssystem lösen

Wir übernehmen den Kranausleger und die Funktion `baue_steifigkeitsmatrix`
aus Kapitel 4.1. Die erste Zelle enthält wieder die vorgegebene
Zeichenfunktion.

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

```{code-cell} python
# Kranausleger aus Kapitel 4.1
knoten_pos = np.array([
    [0.0, 0.0],   # Knoten 0: linkes Lager
    [1.0, 1.0],   # Knoten 1: Spitze, hier hängt die Last
    [2.0, 0.0],   # Knoten 2: rechtes Lager
])
anzahl_knoten = len(knoten_pos)
lager_indizes = [0, 2]
staebe = np.array([
    [0, 1],   # Stab 0
    [1, 2],   # Stab 1
])
kraft_vektor = np.zeros(2 * anzahl_knoten)
kraft_vektor[3] = -5000.0   # Fy an Knoten 1: 5000 N nach unten

elastizitaetsmodul = 2.1e11                  # Stahl in N/m²
durchmesser = 0.01                           # in m
querschnitt = np.pi * durchmesser**2 / 4     # in m²

K = baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt)
```

Das Gleichungssystem $\mathbf{K} \cdot \vec{u} = \vec{F}$ hat sechs
Gleichungen, eine für jeden Freiheitsgrad. Für die Lagerknoten wissen wir
aber schon, wie groß die Verschiebung ist, nämlich null. Wir ersetzen deshalb
die Gleichungen der gesperrten Freiheitsgrade durch genau diese Bedingung.

```{code-cell} python
# Kopien anlegen, damit K und kraft_vektor unverändert bleiben
K_lager = K.copy()
kraft_lager = kraft_vektor.copy()

# Gleichungen der gesperrten Freiheitsgrade ersetzen durch u_d = 0
for n in lager_indizes:
    for d in [2 * n, 2 * n + 1]:
        K_lager[d, :] = 0.0     # ganze Zeile d auf null setzen
        K_lager[d, d] = 1.0     # Diagonale auf eins: 1 * u_d ...
        kraft_lager[d] = 0.0    # ... = 0

print('K_lager in kN/mm:')
print(np.round(K_lager * 1e-6, 2))
print('Zeile 0 von K_lager ohne Umrechnung:', K_lager[0])
print(f'Determinante von K_lager: {np.linalg.det(K_lager):.4e}')
```

In `K_lager` sind die Zeilen 0, 1, 4 und 5 ersetzt. Jede dieser Zeilen
enthält nur noch eine Eins auf der Diagonalen, auf der rechten Seite steht
eine Null. Die Zeile sagt also schlicht $u_d = 0$. In der umgerechneten Matrix
erscheinen die Einsen als `0.`, weil wir alle Einträge mit $10^{-6}$
multiplizieren. Die letzte Ausgabe zeigt Zeile 0 ohne Umrechnung. Die Determinante ist jetzt nicht mehr null, das System ist lösbar.

Die Methode `.copy()` ist wichtig. Ohne sie würden `K_lager` und `K`
dasselbe Array bezeichnen, und wir würden die ursprüngliche
Steifigkeitsmatrix überschreiben. Die brauchen wir aber gleich noch für die
Lagerkräfte.

```{code-cell} python
u = np.linalg.solve(K_lager, kraft_lager)

print('Verschiebungen in mm:')
for n in range(anzahl_knoten):
    print(f'  Knoten {n}: ux = {u[2*n] * 1000:8.4f} mm,  uy = {u[2*n + 1] * 1000:8.4f} mm')
```

Die Spitze senkt sich um $0.43\,\text{mm}$ ab und bewegt sich nicht zur Seite.
Das passt zur Symmetrie: Beide Stäbe sind gleich und liegen spiegelbildlich,
die Last zeigt genau nach unten. Den Wert können wir sogar von Hand prüfen.
In Kapitel 4.1 haben wir gesehen, dass Knoten 1 in beiden Richtungen die
Steifigkeit $11.66\,\text{kN/mm}$ hat. Also gilt
$u_y = -5\,\text{kN} / 11.66\,\text{kN/mm} = -0.43\,\text{mm}$.

Diese Schritte brauchen wir für jedes Fachwerk wieder. Deshalb fassen wir sie
in einer Funktion zusammen.

```{code-cell} python
def berechne_verschiebungen(K, kraft_vektor, lager_indizes):
    """Löst K * u = F für ein Fachwerk mit Festlagern.

    K: Steifigkeitsmatrix in N/m
    kraft_vektor: äußere Knotenkräfte in N
    lager_indizes: Liste der gelagerten Knoten, dort gilt ux = uy = 0
    Rückgabe: Verschiebungsvektor u in m
    """
    K_lager = K.copy()
    kraft_lager = kraft_vektor.copy()
    for n in lager_indizes:
        for d in [2 * n, 2 * n + 1]:
            K_lager[d, :] = 0.0
            K_lager[d, d] = 1.0
            kraft_lager[d] = 0.0
    return np.linalg.solve(K_lager, kraft_lager)

u = berechne_verschiebungen(K, kraft_vektor, lager_indizes)
print('Verschiebungsvektor in mm:', np.round(u * 1000, 4))
```

```{admonition} Mini-Übung (✩)
:class: tip
1. Beantworten Sie ohne Code: Welche Zeilen von `K_lager` würden ersetzt,
   wenn nur Knoten 0 gelagert wäre?
2. An der Spitze greift zusätzlich zur Last eine Seitenkraft von
   $1000\,\text{N}$ nach rechts an. Legen Sie einen neuen Kraftvektor
   `kraft_vektor_seite` an und berechnen Sie die Verschiebungen mit
   `berechne_verschiebungen`.
3. Beantworten Sie ohne Code: Die Verschiebungen der Spitze stehen im selben
   Verhältnis zueinander wie die Kräfte, nämlich $1 : (-5)$. Warum ist das
   bei diesem Fachwerk so?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 2: Last und Seitenkraft an Knoten 1
kraft_vektor_seite = np.zeros(2 * anzahl_knoten)
kraft_vektor_seite[2] = 1000.0    # Fx an Knoten 1
kraft_vektor_seite[3] = -5000.0   # Fy an Knoten 1

u_seite = berechne_verschiebungen(K, kraft_vektor_seite, lager_indizes)
print(f'ux an Knoten 1: {u_seite[2] * 1000:.4f} mm')
print(f'uy an Knoten 1: {u_seite[3] * 1000:.4f} mm')
```
Wäre nur Knoten 0 gelagert, würden nur die Zeilen 0 und 1 ersetzt. Mit der
Seitenkraft bewegt sich die Spitze um $0.0857\,\text{mm}$ nach rechts und wie
bisher um $0.4287\,\text{mm}$ nach unten. Das Verhältnis ist dasselbe wie bei
den Kräften, weil der Block von Knoten 1 die Form
$11.66\,\text{kN/mm} \cdot \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix}$ hat.
Die Spitze ist in beiden Richtungen gleich steif, und die beiden Richtungen
beeinflussen sich nicht gegenseitig. Jede Kraftkomponente wird deshalb
einfach durch dieselbe Steifigkeit geteilt.
````

## Welche Kräfte wirken in den Lagern und in den Stäben?

Die Verschiebungen kennen wir jetzt. Setzen wir sie in die ursprüngliche
Gleichung $\vec{F} = \mathbf{K} \cdot \vec{u}$ ein, erhalten wir die Kräfte an
allen Knoten.

```{code-cell} python
# Knotenkräfte aus der ursprünglichen Steifigkeitsmatrix
knotenkraefte = K @ u

print('Knotenkräfte in N:')
for n in range(anzahl_knoten):
    print(f'  Knoten {n}: Fx = {knotenkraefte[2*n]:8.1f} N,  Fy = {knotenkraefte[2*n + 1]:8.1f} N')

print(f'Summe Fx: {np.sum(knotenkraefte[0::2]):.1f} N')
print(f'Summe Fy: {np.sum(knotenkraefte[1::2]):.1f} N')
```

An Knoten 1 kommt genau die Last heraus, die wir vorgegeben haben. Das ist
unsere Probe. An den Lagerknoten 0 und 2 stehen die **Lagerkräfte**, also die
Kräfte, mit denen die Lager das Fachwerk festhalten. Jedes Lager trägt die
Hälfte der Last nach oben. Zusätzlich drückt das linke Lager mit
$2500\,\text{N}$ nach rechts und das rechte mit $2500\,\text{N}$ nach links.
Alle Kräfte zusammen ergeben null, das Fachwerk ist im Gleichgewicht wie der
Träger in Kapitel 3.2. Der Ausdruck `knotenkraefte[0::2]` wählt jeden zweiten
Eintrag ab Index 0, also alle $x$-Kräfte.

Für die Bemessung wollen wir wissen, wie stark jeder einzelne Stab belastet
wird. Die **Stabkraft** $N$ folgt direkt aus der Federgleichung von
Kapitel 4.1: $N = k \cdot \Delta L$. Die Längenänderung $\Delta L$ ist der
Anteil der Verschiebungen in Stabrichtung, diesmal mit beiden Stabenden.

```{figure} pics/chap04_projektion.svg
:alt: Stab mit der Verschiebung des Endknotens, zerlegt in einen Anteil entlang der Stabachse und einen Anteil senkrecht dazu
:align: center

Nur der Anteil $u^{\parallel}$ der Verschiebung entlang der Stabachse
$\vec{e}$ ändert die Stablänge und erzeugt eine Stabkraft.
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

```{code-cell} python
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

stabkraefte = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                   querschnitt, u)

for s in range(len(staebe)):
    art = 'Zug' if stabkraefte[s] > 0 else 'Druck'
    print(f'Stab {s}: N = {stabkraefte[s]:8.1f} N  ({art})')
```

Beide Stäbe tragen $3536\,\text{N}$ auf **Druck**. Eine positive Stabkraft
bedeutet Zug, der Stab wird länger. Eine negative Stabkraft bedeutet Druck,
der Stab wird kürzer. Das Ergebnis ist plausibel: Die Last drückt die Spitze
nach unten, die Spitze rückt näher an die beiden Lager heran, und beide Stäbe
werden gestaucht. Dass die Stabkraft größer ist als die halbe Last, liegt an
der Schräglage. Jeder Stab muss $2500\,\text{N}$ in senkrechter Richtung
tragen, und bei $45°$ ist die Kraft entlang des Stabs um den Faktor
$\sqrt{2}$ größer: $2500\,\text{N} \cdot \sqrt{2} \approx 3536\,\text{N}$.

```{admonition} Mini-Übung (✩)
:class: tip
1. Beantworten Sie ohne Code: Warum drückt das linke Lager nach rechts und
   nicht nach links?
2. Berechnen Sie die Stabkräfte für den Kraftvektor mit Seitenkraft aus der
   letzten Mini-Übung (Last $5000\,\text{N}$ nach unten und $1000\,\text{N}$
   nach rechts an Knoten 1). Welcher Stab wird stärker belastet?
3. Beantworten Sie ohne Code: Wie groß ist in diesem Fall die Summe der
   waagerechten Lagerkräfte?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 2: Stabkräfte mit Seitenkraft
kraft_vektor_seite = np.zeros(2 * anzahl_knoten)
kraft_vektor_seite[2] = 1000.0
kraft_vektor_seite[3] = -5000.0

u_seite = berechne_verschiebungen(K, kraft_vektor_seite, lager_indizes)
stabkraefte_seite = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                         querschnitt, u_seite)
for s in range(len(staebe)):
    print(f'Stab {s}: N = {stabkraefte_seite[s]:8.1f} N')
```
Stab 0 drückt Knoten 0 schräg nach links unten in das Lager. Damit Knoten 0
an seinem Platz bleibt, muss das Lager nach rechts oben dagegenhalten.

Mit der Seitenkraft trägt Stab 1 rund $4243\,\text{N}$ Druck, Stab 0 nur noch
rund $2828\,\text{N}$. Die Seitenkraft schiebt die Spitze nach rechts, also
in Richtung von Stab 1. Dieser Stab wird dadurch stärker gestaucht, Stab 0
wird etwas entlastet.

Die Summe der waagerechten Lagerkräfte beträgt $-1000\,\text{N}$. Die Lager
müssen die Seitenkraft von $1000\,\text{N}$ nach rechts genau ausgleichen,
damit alle waagerechten Kräfte zusammen null ergeben.
````

## Hält der Kranausleger?

Zuerst stellen wir das Ergebnis grafisch dar. Die Verschiebungen sind mit
weniger als einem Millimeter viel kleiner als die Stäbe, die einen Meter und
mehr lang sind. Damit wir überhaupt etwas sehen, vergrößern wir die
Verschiebungen in der Zeichnung um einen **Überhöhungsfaktor**, hier 500.
Die Rechnung selbst ändert sich dadurch nicht.

```{code-cell} python
zeichne_fachwerk(knoten_pos, staebe, lager_indizes, kraft_vektor=kraft_vektor,
                 verschiebung=u, skalierung=500, stabkraefte=stabkraefte,
                 titel='Kranausleger, Verschiebungen 500-fach überhöht')
```

Gestrichelt ist die Ausgangslage, durchgezogen die überhöhte verformte Lage.
Beide Stäbe sind rot, sie stehen also unter Druck.

Hält der Stahl diese Kräfte aus? Dazu berechnen wir die **Spannung**
$\sigma = N/A$ und vergleichen sie mit der Streckgrenze. Für einen
Baustahl S235 beträgt sie $R_e = 235\,\text{N/mm}^2$.

```{code-cell} python
streckgrenze = 235.0   # Baustahl S235 in N/mm²

# Spannung sigma = N / A, umgerechnet von N/m² in N/mm²
spannungen = stabkraefte / querschnitt * 1e-6

for s in range(len(staebe)):
    auslastung = abs(spannungen[s]) / streckgrenze
    print(f'Stab {s}: sigma = {spannungen[s]:6.1f} N/mm²,  '
          f'Auslastung {auslastung * 100:4.1f} %')
```

Die Spannung beträgt nur rund $45\,\text{N/mm}^2$, also knapp ein Fünftel
der Streckgrenze. Ist der Kranausleger damit sicher? *Nein, denn bei
Druckstäben reicht die Spannung als Nachweis nicht aus.* Ein schlanker Stab
unter Druck kann seitlich ausweichen, er **knickt**, lange bevor der Stahl
zu fließen beginnt. Aus der Festigkeitslehre kennen wir dafür die
Euler-Knicklast. Für einen Stab, der an beiden Enden gelenkig gelagert ist,
lautet sie

$$F_\text{krit} = \frac{\pi^2\,E\,I}{L^2} \qquad \text{mit} \qquad
I = \frac{\pi\,d^4}{64}$$

für einen Kreisquerschnitt.

```{code-cell} python
# Flächenträgheitsmoment des Kreisquerschnitts in m^4
traegheitsmoment = np.pi * durchmesser**4 / 64

# Euler-Knicklast für Stab 0 (beide Stäbe sind gleich lang)
stablaenge = np.sqrt(2.0)
knicklast = np.pi**2 * elastizitaetsmodul * traegheitsmoment / stablaenge**2

print(f'Euler-Knicklast:       {knicklast:8.1f} N')
print(f'Druckkraft im Stab:    {abs(stabkraefte[0]):8.1f} N')
```

Die Knicklast beträgt nur rund $509\,\text{N}$, die Druckkraft ist fast
siebenmal so groß. Die Stäbe mit $1\,\text{cm}$ Durchmesser würden also
knicken, obwohl die Spannung weit unter der Streckgrenze liegt. Für die
Bemessung eines Fachwerks gehören deshalb immer beide Nachweise dazu: die
Spannung für alle Stäbe und das Knicken für alle Druckstäbe.

```{admonition} Mini-Übung (✩)
:class: tip
1. Beantworten Sie ohne Code: Was würden Sie im Plot sehen, wenn Sie
   `skalierung=1` wählen?
2. Wir verwenden Stäbe mit $2\,\text{cm}$ Durchmesser. Berechnen Sie mit den
   drei Funktionen neu: Steifigkeitsmatrix, Verschiebungen und Stabkräfte.
   Berechnen Sie außerdem die Spannung und die Knicklast. Hält der
   Kranausleger jetzt?
3. Beantworten Sie ohne Code: Die Absenkung der Spitze ist auf ein Viertel
   gesunken, die Stabkräfte sind aber gleich geblieben. Warum?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Teilaufgabe 2: Stäbe mit 2 cm Durchmesser
durchmesser_neu = 0.02
querschnitt_neu = np.pi * durchmesser_neu**2 / 4

K_neu = baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt_neu)
u_neu = berechne_verschiebungen(K_neu, kraft_vektor, lager_indizes)
stabkraefte_neu = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                       querschnitt_neu, u_neu)

spannung_neu = stabkraefte_neu[0] / querschnitt_neu * 1e-6
traegheitsmoment_neu = np.pi * durchmesser_neu**4 / 64
knicklast_neu = np.pi**2 * elastizitaetsmodul * traegheitsmoment_neu / np.sqrt(2.0)**2

print(f'Absenkung der Spitze: {u_neu[3] * 1000:.4f} mm')
print(f'Stabkräfte:           {np.round(stabkraefte_neu, 1)} N')
print(f'Spannung:             {spannung_neu:.1f} N/mm²')
print(f'Knicklast:            {knicklast_neu:.1f} N')
```
Mit `skalierung=1` liegen verformte und unverformte Lage praktisch
übereinander. Eine Verschiebung von $0.43\,\text{mm}$ ist bei Stäben von
über einem Meter Länge nicht zu erkennen.

Mit $2\,\text{cm}$ Durchmesser sinkt die Spannung auf rund
$11\,\text{N/mm}^2$, und die Knicklast steigt auf rund $8139\,\text{N}$. Sie
liegt jetzt deutlich über der Druckkraft von $3536\,\text{N}$, der
Kranausleger hält.

Der doppelte Durchmesser vervierfacht den Querschnitt und damit die
Steifigkeit $k = EA/L$. Deshalb sinkt die Absenkung auf ein Viertel. Die
Stabkräfte hängen hier aber gar nicht von der Steifigkeit ab. An der Spitze
treffen sich nur zwei Stäbe, und die beiden Gleichgewichtsbedingungen in
$x$- und $y$-Richtung legen ihre Kräfte schon eindeutig fest, genau wie die
Auflagerkräfte des Trägers in Kapitel 3.2. Die Knicklast wächst sogar mit
$d^4$, also um den Faktor 16.
````

## Zusammenfassung und Ausblick

Die Lager bauen wir ein, indem wir die Gleichungen der gesperrten
Freiheitsgrade durch $u_d = 0$ ersetzen. Danach ist das Gleichungssystem
lösbar, und `np.linalg.solve` liefert alle Verschiebungen. Setzen wir sie in
$\mathbf{K} \cdot \vec{u}$ ein, erhalten wir an den Lagern die Lagerkräfte.
Die Stabkräfte folgen aus der Längenänderung jedes Stabs, positiv bedeutet
Zug und negativ Druck. Für die Bemessung vergleichen wir die Spannung mit der
Streckgrenze und prüfen alle Druckstäbe zusätzlich auf Knicken.

Alle Schritte stecken jetzt in drei Funktionen: `baue_steifigkeitsmatrix`,
`berechne_verschiebungen` und `berechne_stabkraefte`. Im nächsten Kapitel
setzen wir sie in Partnerarbeit ein, um den Dachbinder einer Stahlhalle unter
Schneelast zu berechnen und zu bewerten.
