---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.5 Übungen

Diese Aufgaben sind für das Selbststudium zuhause gedacht und wiederholen den
Stoff der Kapitel 4.1 bis 4.4. Rechnen Sie mit gut eineinhalb Stunden
Bearbeitungszeit. Führen Sie vor den Aufgaben die beiden folgenden Zellen
aus. Sie enthalten die Zeichenfunktion und die drei Funktionen aus den
Kapiteln 4.1, 4.3 und 4.4.

Der Schwierigkeitsgrad steht im Titel jeder Aufgabe:

* ✩ Verständnis: Code und Ausgaben vorhersagen und erklären (ca. 5 min)
* ✩✩ Anwendung: eigenen Code schreiben und Ergebnisse interpretieren (ca. 10 min)
* ✩✩✩ Mini-Projekt: mehrere Konzepte des Parts kombinieren (ca. 30 min)

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


# Funktion aus Kapitel 4.4, mit Loslager
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

````{admonition} Aufgabe 4.1 (✩)
:class: tip
Gegeben ist folgender Code:

```python
import numpy as np

knoten_pos = np.array([
    [0.0, 0.0],
    [1.5, 0.0],
    [1.5, 1.0],
    [0.0, 1.0],
])
staebe = np.array([[0, 1], [1, 2], [2, 3], [3, 0], [0, 2]])
anzahl_knoten = len(knoten_pos)

kraft_vektor = np.zeros(2 * anzahl_knoten)
kraft_vektor[5] = -800.0
```

Notieren Sie Ihre Vermutung, bevor Sie den Code ausführen.

1. Wie viele Freiheitsgrade hat das Fachwerk, und welche Form hat seine
   Steifigkeitsmatrix?
2. Welche Knoten verbindet `staebe[4]`? Wie lang ist dieser Stab?
3. An welchem Knoten und in welche Richtung wirkt die Kraft von
   $800\,\text{N}$?
4. Welcher der beiden Blöcke `K[0:2, 6:8]` und `K[2:4, 6:8]` ist null?
5. Führen Sie den Code aus, bauen Sie die Steifigkeitsmatrix mit
   `baue_steifigkeitsmatrix` für Stahl mit $1\,\text{cm}$ Durchmesser auf und
   überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

knoten_pos = np.array([
    [0.0, 0.0],
    [1.5, 0.0],
    [1.5, 1.0],
    [0.0, 1.0],
])
staebe = np.array([[0, 1], [1, 2], [2, 3], [3, 0], [0, 2]])
anzahl_knoten = len(knoten_pos)

kraft_vektor = np.zeros(2 * anzahl_knoten)
kraft_vektor[5] = -800.0

K = baue_steifigkeitsmatrix(knoten_pos, staebe, 2.1e11, np.pi * 0.01**2 / 4)

print('Form von K:', K.shape)
print('staebe[4]:', staebe[4])
differenz = knoten_pos[2] - knoten_pos[0]
print(f'Länge von Stab 4: {np.sqrt(differenz[0]**2 + differenz[1]**2):.4f} m')
print('K[0:2, 6:8] in kN/mm:')
print(np.round(K[0:2, 6:8] * 1e-6, 2))
print('K[2:4, 6:8] in kN/mm:')
print(np.round(K[2:4, 6:8] * 1e-6, 2))
```
Ausgabe:

```text
Form von K: (8, 8)
staebe[4]: [0 2]
Länge von Stab 4: 1.8028 m
K[0:2, 6:8] in kN/mm:
[[  0.     0.  ]
 [  0.   -16.49]]
K[2:4, 6:8] in kN/mm:
[[0. 0.]
 [0. 0.]]
```

Das Fachwerk hat vier Knoten mit je zwei Freiheitsgraden, also acht
Freiheitsgrade, und `K` hat die Form `(8, 8)`. Stab 4 ist die Diagonale von
Knoten 0 nach Knoten 2 mit der Länge $\sqrt{1.5^2 + 1^2} \approx 1.80\,\text{m}$.
Index 5 ist $2 \cdot 2 + 1$, die Kraft wirkt also an Knoten 2 in
$y$-Richtung, wegen des Minuszeichens nach unten. Der Block `K[2:4, 6:8]`
gehört zu den Knoten 1 und 3. Zwischen ihnen gibt es keinen Stab, deshalb ist
er null. Die Knoten 0 und 3 sind dagegen durch den senkrechten Stab 3
verbunden.
````

````{admonition} Aufgabe 4.2 (✩)
:class: tip
Gegeben ist folgender Code für einen senkrechten Stahlstab:

```python
import numpy as np

