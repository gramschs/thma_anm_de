---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# Übung: Fachwerk mit fünf Knoten

In dieser Übung wenden Sie das in den Kapiteln 4.1–4.3 eingeführte FEM-Schema
auf ein **größeres Fachwerk** mit fünf Knoten und sechs Stäben an. Ziel ist,
dass Sie

- die Datenstrukturen (Knoten, Lager, Stäbe, Lasten) selbst aufbauen,
- die globale Steifigkeitsmatrix zusammensetzen,
- das reduzierte LGS lösen,
- Stabkräfte berechnen und interpretieren,
- die verformte Lage visualisieren.

Sie können sich dabei an den Vorlesungsnotebooks orientieren; der Algorithmus
bleibt derselbe.

---

## 1. Problemstellung

Wir betrachten ein ebenes Fachwerk mit fünf Knoten:

- Knoten 0, 1, 2 liegen auf der unteren Linie,
- Knoten 3 und 4 liegen darüber,
- zwei Knoten sind gelagert,
- an einem Knoten greift eine vertikale Last nach unten an,
- es gibt insgesamt 6 Stäbe.

Skizzieren Sie das Fachwerk grob auf Papier. Sie werden die genauen
Knotenkoordinaten und Stäbe im nächsten Abschnitt definieren.

---

## 2. Grundsetup: Bibliotheken, Material, Knoten, Lager, Stäbe, Lasten

### 2.1 Bibliotheken und Materialeigenschaften

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

### --- Materialeigenschaften ---

elastizitaetsmodul = 2.1e11      # N/m² (Stahl)
durchmesser        = 1.0e-2      # m
querschnitt        = np.pi * 0.25 * durchmesser**2
```

### 2.2 Knotenkoordinaten und Lagerknoten

Tragen Sie die Knotenkoordinaten als Array `knoten_pos` ein. Jede **Spalte**
entspricht einem Knoten, Zeile 0 enthält die x‑Koordinaten, Zeile 1 die
y‑Koordinaten. Definieren Sie außerdem die Liste der Lagerknoten
`lager_indizes`.

```{code-cell} python

### --- Knotenkoordinaten ---

### TODO: knoten_pos so definieren, dass jede Spalte einem Knoten entspricht:

### Knoten 0: (0, 0)

### Knoten 1: (1, 0)

### Knoten 2: (2, 0)

### Knoten 3: (0, 1)

### Knoten 4: (1, 1)

knoten_pos = np.array([
    # TODO: x-Koordinaten in m
    [],
    # TODO: y-Koordinaten in m
    [],
])

anzahl_knoten = knoten_pos.shape[1]

### --- Lagerknoten ---

### TODO: Liste der Indizes der fest gelagerten Knoten, z.B. [0, 3]

lager_indizes = []
print("Anzahl Knoten:", anzahl_knoten)
print("Lagerknoten:", lager_indizes)
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# --- Knotenkoordinaten ---
knoten_pos = np.array([
    [0., 1., 2., 0., 1.],   # x-Koordinaten in m
    [0., 0., 0., 1., 1.],   # y-Koordinaten in m
])

anzahl_knoten = knoten_pos.shape[1]

# --- Lagerknoten ---
lager_indizes = [0, 3]

print("Anzahl Knoten:", anzahl_knoten)
print("Lagerknoten:", lager_indizes)
```
````

### 2.3 Konnektivitätsmatrix und Kraftvektor

Stellen Sie nun die Konnektivitätsmatrix `verbindung` und den Kraftvektor
`kraft_vektor` auf.

Die Stäbe sind wie folgt gedacht (nur als Hinweis; versuchen Sie es zuerst
selbst):

- 0–1 (unten links),
- 1–2 (unten rechts),
- 3–4 (oben),
- 1–3 (diagonal),
- 1–4 (vertikal),
- 2–4 (diagonal).

An einem der Knoten (z.B. Knoten 2) greift eine Vertikallast nach unten an.

```{code-cell} python

