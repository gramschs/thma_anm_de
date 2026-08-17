---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.6 Übung: Warren-Fachwerkträger mit Loslager

```{admonition} Warnung
:class: warning
Dieses Kapitel befindet sich derzeit im Umbau und wird rechtzeitig vor der Vorlesung im WiSe 2026/27 zur Verfügung stehen.
```

In den bisherigen Übungen wurden alle Lagerknoten als **Festlager** behandelt:
beide Freiheitsgrade $u_x$ und $u_y$ wurden gesperrt. In Kapitel 4.2 wurde
bereits erwähnt, dass in der Praxis häufig ein **Loslager** (Rollenlager)
eingesetzt wird, das nur einen Freiheitsgrad sperrt. Diese Übung führt
das Konzept ein und wendet es auf einen Warren-Fachwerkträger an, eine
klassische Brückenform mit gleichseitigen Dreiecken.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können den Unterschied zwischen Festlager und Loslager benennen
  und erklären, wie das Loslager die Lagerkräfte beeinflusst.
* [ ] Sie können die bisherige `lager_indizes`-Logik durch eine allgemeinere
  Liste `gesperrte_dofs` ersetzen und damit Loslager modellieren.
* [ ] Sie können das FEM-Schema auf einen Warren-Fachwerkträger anwenden
  und die Ergebnisse physikalisch interpretieren.
```

---

## Das Loslager: Konzept und Umsetzung

### Physikalischer Hintergrund

Ein **Festlager** (Gelenklager, engl. *pin support*) fixiert einen Knoten
vollständig: weder $u_x$ noch $u_y$ sind erlaubt. Es liefert zwei
Lagerreaktionen ($F_x$ und $F_y$).

Ein **Loslager** (Rollenlager, engl. *roller support*) erlaubt eine
Verschiebung entlang einer Richtung (typischerweise $x$) und sperrt nur die
Querrichtung ($u_y = 0$). Es liefert genau eine Lagerreaktion ($F_y$).

In der Praxis wird bei Brückenträgern das eine Auflager als Festlager, das
andere als Loslager ausgeführt. Das Loslager ermöglicht die Längenänderung
des Trägers infolge von Temperatureinflüssen oder Durchbiegung, ohne dass
Zwangskräfte entstehen.

### Umsetzung im Code

In den Kapiteln 4.1 bis 4.5 wurden die gesperrten Freiheitsgrade indirekt
über `lager_indizes` bestimmt: für jeden Lagerknoten $n$ wurden beide
DOF-Indizes $2n$ (x-Richtung) und $2n+1$ (y-Richtung) automatisch als
gesperrt behandelt.

Für ein Loslager reicht diese Darstellung nicht aus. Stattdessen verwenden wir
eine explizite Liste `gesperrte_dofs`, die genau angibt, welche DOF-Indizes
gesperrt sind:

```{code-cell} python
# Beispiel (nicht ausführen, nur zur Illustration)
#
# Festlager bei Knoten 0: DOFs 0 (u_x) und 1 (u_y) gesperrt
# Loslager  bei Knoten 3: nur DOF  7 (u_y, denn 2*3+1 = 7) gesperrt
#
# gesperrte_dofs = [0, 1, 7]
#
# freie_dofs = np.array([i for i in range(2 * anzahl_knoten)
#                        if i not in gesperrte_dofs])
```

Der Rest des Algorithmus (Steifigkeitsmatrix, LGS reduzieren, lösen,
Lagerkräfte, Stabkräfte) bleibt vollständig unverändert.

```{admonition} Wann ist das System lösbar?
:class: note
Die globale Steifigkeitsmatrix ist ohne Lager singulär, weil das Fachwerk als
Starrkörper verschoben werden könnte. Jedes gesperrte DOF reduziert die Anzahl
der freien Unbekannten um eins. Für ein statisch bestimmtes ebenes Fachwerk
mit $k$ Knoten und $m$ Stäben muss gelten: $m + r = 2k$, wobei $r$ die Anzahl
der Lagerreaktionen ist. Ein Festlager liefert $r = 2$, ein Loslager $r = 1$.
```

---

## Problemstellung

Betrachten Sie einen **Warren-Fachwerkträger** mit sieben Knoten und elf
Stäben. Die Geometrie besteht aus gleichseitigen Dreiecken mit der
Seitenlänge $1\,\text{m}$. Der Abstand zwischen Unter- und Obergurt beträgt
$h = \frac{\sqrt{3}}{2}\,\text{m} \approx 0{,}866\,\text{m}$.

Die vier Untergurtknoten liegen auf $y = 0$, die drei Obergurtknoten auf
$y = h$:

| Knoten | $x$ in m | $y$ in m | Bemerkung |
| --- | --- | --- | --- |
| 0 | 0.000 | 0.000 | Festlager |
| 1 | 1.000 | 0.000 | frei |
| 2 | 2.000 | 0.000 | frei |
| 3 | 3.000 | 0.000 | Loslager (nur $u_y = 0$) |
| 4 | 0.500 | $h$ | frei |
| 5 | 1.500 | $h$ | frei |
| 6 | 2.500 | $h$ | frei |

Die elf Stäbe verteilen sich auf drei Gruppen:

| Gruppe | Stäbe |
| --- | --- |
| Untergurt | 0-1, 1-2, 2-3 |
| Obergurt | 4-5, 5-6 |
| Diagonalen | 0-4, 1-4, 1-5, 2-5, 2-6, 3-6 |

An Knoten 5 (Obergurt, Mitte) greift eine Vertikallast von
$6000\,\text{N}$ nach unten an. Die Materialeigenschaften sind dieselben wie
in den Kapiteln 4.1 bis 4.5: Stahl mit $E = 2{,}1 \cdot 10^{11}\,\text{N/m}^2$,
Stabdurchmesser $d = 1\,\text{cm}$.

Skizzieren Sie das Fachwerk auf Papier. Überlegen Sie, welche Lagerreaktionen
Sie aufgrund der Geometrie und der Laststellung qualitativ erwarten.

---

## Aufgabe 1: Grundsetup und Visualisierung

Legen Sie die Materialeigenschaften, die Knotenkoordinaten, die
Konnektivitätsmatrix und den Kraftvektor an. Definieren Sie `gesperrte_dofs`
für Festlager bei Knoten 0 und Loslager bei Knoten 3. Stellen Sie das Fachwerk
mit der bereitgestellten Funktion `zeichne_fachwerk` dar.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Hier Ihren Code eingeben
```