elastizitaetsmodul = 2.1e11
querschnitt = np.pi * 0.01**2 / 4

knoten_i = np.array([0.0, 0.0])
knoten_j = np.array([0.0, 0.5])

differenz = knoten_j - knoten_i
stablaenge = np.sqrt(differenz[0]**2 + differenz[1]**2)
k = elastizitaetsmodul * querschnitt / stablaenge
e = differenz / stablaenge

block = k * np.array([
    [e[0] * e[0], e[0] * e[1]],
    [e[1] * e[0], e[1] * e[1]],
])
print('e:', e)
print('Block in kN/mm:')
print(np.round(block * 1e-6, 2))
```

Notieren Sie Ihre Vermutung, bevor Sie den Code ausführen.

1. Welchen Wert hat der Einheitsvektor `e`?
2. Welche Einträge des Blocks sind null, welcher nicht?
3. Ein $1\,\text{m}$ langer Stab mit demselben Querschnitt hat die
   Steifigkeit $16.49\,\text{kN/mm}$. Wie groß ist $k$ für diesen Stab?
4. Der obere Knoten verschiebt sich ein kleines Stück waagerecht. Welche
   Kraft entsteht dabei im Stab?
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

elastizitaetsmodul = 2.1e11
querschnitt = np.pi * 0.01**2 / 4

knoten_i = np.array([0.0, 0.0])
knoten_j = np.array([0.0, 0.5])

differenz = knoten_j - knoten_i
stablaenge = np.sqrt(differenz[0]**2 + differenz[1]**2)
k = elastizitaetsmodul * querschnitt / stablaenge
e = differenz / stablaenge

block = k * np.array([
    [e[0] * e[0], e[0] * e[1]],
    [e[1] * e[0], e[1] * e[1]],
])
print('e:', e)
print('Block in kN/mm:')
print(np.round(block * 1e-6, 2))
```
Ausgabe:

```text
e: [0. 1.]
Block in kN/mm:
[[ 0.    0.  ]
 [ 0.   32.99]]
```

Der Stab zeigt senkrecht nach oben, also ist $\vec{e} = (0, 1)^\top$. Im Block
ist nur der Eintrag rechts unten ungleich null. Weil der Stab nur halb so
lang ist wie ein $1\,\text{m}$ langer Stab, ist er doppelt so steif:
$k = 2 \cdot 16.49\,\text{kN/mm} = 32.99\,\text{kN/mm}$. Eine kleine
waagerechte Verschiebung steht senkrecht auf der Stabachse. Sie ändert die
Stablänge nicht, deshalb entsteht keine Kraft. Das zeigen die Nullen in der
ersten Spalte des Blocks.
````

````{admonition} Aufgabe 4.3 (✩)
:class: tip
Gegeben ist folgender Code:

```python
import numpy as np

K = np.array([
    [ 4.0, -2.0],
    [-2.0,  4.0],
])
K_lager = K
K_lager[0, :] = 0.0
K_lager[0, 0] = 1.0

print('K_lager:')
print(K_lager)
print('K:')
print(K)
```

Notieren Sie Ihre Vermutung, bevor Sie den Code ausführen.

1. Was gibt `print(K_lager)` aus?
2. Was gibt `print(K)` aus? Warum?
3. Wie muss die Zeile `K_lager = K` lauten, damit `K` unverändert bleibt?
4. Ein Fachwerk hat ein Festlager an Knoten 1 und ein Loslager an Knoten 3.
   Welche Zeilen der Steifigkeitsmatrix ersetzt `berechne_verschiebungen`?
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

K = np.array([
    [ 4.0, -2.0],
    [-2.0,  4.0],
])
K_lager = K
K_lager[0, :] = 0.0
K_lager[0, 0] = 1.0

print('K_lager:')
print(K_lager)
print('K:')
print(K)
```
Ausgabe:

```text
K_lager:
[[ 1.  0.]
 [-2.  4.]]
K:
[[ 1.  0.]
 [-2.  4.]]