### --- Konnektivitätsmatrix ---

verbindung = np.zeros((anzahl_knoten, anzahl_knoten))

### TODO: verbindung[i, j] = 1 für alle vorhandenen Stäbe (nur obere Dreiecksmatrix)

### Beispiel:

### verbindung[0, 1] = 1

### ...

### Symmetrisieren

verbindung = verbindung + verbindung.T

print("Konnektivitätsmatrix (0/1):")
print(verbindung.astype(int))
print("Anzahl Stäbe:", int(verbindung.sum() / 2))

### --- Kraftvektor ---

kraft_knoten = np.zeros((anzahl_knoten, 2))

### TODO: Lasten eintragen, z.B. vertikale Last an Knoten 2

### kraft_knoten[2, 1] = -3000.   # 3000 N nach unten

kraft_vektor = kraft_knoten.flatten()
print("Kraftvektor:", kraft_vektor)
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# --- Konnektivitätsmatrix ---
verbindung = np.zeros((anzahl_knoten, anzahl_knoten))

# Stäbe: 0-1, 1-2, 1-3, 1-4, 2-4, 3-4
verbindung[0, 1] = 1
verbindung[1, 2] = 1
verbindung[1, 3] = 1
verbindung[1, 4] = 1
verbindung[2, 4] = 1
verbindung[3, 4] = 1

verbindung = verbindung + verbindung.T

print("Konnektivitätsmatrix (0/1):")
print(verbindung.astype(int))
print("Anzahl Stäbe:", int(verbindung.sum() / 2))

# --- Kraftvektor ---
kraft_knoten = np.zeros((anzahl_knoten, 2))
kraft_knoten[2, 1] = -3000.   # 3000 N nach unten an Knoten 2
kraft_vektor = kraft_knoten.flatten()
print("Kraftvektor:", kraft_vektor)
```
````

---

## 3. Fachwerk-Visualisierung

Zur Kontrolle der Geometrie und später der Verformung verwenden wir eine
Plot-Funktion. Bitte diese Funktion nicht verändern.

```{code-cell} python
def zeichne_fachwerk(verschiebung=None, skalierung=500,
                     kraefte_anzeigen=False, kraft_vektor_plot=None,
                     titel=''):
    if verschiebung is None:
        verschiebung = np.zeros(2 * anzahl_knoten)
    if kraft_vektor_plot is None:
        kraft_vektor_plot = np.zeros(2 * anzahl_knoten)

    fig, ax = plt.subplots(figsize=(8, 5))
    knoten_verformt = (knoten_pos

                       + skalierung * verschiebung.reshape((anzahl_knoten, 2)).T)

    # Stäbe: Ausgangslage grau, verformte Lage blau
    for i in range(anzahl_knoten):
        for j in range(i + 1, anzahl_knoten):
            if verbindung[i, j]:
                ax.plot([knoten_pos[0, i],   knoten_pos[0, j]],
                        [knoten_pos[1, i],   knoten_pos[1, j]],
                        color='gray', linewidth=1.5, alpha=0.3)
                ax.plot([knoten_verformt[0, i], knoten_verformt[0, j]],
                        [knoten_verformt[1, i], knoten_verformt[1, j]],
                        color='tab:blue', linewidth=2.5)

    # Knoten
    ax.scatter(knoten_pos[0, :],   knoten_pos[1, :],
               c='gray', s=60, zorder=4, alpha=0.3)
    ax.scatter(knoten_verformt[0, :], knoten_verformt[1, :],
               c='tab:red', s=80, zorder=5)
    for n in range(anzahl_knoten):
        ax.text(knoten_verformt[0, n] + 0.03,
                knoten_verformt[1, n] + 0.03,
                f'K{n}', fontsize=9)

    # Lager als grüne Dreiecke
    h, b = 0.10, 0.10
    for n in lager_indizes:
        x_d = [knoten_verformt[0, n],
               knoten_verformt[0, n] - b / 2,
               knoten_verformt[0, n] + b / 2]
        y_d = [knoten_verformt[1, n],
               knoten_verformt[1, n] - h,
               knoten_verformt[1, n] - h]
        ax.fill(x_d, y_d, color='tab:green', alpha=0.7)

    # Knotenkräfte (optional)
    if kraefte_anzeigen:
        for n in range(anzahl_knoten):
            ax.text(knoten_verformt[0, n] + 0.03,
                    knoten_verformt[1, n] - 0.10,
                    f'Fy={kraft_vektor_plot[2*n+1]:.0f} N', fontsize=7)

    ax.set_title(titel)
    ax.set_aspect('equal')
    ax.grid(True)
    plt.tight_layout()
    plt.show()