Die folgende Funktion ist fertig vorgegeben und darf nicht verändert werden.
Sie unterscheidet Fest- und Loslager durch verschiedene Symbole: Festlager
werden als ausgefüllte Dreiecke, Loslager als hohle Dreiecke mit einer
Rollenlinie dargestellt.

```{code-cell} python
def zeichne_fachwerk(knoten_pos, verbindung, lager_indizes,
                     loslager_indizes=None,
                     verschiebung=None, skalierung=500,
                     stabkraefte=None, titel=''):
    """Zeichnet einen Warren-Fachwerkträger mit Fest- und Loslagern.

    Parameters
    ----------
    knoten_pos : ndarray, Form (n, 2)
    verbindung : ndarray, Form (n, n)
    lager_indizes : list
        Indizes der Festlager.
    loslager_indizes : list oder None
        Indizes der Loslager (Rollenlager).
    verschiebung : ndarray oder None
    skalierung : float
    stabkraefte : dict oder None
        {(i, j): Stabkraft in N}.
    titel : str
    """
    n = knoten_pos.shape[0]
    if loslager_indizes is None:
        loslager_indizes = []
    if verschiebung is None:
        verschiebung = np.zeros(2 * n)
    if stabkraefte is None:
        stabkraefte = {}

    fig, ax = plt.subplots(figsize=(10, 5))
    knoten_verformt = knoten_pos + skalierung * verschiebung.reshape((n, 2))

    # Stäbe
    for i in range(n):
        for j in range(i + 1, n):
            if verbindung[i, j]:
                ax.plot([knoten_pos[i, 0],      knoten_pos[j, 0]],
                        [knoten_pos[i, 1],      knoten_pos[j, 1]],
                        color='gray', linewidth=1.5, alpha=0.3)
                F = stabkraefte.get((i, j), 0)
                farbe = 'tab:red' if (stabkraefte and F < 0) else 'tab:blue'
                ax.plot([knoten_verformt[i, 0], knoten_verformt[j, 0]],
                        [knoten_verformt[i, 1], knoten_verformt[j, 1]],
                        color=farbe, linewidth=2.5)
                if stabkraefte:
                    mx = 0.5 * (knoten_verformt[i, 0] + knoten_verformt[j, 0])
                    my = 0.5 * (knoten_verformt[i, 1] + knoten_verformt[j, 1])
                    ax.text(mx, my + 0.07, f'{F / 1000:.2f} kN',
                            fontsize=7, ha='center', color=farbe)

    # Knoten
    ax.scatter(knoten_pos[:, 0], knoten_pos[:, 1],
               c='gray', s=60, zorder=4, alpha=0.3)
    ax.scatter(knoten_verformt[:, 0], knoten_verformt[:, 1],
               c='tab:red', s=80, zorder=5)
    for k in range(n):
        ax.text(knoten_verformt[k, 0] + 0.04,
                knoten_verformt[k, 1] + 0.04,
                f'K{k}', fontsize=9)

    # Festlager: ausgefülltes Dreieck
    h_d, b_d = 0.10, 0.10
    for k in lager_indizes:
        xd = [knoten_verformt[k, 0],
              knoten_verformt[k, 0] - b_d / 2,
              knoten_verformt[k, 0] + b_d / 2]
        yd = [knoten_verformt[k, 1],
              knoten_verformt[k, 1] - h_d,
              knoten_verformt[k, 1] - h_d]
        ax.fill(xd, yd, color='tab:green', alpha=0.7)

    # Loslager: hohles Dreieck mit Rollenlinie
    for k in loslager_indizes:
        xd = [knoten_verformt[k, 0],
              knoten_verformt[k, 0] - b_d / 2,
              knoten_verformt[k, 0] + b_d / 2,
              knoten_verformt[k, 0]]
        yd = [knoten_verformt[k, 1],
              knoten_verformt[k, 1] - h_d,
              knoten_verformt[k, 1] - h_d,
              knoten_verformt[k, 1]]
        ax.plot(xd, yd, color='tab:green', linewidth=1.5)
        ax.plot([knoten_verformt[k, 0] - b_d / 2 - 0.03,
                 knoten_verformt[k, 0] + b_d / 2 + 0.03],
                [knoten_verformt[k, 1] - h_d - 0.02,
                 knoten_verformt[k, 1] - h_d - 0.02],
                color='tab:green', linewidth=1.5)

    if stabkraefte:
        ax.plot([], [], color='tab:blue', linewidth=3, label='Zug')
        ax.plot([], [], color='tab:red',  linewidth=3, label='Druck')
        ax.legend(fontsize=9, loc='upper right')

    ax.set_title(titel)
    ax.set_aspect('equal')
    ax.grid(True)
    plt.tight_layout()
    plt.show()
```