```

Beide Ausgaben sind gleich: In Zeile 0 steht `[1. 0.]`, Zeile 1 ist
unverändert. Die Zuweisung `K_lager = K` legt keine Kopie an. Beide Namen
bezeichnen dasselbe Array, und jede Änderung an `K_lager` ändert auch `K`.
Mit `K_lager = K.copy()` bleibt `K` erhalten. Das ist wichtig, weil wir das
ursprüngliche `K` später für die Lagerkräfte brauchen.

Beim Festlager an Knoten 1 werden beide Zeilen $2 \cdot 1 = 2$ und
$2 \cdot 1 + 1 = 3$ ersetzt, beim Loslager an Knoten 3 nur die Zeile für die
$y$-Richtung, also $2 \cdot 3 + 1 = 7$.
````

````{admonition} Aufgabe 4.4 (✩)
:class: tip
Gegeben ist folgender Code mit den Ergebnissen einer Fachwerkberechnung:

```python
import numpy as np

stabkraefte = np.array([2500.0, -2500.0, -2500.0, -800.0])   # in N
stablaengen = np.array([1.0, 1.0, 2.0, 2.0])                 # in m

elastizitaetsmodul = 2.1e11
durchmesser = 0.014
traegheitsmoment = np.pi * durchmesser**4 / 64

for s in range(len(stabkraefte)):
    if stabkraefte[s] < 0:
        knicklast = np.pi**2 * elastizitaetsmodul * traegheitsmoment / stablaengen[s]**2
        print(f'Stab {s}: Knicklast {knicklast:6.0f} N, Druckkraft {-stabkraefte[s]:6.0f} N')
```

Notieren Sie Ihre Vermutung, bevor Sie den Code ausführen.

1. Für welche Stäbe gibt die Schleife eine Zeile aus? Warum nicht für
   Stab 0?
2. Ein $1\,\text{m}$ langer Stab mit diesem Durchmesser hat eine Knicklast
   von rund $3900\,\text{N}$. Wie groß ist die Knicklast der
   $2\,\text{m}$ langen Stäbe?
3. Welche der Stäbe 1, 2 und 3 knicken?
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

stabkraefte = np.array([2500.0, -2500.0, -2500.0, -800.0])   # in N
stablaengen = np.array([1.0, 1.0, 2.0, 2.0])                 # in m

elastizitaetsmodul = 2.1e11
durchmesser = 0.014
traegheitsmoment = np.pi * durchmesser**4 / 64

for s in range(len(stabkraefte)):
    if stabkraefte[s] < 0:
        knicklast = np.pi**2 * elastizitaetsmodul * traegheitsmoment / stablaengen[s]**2
        print(f'Stab {s}: Knicklast {knicklast:6.0f} N, Druckkraft {-stabkraefte[s]:6.0f} N')
```
Ausgabe:

```text
Stab 1: Knicklast   3908 N, Druckkraft   2500 N
Stab 2: Knicklast    977 N, Druckkraft   2500 N
Stab 3: Knicklast    977 N, Druckkraft    800 N
```

Die Schleife gibt nur für die Druckstäbe 1, 2 und 3 eine Zeile aus. Stab 0
steht unter Zug und kann nicht knicken. Die Knicklast sinkt mit dem Quadrat
der Länge. Der doppelt so lange Stab hat deshalb nur ein Viertel der
Knicklast, rund $980\,\text{N}$. Stab 1 hält, denn seine Knicklast liegt über
der Druckkraft. Stab 2 knickt, obwohl er dieselbe Kraft trägt wie Stab 1,
weil er doppelt so lang ist. Stab 3 ist ebenso lang, trägt aber nur
$800\,\text{N}$ und hält knapp.
````