### Test: Fachwerk in Ausgangslage

zeichne_fachwerk(titel='Fachwerk in Ausgangslage')
```

---

## 4. Globale Steifigkeitsmatrix aufbauen

Nun bauen Sie die **globale Steifigkeitsmatrix** `steifigkeit_global` auf,
indem Sie über alle Stäbe iterieren und die Elementsteifigkeitsmatrizen
an den richtigen Stellen eintragen.

```{code-cell} python

### --- Globale Steifigkeitsmatrix ---

steifigkeit_global = np.zeros((2 * anzahl_knoten, 2 * anzahl_knoten))

for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            # TODO: Differenzvektor, Stablänge, Winkel
            differenz = knoten_pos[:, j] - knoten_pos[:, i]
            staeblaenge = np.linalg.norm(differenz)
            winkel = np.arctan2(differenz[1], differenz[0])

            # TODO: Stabsteifigkeit k = E * A / L
            staebsteifigkeit = elastizitaetsmodul * querschnitt / staeblaenge

            # TODO: Elementsteifigkeitsmatrix k_element = k * e * e^T
            cos_w = np.cos(winkel)
            sin_w = np.sin(winkel)
            k_element = staebsteifigkeit * np.array([
                [cos_w**2,      sin_w * cos_w],
                [sin_w * cos_w, sin_w**2    ],
            ])

            # TODO: In globale Matrix eintragen (4 Blöcke)
            steifigkeit_global[2*i:2*(i+1), 2*i:2*(i+1)] += k_element
            steifigkeit_global[2*j:2*(j+1), 2*j:2*(j+1)] += k_element
            steifigkeit_global[2*i:2*(i+1), 2*j:2*(j+1)] -= k_element
            steifigkeit_global[2*j:2*(j+1), 2*i:2*(i+1)] -= k_element

print("Form von steifigkeit_global:", steifigkeit_global.shape)
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# --- Globale Steifigkeitsmatrix ---
steifigkeit_global = np.zeros((2 * anzahl_knoten, 2 * anzahl_knoten))

for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            # Geometrie
            differenz   = knoten_pos[:, j] - knoten_pos[:, i]
            staeblaenge = np.linalg.norm(differenz)
            winkel      = np.arctan2(differenz[1], differenz[0])

            # Stabsteifigkeit
            staebsteifigkeit = elastizitaetsmodul * querschnitt / staeblaenge

            # Elementsteifigkeitsmatrix
            cos_w = np.cos(winkel)
            sin_w = np.sin(winkel)
            k_element = staebsteifigkeit * np.array([
                [cos_w**2,      sin_w * cos_w],
                [sin_w * cos_w, sin_w**2    ],
            ])

            # In globale Matrix eintragen
            steifigkeit_global[2*i:2*(i+1), 2*i:2*(i+1)] += k_element
            steifigkeit_global[2*j:2*(j+1), 2*j:2*(j+1)] += k_element
            steifigkeit_global[2*i:2*(i+1), 2*j:2*(j+1)] -= k_element
            steifigkeit_global[2*j:2*(j+1), 2*i:2*(i+1)] -= k_element