````{admonition} Lösung zu Aufgabe 1
:class: tip
:class: dropdown
```python
import numpy as np
import matplotlib.pyplot as plt

# Materialeigenschaften
elastizitaetsmodul = 2.1e11
durchmesser        = 1.0e-2
querschnitt        = np.pi * 0.25 * durchmesser**2

# Knotenkoordinaten
h = np.sqrt(3) / 2   # Höhe eines gleichseitigen Dreiecks mit Seitenlänge 1 m

knoten_pos = np.array([
    [0.0, 0.0],   # Knoten 0: Festlager
    [1.0, 0.0],   # Knoten 1: frei
    [2.0, 0.0],   # Knoten 2: frei
    [3.0, 0.0],   # Knoten 3: Loslager
    [0.5, h  ],   # Knoten 4: frei
    [1.5, h  ],   # Knoten 5: frei, Last greift hier an
    [2.5, h  ],   # Knoten 6: frei
])
anzahl_knoten = knoten_pos.shape[0]

# Konnektivitätsmatrix
verbindung = np.zeros((anzahl_knoten, anzahl_knoten))
for i, j in [(0, 1), (1, 2), (2, 3),             # Untergurt
             (4, 5), (5, 6),                       # Obergurt
             (0, 4), (1, 4), (1, 5), (2, 5),      # Diagonalen links/mitte
             (2, 6), (3, 6)]:                      # Diagonalen rechts
    verbindung[i, j] = 1
verbindung = verbindung + verbindung.T

# Kraftvektor
kraft_knoten = np.zeros((anzahl_knoten, 2))
kraft_knoten[5, 1] = -6000.   # 6000 N nach unten an Knoten 5
kraft_vektor = kraft_knoten.flatten()

# Gesperrte Freiheitsgrade
# Festlager Knoten 0: DOFs 0 (u_x) und 1 (u_y)
# Loslager  Knoten 3: DOF  7 (u_y, denn 2*3+1 = 7)
gesperrte_dofs = [0, 1, 7]
freie_dofs = np.array([i for i in range(2 * anzahl_knoten)
                        if i not in gesperrte_dofs])

print("Anzahl Knoten:", anzahl_knoten)
print("Anzahl Stäbe:", int(verbindung.sum() / 2))
print("Gesperrte DOFs:", gesperrte_dofs)
print("Freie DOFs:    ", freie_dofs.tolist())

zeichne_fachwerk(knoten_pos, verbindung,
                 lager_indizes=[0],
                 loslager_indizes=[3],
                 titel='Warren-Fachwerkträger in Ausgangslage')
```
````