```{admonition} Aufgabe 4.5 (✩✩)
:class: tip
Wir berechnen die Wandkonsole aus Kapitel 4.2. Die beiden Knoten an der Wand
sind Festlager, an der Spitze hängt eine Rohrleitung mit $3000\,\text{N}$.
Alle Stäbe sind aus Stahl mit $1\,\text{cm}$ Durchmesser.

| Knoten | 0 | 1 | 2 | 3 | 4 |
| --- | --- | --- | --- | --- | --- |
| $x$ in m | 0.0 | 0.0 | 1.0 | 1.0 | 2.0 |
| $y$ in m | 0.0 | 1.0 | 0.0 | 1.0 | 0.0 |

Stabliste: `[[0, 2], [2, 4], [1, 3], [2, 3], [1, 2], [3, 4]]`, Festlager an
den Knoten 0 und 1, Last an Knoten 4 nach unten.

1. Berechnen Sie die Verschiebungen und geben Sie die Verschiebung der
   Spitze in mm aus.
2. Berechnen Sie die Lagerkräfte an den Knoten 0 und 1. Welcher der beiden
   Wandanschlüsse wird auf Zug belastet, also aus der Wand herausgezogen?
3. Berechnen Sie die Stabkräfte. Prüfen Sie die Kräfte in den Stäben 1 und 5
   von Hand mit dem Gleichgewicht an Knoten 4: An diesem Knoten treffen nur
   der waagerechte Stab 1, die Schräge 5 und die Last zusammen.

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
knoten_pos = np.array([
    [0.0, 0.0],
    [0.0, 1.0],
    [1.0, 0.0],
    [1.0, 1.0],
    [2.0, 0.0],
])
staebe = np.array([[0, 2], [2, 4], [1, 3], [2, 3], [1, 2], [3, 4]])
lager_indizes = [0, 1]
kraft_vektor = np.zeros(2 * len(knoten_pos))
kraft_vektor[9] = -3000.0   # Fy an Knoten 4

elastizitaetsmodul = 2.1e11
querschnitt = np.pi * 0.01**2 / 4

# Verarbeitung
K = baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt)
u = berechne_verschiebungen(K, kraft_vektor, lager_indizes)
knotenkraefte = K @ u
stabkraefte = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                   querschnitt, u)

# Ausgabe
print(f'Spitze: ux = {u[8] * 1000:.3f} mm, uy = {u[9] * 1000:.3f} mm')
for n in lager_indizes:
    print(f'Lager an Knoten {n}: Fx = {knotenkraefte[2*n]:7.1f} N, '
          f'Fy = {knotenkraefte[2*n + 1]:7.1f} N')
for s in range(len(staebe)):
    art = 'Zug' if stabkraefte[s] > 0 else 'Druck'
    print(f'Stab {s}: N = {stabkraefte[s]:7.1f} N  ({art})')
```
Ausgabe:

```text
Spitze: ux = -0.546 mm, uy = -2.302 mm
Lager an Knoten 0: Fx =  6000.0 N, Fy =     0.0 N
Lager an Knoten 1: Fx = -6000.0 N, Fy =  3000.0 N
Stab 0: N = -6000.0 N  (Druck)
Stab 1: N = -3000.0 N  (Druck)
Stab 2: N =  3000.0 N  (Zug)
Stab 3: N = -3000.0 N  (Druck)
Stab 4: N =  4242.6 N  (Zug)
Stab 5: N =  4242.6 N  (Zug)
```

Die Spitze senkt sich um rund $2.3\,\text{mm}$ ab und bewegt sich dabei
einen halben Millimeter zur Wand hin. Das untere Lager drückt mit $6000\,\text{N}$
nach rechts gegen die Konsole, das obere zieht sie mit $6000\,\text{N}$ zur
Wand. Der obere Wandanschluss wird also aus der Wand herausgezogen und trägt
zusätzlich die gesamte senkrechte Last. Seine Dübel müssen entsprechend
bemessen sein.

Handprobe an Knoten 4: Die Last von $3000\,\text{N}$ nach unten kann nur die
Schräge 5 aufnehmen, denn Stab 1 ist waagerecht. Die Schräge liegt unter
$45°$, also ist ihre Kraft $3000\,\text{N} \cdot \sqrt{2} \approx
4243\,\text{N}$. Sie zieht Knoten 4 nach links oben, also ist sie ein
Zugstab. Die waagerechte Komponente von $3000\,\text{N}$ muss Stab 1
ausgleichen, indem er Knoten 4 nach rechts drückt: $-3000\,\text{N}$ Druck.
Beides stimmt mit der Rechnung überein.
````

