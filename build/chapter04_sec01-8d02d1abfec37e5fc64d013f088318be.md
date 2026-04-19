---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.1 Fachwerke: Knoten, Stäbe und Kräfte

In Kapitel 3 haben wir lineare Gleichungssysteme gelöst, deren Koeffizienten
aus der Physik stammten: Wärmewiderstände, Einkaufsmengen, Kalibrierungsfaktoren.
In diesem Kapitel kommen die Koeffizienten aus der Strukturmechanik. *Wie stark
biegt sich ein Fachwerkträger unter Last durch, und welche Kräfte entstehen
in den einzelnen Stäben?*

Ein **ideales ebenes Fachwerk** besteht aus geraden Stäben, die an Gelenken
verbunden sind. An den Gelenken greifen Kräfte an, und bestimmte Gelenke
sind fest im Raum verankert. Wir nennen die Gelenke **Knoten**. Jeder Knoten
kann sich in $x$- und $y$-Richtung verschieben, hat also zwei **Freiheitsgrade**.
Das Ziel ist, diese Verschiebungen zu berechnen.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können ein ebenes Fachwerk durch Knotenkoordinaten, eine
  Konnektivitätsmatrix und einen Kraftvektor beschreiben.
* [ ] Sie wissen, was ein **Freiheitsgrad** ist und wie er sich im
  Kraft- und Verschiebungsvektor widerspiegelt.
* [ ] Sie können die Konnektivitätsmatrix für ein gegebenes Fachwerk
  aufstellen und ablesen, welche Stäbe existieren.
```

## Das Fachwerk beschreiben: Knoten und Stäbe

Als Beispiel betrachten wir einen Kranausleger: zwei Stäbe treffen in einem
freien Knoten zusammen, an dem eine Last hängt. Die beiden Endknoten sind
fest gelagert.

```{figure} pics/chap04_lastkran.svg