---

## Aufgabe 2: Steifigkeitsmatrix aufbauen und LGS lösen

Bauen Sie die globale Steifigkeitsmatrix auf. Verwenden Sie `gesperrte_dofs`
aus Aufgabe 1, um `freie_dofs` zu bestimmen. Reduzieren Sie das LGS und
lösen Sie es. Geben Sie den vollständigen Verschiebungsvektor knotenweise aus.

```{admonition} Hinweis
:class: tip
Die Steifigkeitsmatrix und das Assemblierungsschema sind identisch mit den
Kapiteln 4.1 bis 4.5. Nur die Bestimmung von `freie_dofs` ändert sich.
```

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung zu Aufgabe 2
:class: tip
:class: dropdown
```python
# Globale Steifigkeitsmatrix
steifigkeit_global = np.zeros((2 * anzahl_knoten, 2 * anzahl_knoten))

for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            differenz       = knoten_pos[j] - knoten_pos[i]
            stablaenge      = np.linalg.norm(differenz)
            winkel          = np.arctan2(differenz[1], differenz[0])
            stabsteifigkeit = elastizitaetsmodul * querschnitt / stablaenge
            cos_w = np.cos(winkel)
            sin_w = np.sin(winkel)
            k_element = stabsteifigkeit * np.array([
                [cos_w**2,       sin_w * cos_w],
                [sin_w * cos_w,  sin_w**2     ],
            ])
            steifigkeit_global[2*i : 2*(i+1), 2*i : 2*(i+1)] += k_element
            steifigkeit_global[2*j : 2*(j+1), 2*j : 2*(j+1)] += k_element
            steifigkeit_global[2*i : 2*(i+1), 2*j : 2*(j+1)] -= k_element
            steifigkeit_global[2*j : 2*(j+1), 2*i : 2*(i+1)] -= k_element

# Reduziertes System lösen
steifigkeit_reduziert  = steifigkeit_global[freie_dofs, :][:, freie_dofs]
kraft_reduziert        = kraft_vektor[freie_dofs]
verschiebung_reduziert = np.linalg.solve(steifigkeit_reduziert,
                                          kraft_reduziert)

# Vollständiger Verschiebungsvektor
verschiebung_gesamt = np.zeros(2 * anzahl_knoten)
verschiebung_gesamt[freie_dofs] = verschiebung_reduziert

print("Verschiebungen (in mm):")
for n in range(anzahl_knoten):
    ux = verschiebung_gesamt[2 * n]     * 1e3
    uy = verschiebung_gesamt[2 * n + 1] * 1e3
    print(f"  Knoten {n}: ux = {ux:10.4f} mm,  uy = {uy:10.4f} mm")
```
````

---

## Aufgabe 3: Lagerkräfte und Gleichgewichtsprüfung

Berechnen Sie die Lagerkräfte über $\vec{F} = \mathbf{K} \cdot \vec{u}$.
Prüfen Sie das Gleichgewicht mit `np.allclose`. Beantworten Sie außerdem
die folgenden Fragen, bevor Sie den Code ausführen:

1. Welche Vorzeichen erwarten Sie für $F_{y}$ an Knoten 0 und Knoten 3?
2. Warum ist die horizontale Lagerreaktion $F_{x}$ an Knoten 3 gleich null?
3. Warum sind die vertikalen Lagerreaktionen bei symmetrischer Last und
   symmetrischer Geometrie gleich groß?

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung zu Aufgabe 3
:class: tip
:class: dropdown
```python
reaktion = steifigkeit_global @ verschiebung_gesamt

probe = np.allclose(reaktion[freie_dofs], kraft_vektor[freie_dofs])
print(f"Gleichgewichtsprobe bestanden: {probe}")

print("\nKnotenweise Zusammenfassung:")
print(f"{'Knoten':>6}  {'ux (mm)':>10}  {'uy (mm)':>10}  "
      f"{'Fx (N)':>10}  {'Fy (N)':>10}  {'Typ':>12}")
print("-" * 72)
for n in range(anzahl_knoten):
    ux  = verschiebung_gesamt[2 * n]     * 1e3
    uy  = verschiebung_gesamt[2 * n + 1] * 1e3
    fx  = reaktion[2 * n]
    fy  = reaktion[2 * n + 1]
    if n == 0:
        typ = "Festlager"
    elif n == 3:
        typ = "Loslager"
    else:
        typ = "frei"
    print(f"{n:>6}  {ux:>10.4f}  {uy:>10.4f}  "
          f"{fx:>10.2f}  {fy:>10.2f}  {typ:>12}")

summe_fx = np.sum(reaktion[0::2])
summe_fy = np.sum(reaktion[1::2])
print(f"\nSumme aller Kräfte: Fx = {summe_fx:.4f} N,  Fy = {summe_fy:.4f} N")
```

Zu den Verständnisfragen:

1. Beide vertikalen Lagerreaktionen sind positiv (nach oben), weil sie die
   nach unten gerichtete Last tragen.
2. $F_x$ an Knoten 3 ist null, weil das Loslager keine horizontale Reaktion
   liefern kann: DOF 6 ($u_x$ von Knoten 3) ist frei. Aus der Gleichgewichts-
   bedingung folgt, dass die gesamte horizontale Reaktion bei Knoten 0 liegt.
   Bei symmetrischer Last (und damit $F_{x,\text{gesamt}} = 0$) ist auch
   $F_x$ bei Knoten 0 gleich null.
3. Die Geometrie ist symmetrisch ($x = 1{,}5\,\text{m}$ als Spiegelachse),
   die Last greift im Symmetriezentrum an (Knoten 5). Beide Auflager tragen
   daher gleich viel: $F_{y,0} = F_{y,3} = 3000\,\text{N}$.
````

---

## Aufgabe 4: Stabkräfte und Visualisierung

Berechnen Sie die Stabkräfte und stellen Sie die verformte Lage mit
farblich kodierten Stabkräften dar. Identifizieren Sie den am stärksten
auf Zug und den am stärksten auf Druck beanspruchten Stab.

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung zu Aufgabe 4
:class: tip
:class: dropdown
```python
stabkraefte = {}

print("Stabkräfte (positiv = Zug, negativ = Druck):")
print(f"{'Stab':>6}  {'Länge (m)':>10}  {'Kraft (N)':>12}  {'Typ':>8}")
print("-" * 44)

for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            differenz       = knoten_pos[j] - knoten_pos[i]
            stablaenge      = np.linalg.norm(differenz)
            winkel          = np.arctan2(differenz[1], differenz[0])
            stabsteifigkeit = elastizitaetsmodul * querschnitt / stablaenge
            einheitsvektor  = np.array([np.cos(winkel), np.sin(winkel)])
            u_i = verschiebung_gesamt[2 * i : 2 * (i + 1)]
            u_j = verschiebung_gesamt[2 * j : 2 * (j + 1)]
            u_parallel = np.dot(einheitsvektor, u_j - u_i)
            F = stabsteifigkeit * u_parallel
            stabkraefte[(i, j)] = F
            typ = 'Zug' if F > 0 else 'Druck'
            print(f"  {i}-{j}   {stablaenge:>8.3f} m  {F:>12.2f} N  {typ:>8}")

# Maximal belastete Stäbe
max_zug   = max(stabkraefte, key=lambda k: stabkraefte[k])
max_druck = min(stabkraefte, key=lambda k: stabkraefte[k])
print(f"\nGrößte Zugkraft:    Stab {max_zug},  "
      f"F = {stabkraefte[max_zug]:.2f} N")
print(f"Größte Druckkraft:  Stab {max_druck},  "
      f"F = {stabkraefte[max_druck]:.2f} N")

zeichne_fachwerk(knoten_pos, verbindung,
                 lager_indizes=[0],
                 loslager_indizes=[3],
                 verschiebung=verschiebung_gesamt,
                 skalierung=500,
                 stabkraefte=stabkraefte,
                 titel='Warren-Fachwerkträger: Stabkräfte (blau=Zug, rot=Druck)')
```
````