```{admonition} Aufgabe 4.6 (✩✩)
:class: tip
Wie hoch sollte die Spitze des Kranauslegers aus Kapitel 4.1 liegen? Die
Lager bleiben bei $(0, 0)$ und $(2\,\text{m}, 0)$, die Spitze liegt bei
$(1\,\text{m}, h)$. Last $5000\,\text{N}$ nach unten, Stahl mit
$1\,\text{cm}$ Durchmesser.

1. Legen Sie mit `np.linspace` zwölf Höhen von $0.25\,\text{m}$ bis
   $3.0\,\text{m}$ an. Berechnen Sie in einer Schleife für jede Höhe die
   Kraft in Stab 0 und die Absenkung der Spitze.
2. Stellen Sie beide Größen in zwei Subplots über $h$ dar.
3. Bei welcher Höhe ist die Absenkung am kleinsten? Verwenden Sie
   `np.argmin`.
4. Warum wird die Stabkraft für kleine Höhen so groß? Warum wird die
   Absenkung für große Höhen wieder größer, obwohl die Stabkraft weiter sinkt?

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
hoehen = np.linspace(0.25, 3.0, 12)   # in m
staebe = np.array([[0, 1], [1, 2]])
lager_indizes = [0, 2]
kraft_vektor = np.zeros(6)
kraft_vektor[3] = -5000.0
elastizitaetsmodul = 2.1e11
querschnitt = np.pi * 0.01**2 / 4

# Verarbeitung
stabkraft_0 = np.zeros(len(hoehen))
absenkung = np.zeros(len(hoehen))
for m in range(len(hoehen)):
    knoten_pos = np.array([[0.0, 0.0], [1.0, hoehen[m]], [2.0, 0.0]])
    K = baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt)
    u = berechne_verschiebungen(K, kraft_vektor, lager_indizes)
    stabkraefte = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                       querschnitt, u)
    stabkraft_0[m] = stabkraefte[0]
    absenkung[m] = -u[3] * 1000   # in mm, positiv nach unten

m_min = np.argmin(absenkung)

# Ausgabe
print(f'Kleinste Absenkung {absenkung[m_min]:.3f} mm bei h = {hoehen[m_min]:.2f} m')
print(f'Stabkraft bei h = 0.25 m: {stabkraft_0[0]:.0f} N')
print(f'Stabkraft bei h = 3.00 m: {stabkraft_0[-1]:.0f} N')

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(10, 4))
ax1.plot(hoehen, stabkraft_0 / 1000, marker='o')
ax1.set_xlabel('Höhe h in m')
ax1.set_ylabel('Kraft in Stab 0 in kN')
ax1.grid(True)
ax2.plot(hoehen, absenkung, marker='o')
ax2.set_xlabel('Höhe h in m')
ax2.set_ylabel('Absenkung der Spitze in mm')
ax2.grid(True)
plt.tight_layout()
plt.show()
```
Ausgabe:

```text
Kleinste Absenkung 0.395 mm bei h = 1.50 m
Stabkraft bei h = 0.25 m: -10308 N
Stabkraft bei h = 3.00 m: -2635 N
```

Bei flachen Auslegern liegen die Stäbe fast waagerecht. Um die senkrechte
Last von $2500\,\text{N}$ pro Stab zu tragen, braucht ein fast waagerechter
Stab eine riesige Längskraft, denn nur ein kleiner Teil davon zeigt nach
oben. Bei $h = 0.25\,\text{m}$ sind es über $10\,\text{kN}$ Druck pro Stab.
Mit wachsender Höhe nähert sich die Stabkraft der halben Last von
$2500\,\text{N}$. Gleichzeitig
werden die Stäbe aber immer länger und damit weicher, denn $k = EA/L$. Ab
einer bestimmten Höhe überwiegt dieser Effekt, und die Absenkung steigt
wieder. Die kleinste Absenkung liegt in unserem Raster bei
$h = 1.50\,\text{m}$. Eine genauere Rechnung ergibt $h = \sqrt{2}\,\text{m}
\approx 1.41\,\text{m}$.
````

````{admonition} Aufgabe 4.7 (✩✩✩) Mini-Projekt: Wartungssteg in einer Werkhalle
:class: tip
Ein Wartungssteg in einer Werkhalle überspannt $3\,\text{m}$. Er ist als
Fachwerk aus gleichseitigen Dreiecken mit $1\,\text{m}$ Seitenlänge gebaut.
Der Laufsteg liegt auf dem Untergurt. Zwei Monteure mit Werkzeug belasten
die Knoten 1 und 2 mit je $3000\,\text{N}$ nach unten. Links liegt der Steg
auf einem Festlager, rechts auf einem Loslager. Die Höhe des Fachwerks ist
$h = \sqrt{3}/2\,\text{m} \approx 0.866\,\text{m}$.