print("Form von steifigkeit_global:", steifigkeit_global.shape)
```
````

---

## 5. LGS reduzieren und Verschiebungen berechnen

Bestimmen Sie die freien Freiheitsgrade, reduzieren Sie das LGS und
lösen Sie es mit `np.linalg.solve`.

```{code-cell} python

### --- Freie Freiheitsgrade bestimmen ---

freie_indizes = [n for n in range(anzahl_knoten) if n not in lager_indizes]

freie_dofs = []
for n in freie_indizes:
    # TODO: DOF-Indizes 2*n (x) und 2*n+1 (y) hinzufügen
    freie_dofs += [2*n, 2*n + 1]

freie_dofs = np.array(freie_dofs)
print("Freie Knoten:", freie_indizes)
print("Freie DOFs:", freie_dofs)

### --- Reduziertes System lösen ---

steifigkeit_reduziert = steifigkeit_global[freie_dofs, :][:, freie_dofs]
kraft_reduziert       = kraft_vektor[freie_dofs]

verschiebung_reduziert = np.linalg.solve(steifigkeit_reduziert,
                                         kraft_reduziert)

verschiebung_gesamt = np.zeros(2 * anzahl_knoten)
verschiebung_gesamt[freie_dofs] = verschiebung_reduziert

print("Verschiebungen (in mm):")
for n in range(anzahl_knoten):
    ux = verschiebung_gesamt[2*n]   * 1e3
    uy = verschiebung_gesamt[2*n+1] * 1e3
    print(f"Knoten {n}: ux = {ux:.4f} mm, uy = {uy:.4f} mm")
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# --- Freie Freiheitsgrade bestimmen ---
freie_indizes = [n for n in range(anzahl_knoten) if n not in lager_indizes]

freie_dofs = []
for n in freie_indizes:
    freie_dofs += [2*n, 2*n + 1]

freie_dofs = np.array(freie_dofs)
print("Freie Knoten:", freie_indizes)
print("Freie DOFs:", freie_dofs)

# --- Reduziertes System lösen ---
steifigkeit_reduziert = steifigkeit_global[freie_dofs, :][:, freie_dofs]
kraft_reduziert       = kraft_vektor[freie_dofs]

verschiebung_reduziert = np.linalg.solve(steifigkeit_reduziert,
                                         kraft_reduziert)

verschiebung_gesamt = np.zeros(2 * anzahl_knoten)
verschiebung_gesamt[freie_dofs] = verschiebung_reduziert

print("Verschiebungen (in mm):")
for n in range(anzahl_knoten):
    ux = verschiebung_gesamt[2*n]   * 1e3
    uy = verschiebung_gesamt[2*n+1] * 1e3
    print(f"Knoten {n}: ux = {ux:.4f} mm, uy = {uy:.4f} mm")
```
````

---

## 6. Stabkräfte berechnen

Berechnen Sie nun für jeden Stab die **Stabkraft** aus den Knotenverschiebungen.
Nutzen Sie dazu die Projektion der relativen Verschiebung auf die Stabachse.

```{code-cell} python
print("Stabkräfte (positiv = Zug, negativ = Druck):")
print(f"{'Stab':>6}  {'Länge':>8}  {'N in N':>12}  {'Typ':>8}")
print("-" * 42)

for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            differenz   = knoten_pos[:, j] - knoten_pos[:, i]
            staeblaenge = np.linalg.norm(differenz)
            winkel      = np.arctan2(differenz[1], differenz[0])
            staebsteifigkeit = elastizitaetsmodul * querschnitt / staeblaenge

            # TODO: Einheitsvektor entlang der Stabachse
            einheitsvektor = np.array([np.cos(winkel), np.sin(winkel)])

            # TODO: Relativverschiebung projizieren und Stabkraft berechnen
            u_i = verschiebung_gesamt[2*i : 2*(i+1)]
            u_j = verschiebung_gesamt[2*j : 2*(j+1)]
            delta = np.dot(einheitsvektor, u_j - u_i)
            stabkraft = staebsteifigkeit * delta

            stabtyp = 'Zug' if stabkraft > 0 else 'Druck'
            print(f"{i}-{j:>1}   {staeblaenge:>8.3f} m  "
                  f"{stabkraft:>12.2f} N  {stabtyp:>8}")
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
print("Stabkräfte (positiv = Zug, negativ = Druck):")
print(f"{'Stab':>6}  {'Länge':>8}  {'N in N':>12}  {'Typ':>8}")
print("-" * 42)