---

## Aufgabe 5: Vergleich mit Festlager

```{admonition} Hinweis zur Notebook-Reihenfolge
:class: warning
Arbeiten Sie in dieser Aufgabe in neuen Code-Zellen, ohne die Originalwerte
zu überschreiben.
```

Ersetzen Sie das Loslager bei Knoten 3 durch ein Festlager, indem Sie
`gesperrte_dofs = [0, 1, 6, 7]` setzen (beide DOFs von Knoten 3 gesperrt).
Berechnen Sie das System neu und vergleichen Sie:

1. Die Verschiebung $u_x$ von Knoten 3: Warum ist sie beim Festlager gleich
   null, beim Loslager aber ungleich null?
2. Die horizontale Lagerreaktion $F_x$ bei Knoten 3: Warum entsteht beim
   Festlager eine horizontale Zwangskraft, beim Loslager jedoch nicht?
3. Die Stabkräfte: Ändern sich die Werte? Begründen Sie das Ergebnis.

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung zu Aufgabe 5
:class: tip
:class: dropdown
```python
# Festlager bei beiden Auflagern
gesperrte_dofs_fest = [0, 1, 6, 7]
freie_dofs_fest = np.array([i for i in range(2 * anzahl_knoten)
                             if i not in gesperrte_dofs_fest])

K_red_fest = steifigkeit_global[freie_dofs_fest, :][:, freie_dofs_fest]
F_red_fest = kraft_vektor[freie_dofs_fest]
u_red_fest = np.linalg.solve(K_red_fest, F_red_fest)

u_fest = np.zeros(2 * anzahl_knoten)
u_fest[freie_dofs_fest] = u_red_fest

reaktion_fest = steifigkeit_global @ u_fest

print("Vergleich Knoten 3 (Loslager vs. Festlager):")
print(f"  u_x Knoten 3 (Loslager):  "
      f"{verschiebung_gesamt[6] * 1e3:10.4f} mm")
print(f"  u_x Knoten 3 (Festlager): "
      f"{u_fest[6] * 1e3:10.4f} mm")
print()
print(f"  F_x Knoten 3 (Loslager):  {reaktion[6]:10.2f} N")
print(f"  F_x Knoten 3 (Festlager): {reaktion_fest[6]:10.2f} N")
print()

# Stabkräfte beim Festlager
stabkraefte_fest = {}
for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            differenz       = knoten_pos[j] - knoten_pos[i]
            stablaenge      = np.linalg.norm(differenz)
            winkel          = np.arctan2(differenz[1], differenz[0])
            stabsteifigkeit = elastizitaetsmodul * querschnitt / stablaenge
            einheitsvektor  = np.array([np.cos(winkel), np.sin(winkel)])
            u_i = u_fest[2 * i : 2 * (i + 1)]
            u_j = u_fest[2 * j : 2 * (j + 1)]
            stabkraefte_fest[(i, j)] = (stabsteifigkeit
                                         * np.dot(einheitsvektor, u_j - u_i))

print("Stabkräfte Loslager vs. Festlager:")
print(f"{'Stab':>6}  {'Loslager (N)':>14}  {'Festlager (N)':>14}")
print("-" * 40)
for key in stabkraefte:
    print(f"  {key[0]}-{key[1]}   "
          f"{stabkraefte[key]:>14.2f}  "
          f"{stabkraefte_fest[key]:>14.2f}")
```

Zu den Verständnisfragen:

1. Beim Loslager ist $u_x$ von Knoten 3 frei; der Träger dehnt sich unter
   Last leicht horizontal aus. Beim Festlager wird diese Dehnung verhindert:
   $u_x = 0$ wird erzwungen.