| Knoten | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| $x$ in m | 0.0 | 1.0 | 2.0 | 3.0 | 0.5 | 1.5 | 2.5 |
| $y$ in m | 0.0 | 0.0 | 0.0 | 0.0 | $h$ | $h$ | $h$ |

Stabliste:
`[[0, 1], [1, 2], [2, 3], [4, 5], [5, 6], [0, 4], [1, 4], [1, 5], [2, 5], [2, 6], [3, 6]]`

Die Stäbe sind Rundstäbe aus S235 mit $E = 2.1 \cdot 10^{11}\,\text{N/m}^2$
und $R_e = 235\,\text{N/mm}^2$.

**Teil 1:** Legen Sie das Fachwerk an und zeichnen Sie es mit Fest- und
Loslager und den Lasten.

**Teil 2:** Berechnen Sie für einen Durchmesser von $12\,\text{mm}$ die
Verschiebungen, die Lagerkräfte und die Stabkräfte. Zeichnen Sie das
Fachwerk mit Stabkräften und 200-fach überhöhten Verschiebungen.

**Teil 3:** Prüfen Sie alle Stäbe auf Spannung und alle Druckstäbe auf
Knicken. Welche Stäbe fallen durch?

**Teil 4:** Rundstäbe gibt es mit 12, 16, 20 und 25 mm Durchmesser. Bestimmen
Sie mit einer Schleife den kleinsten Durchmesser, bei dem alle Stäbe beide
Nachweise bestehen.

**Abschlussfrage:** Die beiden Diagonalen 7 und 8, die vom Untergurt zum
mittleren Knoten 5 führen, tragen keine Kraft. Warum? Und warum ändern sich
die Stabkräfte nicht, wenn wir den Durchmesser ändern?

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
h = np.sqrt(3) / 2
knoten_pos = np.array([
    [0.0, 0.0], [1.0, 0.0], [2.0, 0.0], [3.0, 0.0],   # Untergurt
    [0.5, h], [1.5, h], [2.5, h],                     # Obergurt
])
staebe = np.array([[0, 1], [1, 2], [2, 3], [4, 5], [5, 6], [0, 4],
                   [1, 4], [1, 5], [2, 5], [2, 6], [3, 6]])
lager_indizes = [0]
loslager_indizes = [3]
kraft_vektor = np.zeros(2 * len(knoten_pos))
kraft_vektor[3] = -3000.0   # Fy an Knoten 1
kraft_vektor[5] = -3000.0   # Fy an Knoten 2

elastizitaetsmodul = 2.1e11
streckgrenze = 235.0   # in N/mm²

# Stablängen, für die Knicknachweise
stablaengen = np.zeros(len(staebe))
for s in range(len(staebe)):
    i, j = staebe[s]
    differenz = knoten_pos[j] - knoten_pos[i]
    stablaengen[s] = np.sqrt(differenz[0]**2 + differenz[1]**2)

# Verarbeitung Teil 1: Fachwerk zeichnen
zeichne_fachwerk(knoten_pos, staebe, lager_indizes,
                 loslager_indizes=loslager_indizes, kraft_vektor=kraft_vektor,
                 titel='Wartungssteg')

# Verarbeitung Teil 2: Rechnung für d = 12 mm
durchmesser = 0.012
querschnitt = np.pi * durchmesser**2 / 4
K = baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt)
u = berechne_verschiebungen(K, kraft_vektor, lager_indizes, loslager_indizes)
knotenkraefte = K @ u
stabkraefte = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                   querschnitt, u)

print(f'Größte Absenkung: {-np.min(u[1::2]) * 1000:.3f} mm')
print(f'Lager 0: Fx = {knotenkraefte[0]:.1f} N, Fy = {knotenkraefte[1]:.1f} N')
print(f'Lager 3: Fy = {knotenkraefte[7]:.1f} N')
print('Stabkräfte in N:', np.round(stabkraefte, 1))
zeichne_fachwerk(knoten_pos, staebe, lager_indizes,
                 loslager_indizes=loslager_indizes, kraft_vektor=kraft_vektor,
                 verschiebung=u, skalierung=200, stabkraefte=stabkraefte,
                 titel='Wartungssteg, d = 12 mm, 200-fach überhöht')