Darstellung des im Beispiel verwendeten ebenen Dreiknoten-Fachwerks
(Kranausleger) mit beidseitiger Lagerung und punktförmiger Last am mittleren
Knoten. (Quelle: eigene Abbildung; Lizenz [CC BY-SA
4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

Bevor wir losrechnen, legen wir fest, wie wir das Fachwerk im Rechner abbilden.
Wir beschreiben nicht die Stäbe selbst, sondern nur die Informationen, die wir
später für das Gleichungssystem brauchen. Dazu gehören:

- die **Knotenkoordinaten** im $x$-$y$-Koordinatensystem,
- die Information, welche Knoten **fest gelagert** sind,
- die **Konnektivität**, also welche Knoten durch einen Stab verbunden sind, und
- die **Kräfte** an den Knoten.

Im folgenden Code bauen wir genau diese Datenstrukturen auf. Aus ihnen wird in
Kapitel 4.2 die **Steifigkeitsmatrix** $\mathbf{K}$ entstehen, mit der wir dann
das LGS

\begin{equation*}
\mathbf{K}\,\vec{u} = \vec{F}
\end{equation*}

lösen.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# --- Materialeigenschaften ---
elastizitaetsmodul = 2.1e11   # Stahl in N/m²
durchmesser        = 1.0e-2   # Stabdurchmesser in m
querschnitt        = np.pi * 0.25 * durchmesser**2   # Kreisquerschnitt in m²

# --- Knotenkoordinaten: knoten_pos[:, n] = [x_n, y_n] in m ---
# Knoten 0: linkes Lager   (0.0, 0.0)
# Knoten 1: freier Knoten  (1.0, 1.0)  <- Last greift hier an
# Knoten 2: rechtes Lager  (2.0, 0.0)
knoten_pos = np.array([
    [0.0,  1.0,  2.0],   # x-Koordinaten in m
    [0.0,  1.0,  0.0],   # y-Koordinaten in m
])
anzahl_knoten = knoten_pos.shape[1]   # Anzahl der Spalten = Anzahl Knoten

# --- Lagerknoten: diese Knoten können sich nicht verschieben ---
lager_indizes = [0, 2]

# --- Konnektivitätsmatrix: verbindung[i, j] = 1 wenn Stab i-j existiert ---
# Stab 0-1: linkes Lager  -> freier Knoten
# Stab 1-2: freier Knoten -> rechtes Lager
verbindung = np.zeros((anzahl_knoten, anzahl_knoten))
verbindung[0, 1] = 1
verbindung[1, 2] = 1
verbindung = verbindung + verbindung.T   # symmetrisch machen

# --- Kraftvektor: kraft_knoten[n, :] = [Fx, Fy] an Knoten n in N ---
kraft_knoten = np.zeros((anzahl_knoten, 2))
kraft_knoten[1, 1] = -5000.   # 5000 N nach unten an Knoten 1

# kraft_vektor fasst alle Knotenkräfte als 1D-Array zusammen:
# [Fx_0, Fy_0, Fx_1, Fy_1, Fx_2, Fy_2]
kraft_vektor = kraft_knoten.flatten()

print("Knotenkoordinaten (x, y):")
for n in range(anzahl_knoten):
    print(f"  Knoten {n}: ({knoten_pos[0, n]:.1f} m, {knoten_pos[1, n]:.1f} m)")

print("\nKonnektivitätsmatrix:")
print(verbindung.astype(int))

print("\nKraftvektor:")
print(kraft_vektor, "N")
```

Die **Konnektivitätsmatrix** `verbindung` ist das Herzstück der
Fachwerkbeschreibung. Eine 1 an Position `[i, j]` bedeutet: zwischen Knoten $i$
und Knoten $j$ existiert ein Stab. Die Matrix ist symmetrisch, weil ein Stab von
$i$ nach $j$ dasselbe ist wie von $j$ nach $i$.

Der Kraftvektor hat $2 \cdot n_\text{Knoten}$ Einträge, weil jeder Knoten
zwei Freiheitsgrade hat. Der Eintrag an Index $2n$ ist die Kraft in
$x$-Richtung, der Eintrag an Index $2n+1$ die Kraft in $y$-Richtung
an Knoten $n$.

````{admonition} Mini-Übung
:class: tip
1. Was gibt `kraft_vektor[3]` zurück, und was bedeutet dieser Wert
   physikalisch? Beantworten Sie die Frage zuerst im Kopf, dann prüfen
   Sie mit Code.
2. Wir wollen zusätzlich eine horizontale Kraft von 1000 N nach rechts
   an Knoten 1 anlegen. Welchen Eintrag in `kraft_knoten` müssen Sie
   ändern, und auf welchen Wert?
3. In der Konnektivitätsmatrix steht `verbindung[0, 2] = 0`. Was würde
   ein Stab zwischen Knoten 0 und Knoten 2 geometrisch darstellen?
   Beschreiben Sie in einem Satz, ohne Code auszuführen.
````

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# Frage 1: Index 3 = 2*1 + 1 = y-Richtung von Knoten 1
print(kraft_vektor[3])   # -5000.0 N
# -> die 5000 N nach unten (negatives y) an Knoten 1

# Frage 2: Horizontalkraft 1000 N nach rechts an Knoten 1
# kraft_knoten[1, 0] = 1000.   (Index 0 = x-Richtung)

# Frage 3: verbindung[0, 2] = 1 würde einen horizontalen Basisstab einfügen,
# der das linke und das rechte Lager direkt miteinander verbindet.
```

`kraft_vektor[3]` hat den Index $2 \cdot 1 + 1 = 3$, also die $y$-Kraft an
Knoten 1: die Last von $-5000$ N (Vorzeichen nach unten). Die horizontale
Kraft gehört in Spalte 0 von `kraft_knoten`, weil Spalte 0 der $x$-Richtung
entspricht.
````

## Das Fachwerk visualisieren

Bevor wir rechnen, plotten wir das Fachwerk, um die Geometrie zu überprüfen.
Eine Hilfsfunktion übernimmt den Plot und kann später auch die verformte
Lage darstellen.

```{code-cell} python
def zeichne_fachwerk(verschiebung=None, skalierung=500,
                     kraefte_anzeigen=False, kraft_vektor_plot=None,
                     titel=''):
    """Zeichnet das Fachwerk in Ausgangs- und skalierter Verformungslage.

    Parameters
    ----------
    verschiebung : ndarray, optional
        Verschiebungsvektor (2*anzahl_knoten,). Standard: keine Verformung.
    skalierung : float
        Überhöhungsfaktor für die Darstellung der Verformung.
    kraefte_anzeigen : bool
        Wenn True, werden Knotenkräfte als Text eingeblendet.
    kraft_vektor_plot : ndarray, optional
        Kraftvektor für die Beschriftung.
    titel : str
        Diagrammtitel.
    """
    if verschiebung is None:
        verschiebung = np.zeros(2 * anzahl_knoten)
    if kraft_vektor_plot is None:
        kraft_vektor_plot = np.zeros(2 * anzahl_knoten)

    fig, ax = plt.subplots(figsize=(7, 4))

    # Knotenposition nach Verformung (Verformung überhöht dargestellt)
    knoten_verformt = (knoten_pos
                       + skalierung * verschiebung.reshape((anzahl_knoten, 2)).T)

    # Stäbe: Ausgangslage grau, verformte Lage blau
    for i in range(anzahl_knoten):
        for j in range(i + 1, anzahl_knoten):
            if verbindung[i, j]:
                ax.plot([knoten_pos[0, i],    knoten_pos[0, j]],
                        [knoten_pos[1, i],    knoten_pos[1, j]],
                        color='gray', linewidth=1.5, alpha=0.3)
                ax.plot([knoten_verformt[0, i], knoten_verformt[0, j]],
                        [knoten_verformt[1, i], knoten_verformt[1, j]],
                        color='tab:blue', linewidth=2.5)

    # Knoten: Ausgangslage grau, verformte Lage rot
    ax.scatter(knoten_pos[0, :],    knoten_pos[1, :],
               c='gray', s=60, zorder=4, alpha=0.3)
    ax.scatter(knoten_verformt[0, :], knoten_verformt[1, :],
               c='tab:red', s=80, zorder=5)
    for n in range(anzahl_knoten):
        ax.text(knoten_verformt[0, n] + 0.04, knoten_verformt[1, n] + 0.04,
                f'K{n}', fontsize=9)

    # Lager als grüne Dreiecke
    h, b = 0.12, 0.12
    for n in lager_indizes:
        x_dreieck = [knoten_verformt[0, n],
                     knoten_verformt[0, n] - b / 2,
                     knoten_verformt[0, n] + b / 2]
        y_dreieck = [knoten_verformt[1, n],
                     knoten_verformt[1, n] - h,
                     knoten_verformt[1, n] - h]
        ax.fill(x_dreieck, y_dreieck, color='tab:green', alpha=0.7)

    # Knotenkräfte als Text
    if kraefte_anzeigen:
        for n in range(anzahl_knoten):
            ax.text(knoten_verformt[0, n] + 0.04,
                    knoten_verformt[1, n] - 0.15,
                    f'Fx = {kraft_vektor_plot[2*n]:.0f} N\n'
                    f'Fy = {kraft_vektor_plot[2*n+1]:.0f} N',
                    fontsize=7)

    ax.set_title(titel)
    ax.set_aspect('equal')
    ax.grid(True)
    plt.tight_layout()
    plt.show()


zeichne_fachwerk(titel='Kranausleger in Ausgangslage')
```

````{admonition} Mini-Übung
:class: tip
1. Verändern Sie die $y$-Koordinate von Knoten 1 von 1.0 auf 0.5 m und
   rufen Sie `zeichne_fachwerk` erneut auf. Beschreiben Sie in einem Satz,
   wie sich die Geometrie des Fachwerks dadurch ändert. Setzen Sie den Wert
   danach wieder auf 1.0 m zurück.
2. Warum ist es sinnvoll, die Verformung mit einem Überhöhungsfaktor
   darzustellen? Was würde man bei `skalierung=1` sehen?
````

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# Frage 1: y-Koordinate von Knoten 1 auf 0.5 m absenken
knoten_pos[1, 1] = 0.5
zeichne_fachwerk(titel='Fachwerk mit abgesenktem Knoten 1')
knoten_pos[1, 1] = 1.0   # zurücksetzen

# Frage 2: Reale Verformungen von Stahl unter Last sind winzig
# (typisch Bruchteile von Millimetern bei Meterstäben).
# Bei skalierung=1 wäre die Verformung im Plot nicht erkennbar.
# Der Überhöhungsfaktor macht sie sichtbar, ohne die Physik zu ändern.
```

Mit `knoten_pos[1, 1] = 0.5` wird das Dreieck flacher: Die Stäbe verlaufen
weniger steil, der freie Knoten liegt nur noch halb so hoch. Reale
Stahlverformungen liegen im Mikrometer- bis Millimeterbereich und wären
bei `skalierung=1` im Plot nicht erkennbar.
````

## Zusammenfassung und Ausblick

Ein ebenes Fachwerk wird vollständig durch drei Datenstrukturen beschrieben:
die Knotenkoordinaten `knoten_pos`, die Konnektivitätsmatrix `verbindung`
(welche Stäbe existieren) und den Kraftvektor `kraft_vektor` (welche Lasten
wo angreifen). Der Kraftvektor hat $2n$ Einträge, weil jeder Knoten einen
$x$- und einen $y$-Freiheitsgrad besitzt.

Im nächsten Kapitel berechnen wir aus diesen Daten die **globale
Steifigkeitsmatrix** und lösen das LGS $\mathbf{K} \cdot \vec{u} = \vec{F}$,
um die Knotenverschiebungen und Lagerkräfte zu bestimmen.