2. Beim Festlager entsteht eine horizontale Zwangskraft, weil das Lager die
   erzwungene Verschiebungsunterdrückung durch eine Reaktionskraft kompensiert.
   Beim Loslager gibt es keine horizontale Einspannung: $F_x = 0$ folgt
   direkt aus der fehlenden Sperrung.
3. Bei symmetrischer Last auf einem symmetrischen Träger sind die
   Stabkräfte beim Loslager und beim Festlager gleich: Die zusätzliche
   Sperrung von $u_x$ am Festlager ändert nichts, weil die symmetrische
   Last ohnehin keine horizontale Verschiebung erzeugt.
````

---

## Aufgabe 6: Vertiefende Aufgaben

```{admonition} Vertiefung
:class: tip
1. **Asymmetrische Last:** Verschieben Sie die Last von Knoten 5 auf
   Knoten 4 (ebenfalls $6000\,\text{N}$ nach unten). Berechnen Sie die
   Lagerkräfte neu. Ist $F_x$ bei Knoten 0 nun null? Warum nicht?
   Warum unterscheiden sich die vertikalen Lagerreaktionen jetzt?

2. **Einfluss des Loslagerwinkels:** In dieser Übung lässt das Loslager
   die $x$-Richtung frei und sperrt $y$. Wie müsste `gesperrte_dofs`
   aussehen, wenn das Loslager stattdessen die $y$-Richtung freilässt
   und $x$ sperrt? In welchen Anwendungsfällen wäre das sinnvoll?

3. **Zusätzlicher Stab:** Fügen Sie einen Stab zwischen den Knoten 4 und
   6 ein (Obergurt-Querverbindung). Berechnen Sie Steifigkeitsmatrix,
   Verschiebungen und Stabkräfte neu. Ändert sich die maximale Absenkung
   von Knoten 5?
```

```{code-cell} python
# Hier Ihren Code für die Vertiefungsaufgaben eingeben
```

````{admonition} Lösung zu Aufgabe 6 (Teilaufgabe 1)
:class: tip
:class: dropdown
```python
# Asymmetrische Last: 6000 N an Knoten 4 statt Knoten 5
kraft_knoten_asym = np.zeros((anzahl_knoten, 2))
kraft_knoten_asym[4, 1] = -6000.
kraft_vektor_asym = kraft_knoten_asym.flatten()

K_red_asym = steifigkeit_global[freie_dofs, :][:, freie_dofs]
u_red_asym = np.linalg.solve(K_red_asym, kraft_vektor_asym[freie_dofs])
u_asym     = np.zeros(2 * anzahl_knoten)
u_asym[freie_dofs] = u_red_asym

reaktion_asym = steifigkeit_global @ u_asym

print("Lagerreaktionen bei asymmetrischer Last (Knoten 4):")
for n in [0, 3]:
    fx = reaktion_asym[2 * n]
    fy = reaktion_asym[2 * n + 1]
    print(f"  Knoten {n}: Fx = {fx:.2f} N,  Fy = {fy:.2f} N")
```

Die Last bei Knoten 4 liegt nicht auf der Symmetrieachse. Daher ist die
Verteilung auf die Auflager nicht mehr gleichmäßig: das nähere Auflager
(Knoten 0) trägt mehr. Außerdem erzeugt die exzentrische Last eine
horizontale Komponente, die als $F_x$ am Festlager bei Knoten 0 auftritt.
Das Loslager bei Knoten 3 kann diese Reaktion nicht übernehmen.
````

---

## Zusammenfassung

In dieser Übung haben Sie das FEM-Schema für ebene Fachwerke um das Loslager
erweitert. Die zentrale Änderung besteht darin, die implizite Variable
`lager_indizes` durch die explizite Liste `gesperrte_dofs` zu ersetzen: sie
enthält genau die DOF-Indizes, die auf null gesetzt werden. Für ein Festlager
bei Knoten $n$ sind das die Einträge $2n$ und $2n+1$, für ein Loslager
(Querrichtung gesperrt) nur der Eintrag $2n+1$. Der gesamte übrige
Algorithmus bleibt unverändert.

Am Warren-Fachwerkträger zeigt sich die physikalische Bedeutung: Das
Loslager lässt die thermische und lastbedingte Längenänderung des Trägers
zu, verhindert damit Zwangskräfte und liefert eine Lagerreaktion
ausschließlich in der gesperrten Richtung.