# Verarbeitung Teil 3: Nachweise für d = 12 mm
traegheitsmoment = np.pi * durchmesser**4 / 64
for s in range(len(staebe)):
    spannung = stabkraefte[s] / querschnitt * 1e-6
    if abs(spannung) > streckgrenze:
        print(f'Stab {s}: Spannungsnachweis nicht erfüllt')
    if stabkraefte[s] < 0:
        knicklast = np.pi**2 * elastizitaetsmodul * traegheitsmoment / stablaengen[s]**2
        if -stabkraefte[s] > knicklast:
            print(f'Stab {s}: knickt ({-stabkraefte[s]:.0f} N > {knicklast:.0f} N)')

# Verarbeitung Teil 4: kleinster Durchmesser aus der Reihe
for durchmesser in [0.012, 0.016, 0.020, 0.025]:
    querschnitt = np.pi * durchmesser**2 / 4
    traegheitsmoment = np.pi * durchmesser**4 / 64
    K = baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt)
    u = berechne_verschiebungen(K, kraft_vektor, lager_indizes, loslager_indizes)
    stabkraefte = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                       querschnitt, u)
    anzahl_versagt = 0
    for s in range(len(staebe)):
        spannung = stabkraefte[s] / querschnitt * 1e-6
        knicklast = np.pi**2 * elastizitaetsmodul * traegheitsmoment / stablaengen[s]**2
        if abs(spannung) > streckgrenze:
            anzahl_versagt = anzahl_versagt + 1
        elif stabkraefte[s] < 0 and -stabkraefte[s] > knicklast:
            anzahl_versagt = anzahl_versagt + 1
    print(f'd = {durchmesser * 1000:.0f} mm: {anzahl_versagt} Stäbe versagen')
```
Ausgabe:

```text
Größte Absenkung: 0.674 mm
Lager 0: Fx = 0.0 N, Fy = 3000.0 N
Lager 3: Fy = 3000.0 N
Stabkräfte in N: [ 1732.1  3464.1  1732.1 -3464.1 -3464.1 -3464.1  3464.1    -0.     -0.
  3464.1 -3464.1]
Stab 3: knickt (3464 N > 2110 N)
Stab 4: knickt (3464 N > 2110 N)
Stab 5: knickt (3464 N > 2110 N)
Stab 10: knickt (3464 N > 2110 N)
d = 12 mm: 4 Stäbe versagen
d = 16 mm: 0 Stäbe versagen
d = 20 mm: 0 Stäbe versagen
d = 25 mm: 0 Stäbe versagen
```

Die Lager tragen je $3000\,\text{N}$, die waagerechte Kraft am Festlager ist
null. Der Untergurt steht unter Zug, der Obergurt und die äußeren Diagonalen
unter Druck. Mit $12\,\text{mm}$ Durchmesser liegt die Spannung weit unter der
Streckgrenze, aber die vier Druckstäbe 3, 4, 5 und 10 knicken. Ihre Knicklast
beträgt nur rund $2110\,\text{N}$ bei einer Druckkraft von $3464\,\text{N}$.
Der kleinste Durchmesser, bei dem alle Nachweise erfüllt sind, ist
$16\,\text{mm}$. Weil die Knicklast mit $d^4$ wächst, reicht schon dieser
kleine Schritt: $(16/12)^4 \approx 3.2$.

**Abschlussfrage:** Am mittleren Knoten 5 greift keine Last an. Die beiden
Obergurtstäbe 3 und 4 liegen dort auf einer waagerechten Linie. Eine Kraft in
einer der beiden Diagonalen hätte eine senkrechte Komponente, die an Knoten 5
niemand ausgleichen könnte. Deshalb müssen beide Diagonalen kraftlos sein.
Aus der Technischen Mechanik kennen Sie solche Stäbe als Nullstäbe. Die
Ausgabe `-0.` ist dabei nur ein Rundungsfehler, im Plot sind die beiden
Stäbe grau.

Die Stabkräfte hängen nicht vom Durchmesser ab, weil das Fachwerk statisch
bestimmt ist: Die Gleichgewichtsbedingungen an den Knoten legen alle Kräfte
eindeutig fest, genau wie beim Kranausleger in Kapitel 4.3. Der Durchmesser
ändert nur, wie stark sich das Fachwerk dabei verformt.
````