for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            differenz   = knoten_pos[:, j] - knoten_pos[:, i]
            staeblaenge = np.linalg.norm(differenz)
            winkel      = np.arctan2(differenz[1], differenz[0])
            staebsteifigkeit = elastizitaetsmodul * querschnitt / staeblaenge

            # Einheitsvektor entlang der Stabachse
            einheitsvektor = np.array([np.cos(winkel), np.sin(winkel)])

            # Projektion der Relativverschiebung
            u_i = verschiebung_gesamt[2*i : 2*(i+1)]
            u_j = verschiebung_gesamt[2*j : 2*(j+1)]
            delta = np.dot(einheitsvektor, u_j - u_i)

            # Stabkraft
            stabkraft = staebsteifigkeit * delta

            stabtyp = 'Zug' if stabkraft > 0 else 'Druck'
            print(f"{i}-{j:>1}   {staeblaenge:>8.3f} m  "
                  f"{stabkraft:>12.2f} N  {stabtyp:>8}")
```
````

---

## 7. Verformung darstellen

Stellen Sie die verformte Lage des Fachwerks mit einem Überhöhungsfaktor dar.

```{code-cell} python
zeichne_fachwerk(verschiebung=verschiebung_gesamt,
                 skalierung=500,
                 kraefte_anzeigen=False,
                 kraft_vektor_plot=None,
                 titel='Verformtes Fachwerk (Überhöhungsfaktor 500)')
```

---

## 8. Vertiefende Aufgaben

```{admonition} Vertiefung
:class: tip


1. **Stab mit größter Zug‑ bzw. Druckkraft**  

   Welcher Stab steht am stärksten auf Zug, welcher am stärksten auf Druck?
   Ist dieses Ergebnis für die belastete Seite qualitativ plausibel?


2. **Einfluss des Durchmessers**  

   Verdoppeln Sie den Durchmesser (`durchmesser = 2e-2` m), berechnen Sie
   den Querschnitt und das System neu. Um welchen Faktor ändert sich die
   vertikale Verschiebung von Knoten 2 (`u_y`)? Begründen Sie den Zusammenhang
   mit $k = EA/L$.


3. **Zusätzlicher Stab**  

   Fügen Sie einen weiteren Stab zwischen Knoten 0 und 4 ein
   (`verbindung[0, 4] = 1` vor der Symmetrisierung) und berechnen Sie
   Steifigkeitsmatrix, Verschiebungen und Stabkräfte neu. Wird das Fachwerk
   steifer oder weicher? Vergleichen Sie die maximale Verschiebung mit und
   ohne zusätzlichen Stab.
```

```{code-cell} python

### TODO: Hier Ihren Code für die Vertiefungsaufgaben ergänzen.

### Hinweise:

### - Für Aufgabe 2: durchmesser und querschnitt neu setzen, steifigkeit_global

### neu aufbauen, verschiebung_gesamt neu berechnen und u_y von Knoten 2 vergleichen.

### - Für Aufgabe 3: verbindung um Stab 0-4 erweitern (vor der Symmetrisierung),

### dann alle Schritte (steifigkeit_global, Lösung, Stabkräfte, Verformung)

### erneut durchführen und die maximale Verschiebung vergleichen.

```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# --- Aufgabe 2: Einfluss des Durchmessers (Beispielcode) ---
durchmesser_neu  = 2.0e-2
querschnitt_neu  = np.pi * 0.25 * durchmesser_neu**2

steifigkeit_global_neu = np.zeros((2 * anzahl_knoten, 2 * anzahl_knoten))
for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            differenz   = knoten_pos[:, j] - knoten_pos[:, i]
            staeblaenge = np.linalg.norm(differenz)
            winkel      = np.arctan2(differenz[1], differenz[0])
            staebsteifigkeit_neu = elastizitaetsmodul * querschnitt_neu / staeblaenge
            cos_w = np.cos(winkel)
            sin_w = np.sin(winkel)
            k_element_neu = staebsteifigkeit_neu * np.array([
                [cos_w**2,      sin_w * cos_w],
                [sin_w * cos_w, sin_w**2    ],
            ])
            steifigkeit_global_neu[2*i:2*(i+1), 2*i:2*(i+1)] += k_element_neu
            steifigkeit_global_neu[2*j:2*(j+1), 2*j:2*(j+1)] += k_element_neu
            steifigkeit_global_neu[2*i:2*(i+1), 2*j:2*(j+1)] -= k_element_neu
            steifigkeit_global_neu[2*j:2*(j+1), 2*i:2*(i+1)] -= k_element_neu

steifigkeit_reduziert_neu = steifigkeit_global_neu[freie_dofs, :][:, freie_dofs]
verschiebung_reduziert_neu = np.linalg.solve(steifigkeit_reduziert_neu,
                                             kraft_vektor[freie_dofs])
verschiebung_gesamt_neu = np.zeros(2 * anzahl_knoten)
verschiebung_gesamt_neu[freie_dofs] = verschiebung_reduziert_neu

u2y_alt = verschiebung_gesamt[2*2+1]
u2y_neu = verschiebung_gesamt_neu[2*2+1]
print(f"u2y (d=1 cm): {u2y_alt*1e3:.4f} mm")
print(f"u2y (d=2 cm): {u2y_neu*1e3:.4f} mm")
print(f"Faktor u_alt/u_neu: {u2y_alt / u2y_neu:.1f}")
# Erwartung: Faktor ~4, da A ~ d^2 und u ~ 1/k ~ 1/A

# --- Aufgabe 3: zusätzlicher Stab 0-4 (nur prinzipielle Schritte) ---
# Hinweis: In einem realen Workflow würden Sie verbindung neu definieren
# (mit Stab 0-4) und dann alle Schritte von vorne durchlaufen.
# Hier nur schematisch:
#
# verbindung[0, 4] = 1
# verbindung = verbindung + verbindung.T
# -> steifigkeit_global neu aufbauen (wie oben),
# -> verschiebung_gesamt neu berechnen,
# -> maximale Verschiebung vergleichen:
# max_u_alt = np.max(np.abs(verschiebung_gesamt[1::2]))
# max_u_neu = np.max(np.abs(verschiebung_gesamt_neu[1::2]))
# Das Fachwerk mit zusätzlichem Stab sollte steifer sein (kleinere max. Verschiebung).
```
````

---

## 9. Zusammenfassung

In dieser Übung haben Sie

- ein Fachwerk mit fünf Knoten durch Knotenkoordinaten, Lagerknoten,
  Konnektivitätsmatrix und Kraftvektor beschrieben,
- die globale Steifigkeitsmatrix aus den Stäben assembliert,
- das reduzierte LGS gelöst und die Knotenverschiebungen berechnet,
- aus den Verschiebungen die Stabkräfte (Zug/Druck) bestimmt,
- die verformte Lage mit einem Überhöhungsfaktor visualisiert und
- den Einfluss von Querschnittsänderungen und zusätzlicher Stabverbindung
  qualitativ und quantitativ untersucht.

Der Algorithmus ist derselbe wie beim Dreiecksfachewerk – nur die Größe von
Matrizen und Vektoren ist gewachsen.