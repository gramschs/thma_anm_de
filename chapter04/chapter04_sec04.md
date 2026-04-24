---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.4 Übung: Fachwerk mit fünf Knoten

In dieser Übung wenden Sie das in den Kapiteln 4.1-4.3 eingeführte FEM-Schema
auf ein **größeres Fachwerk** mit fünf Knoten und sechs Stäben an. Ziel ist,
dass Sie

- die Datenstrukturen (Knoten, Lager, Stäbe, Lasten) selbst aufbauen,
- die globale Steifigkeitsmatrix zusammensetzen,
- das reduzierte LGS lösen,
- Lagerkräfte berechnen und das Gleichgewicht prüfen,
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

# --- Materialeigenschaften ---
elastizitaetsmodul = 2.1e11      # N/m² (Stahl)
durchmesser        = 1.0e-2      # m
querschnitt        = np.pi * 0.25 * durchmesser**2
```

### 2.2 Knotenkoordinaten und Lagerknoten

Tragen Sie die Knotenkoordinaten als Array `knoten_pos` ein. Wie in den
Vorlesungskapiteln 4.1-4.3 gilt die Konvention: jede **Zeile** entspricht
einem Knoten, die beiden Spalten enthalten die x- bzw. y-Koordinate.
Die Form von `knoten_pos` ist also `(anzahl_knoten, 2)`.

Definieren Sie außerdem die Liste der Lagerknoten `lager_indizes`.

```{code-cell} python
# --- Knotenkoordinaten ---

# TODO: knoten_pos so definieren, dass jede Zeile einem Knoten entspricht:
# Knoten 0: (0, 0)
# Knoten 1: (1, 0)
# Knoten 2: (2, 0)
# Knoten 3: (0, 1)
# Knoten 4: (1, 1)

knoten_pos = np.array([
    # TODO: [x, y] pro Knoten eintragen
])

anzahl_knoten = knoten_pos.shape[0]

# --- Lagerknoten ---
# TODO: Liste der Indizes der fest gelagerten Knoten, z.B. [0, 3]

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
    [0., 0.],   # Knoten 0
    [1., 0.],   # Knoten 1
    [2., 0.],   # Knoten 2
    [0., 1.],   # Knoten 3
    [1., 1.],   # Knoten 4
])

anzahl_knoten = knoten_pos.shape[0]

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

- 0-1 (unten links),
- 1-2 (unten rechts),
- 3-4 (oben),
- 1-3 (diagonal),
- 1-4 (vertikal),
- 2-4 (diagonal).

An einem der Knoten (z. B. Knoten 2) greift eine Vertikallast nach unten an.

```{code-cell} python
# --- Konnektivitätsmatrix ---
verbindung = np.zeros((anzahl_knoten, anzahl_knoten))

# TODO: verbindung[i, j] = 1 für alle vorhandenen Stäbe (nur obere Dreiecksmatrix)
# Beispiel:
# verbindung[0, 1] = 1

# ...

# Symmetrisieren
verbindung = verbindung + verbindung.T

print("Konnektivitätsmatrix (0/1):")
print(verbindung.astype(int))
print("Anzahl Stäbe:", int(verbindung.sum() / 2))

# --- Kraftvektor ---
kraft_knoten = np.zeros((anzahl_knoten, 2))

# TODO: Lasten eintragen, z.B. vertikale Last an Knoten 2

# kraft_knoten[2, 1] = -3000.   # 3000 N nach unten

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
Plot-Funktion. Die Funktion erhält alle benötigten Daten als **Parameter**
und ist daher wiederverwendbar — auch wenn Sie in den Vertiefungsaufgaben
z. B. eine andere Konnektivitätsmatrix verwenden.

Bitte diese Funktion nicht verändern.

```{code-cell} python
def zeichne_fachwerk(knoten_pos, verbindung, lager_indizes,
                     verschiebung=None, skalierung=500,
                     kraefte_anzeigen=False, kraft_vektor_plot=None,
                     titel=''):
    """Zeichnet ein Fachwerk in Ausgangs- und verformter Lage.

   ### Parameter
    knoten_pos : ndarray, Form (n, 2)
        Knotenkoordinaten (Zeile = Knoten, Spalten = x, y).
    verbindung : ndarray, Form (n, n)
        Symmetrische Konnektivitätsmatrix.
    lager_indizes : list
        Indizes der fest gelagerten Knoten.
    verschiebung : ndarray oder None
        Verschiebungsvektor (Länge 2n). Standard: Nullverschiebung.
    skalierung : float
        Überhöhungsfaktor für die Verformungsdarstellung.
    kraefte_anzeigen : bool
        Falls True, werden Knotenkräfte annotiert.
    kraft_vektor_plot : ndarray oder None
        Kraftvektor für die Annotation (Länge 2n).
    titel : str
        Titel des Plots.
    """
    n = knoten_pos.shape[0]

    if verschiebung is None:
        verschiebung = np.zeros(2 * n)
    if kraft_vektor_plot is None:
        kraft_vektor_plot = np.zeros(2 * n)

    fig, ax = plt.subplots(figsize=(8, 5))
    knoten_verformt = (knoten_pos

                       + skalierung * verschiebung.reshape((n, 2)))

    # Stäbe: Ausgangslage grau, verformte Lage blau
    for i in range(n):
        for j in range(i + 1, n):
            if verbindung[i, j]:
                ax.plot([knoten_pos[i, 0],      knoten_pos[j, 0]],
                        [knoten_pos[i, 1],      knoten_pos[j, 1]],
                        color='gray', linewidth=1.5, alpha=0.3)
                ax.plot([knoten_verformt[i, 0], knoten_verformt[j, 0]],
                        [knoten_verformt[i, 1], knoten_verformt[j, 1]],
                        color='tab:blue', linewidth=2.5)

    # Knoten
    ax.scatter(knoten_pos[:, 0],      knoten_pos[:, 1],
               c='gray', s=60, zorder=4, alpha=0.3)
    ax.scatter(knoten_verformt[:, 0], knoten_verformt[:, 1],
               c='tab:red', s=80, zorder=5)
    for k in range(n):
        ax.text(knoten_verformt[k, 0] + 0.03,
                knoten_verformt[k, 1] + 0.03,
                f'K{k}', fontsize=9)

    # Lager als grüne Dreiecke
    h, b = 0.10, 0.10
    for k in lager_indizes:
        x_d = [knoten_verformt[k, 0],
               knoten_verformt[k, 0] - b / 2,
               knoten_verformt[k, 0] + b / 2]
        y_d = [knoten_verformt[k, 1],
               knoten_verformt[k, 1] - h,
               knoten_verformt[k, 1] - h]
        ax.fill(x_d, y_d, color='tab:green', alpha=0.7)

    # Knotenkräfte (optional)
    if kraefte_anzeigen:
        for k in range(n):
            ax.text(knoten_verformt[k, 0] + 0.03,
                    knoten_verformt[k, 1] - 0.10,
                    f'Fy={kraft_vektor_plot[2*k+1]:.0f} N', fontsize=7)

    ax.set_title(titel)
    ax.set_aspect('equal')
    ax.grid(True)
    plt.tight_layout()
    plt.show()

# Test: Fachwerk in Ausgangslage
# TODO: bitte auskommentieren
#zeichne_fachwerk(knoten_pos, verbindung, lager_indizes,
#                 titel='Fachwerk in Ausgangslage')
```

---

## 4. Globale Steifigkeitsmatrix aufbauen

Nun bauen Sie die **globale Steifigkeitsmatrix** `steifigkeit_global` auf,
indem Sie über alle Stäbe iterieren und die Elementsteifigkeitsmatrizen
an den richtigen Stellen eintragen.

```{code-cell} python
# --- Globale Steifigkeitsmatrix ---
steifigkeit_global = np.zeros((2 * anzahl_knoten, 2 * anzahl_knoten))

for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            # TODO: Differenzvektor von Knoten i nach Knoten j
            # TODO: Stablänge berechnen (np.linalg.norm)
            # TODO: Winkel berechnen (np.arctan2)
            # TODO: Stabsteifigkeit k = E * A / L
            # TODO: Elementsteifigkeitsmatrix k_el = k * e * e^T  (2x2)
            # TODO: Vier Blöcke in steifigkeit_global eintragen:
            #       (i,i) += k_el, (j,j) += k_el,
            #       (i,j) -= k_el, (j,i) -= k_el
            pass

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
            differenz  = knoten_pos[j] - knoten_pos[i]
            stablaenge = np.linalg.norm(differenz)
            winkel     = np.arctan2(differenz[1], differenz[0])

            # Stabsteifigkeit
            stabsteifigkeit = elastizitaetsmodul * querschnitt / stablaenge

            # Elementsteifigkeitsmatrix
            cos_w = np.cos(winkel)
            sin_w = np.sin(winkel)
            k_element = stabsteifigkeit * np.array([
                [cos_w**2,      sin_w * cos_w],
                [sin_w * cos_w, sin_w**2     ],
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
# --- Freie Freiheitsgrade bestimmen ---

# TODO: Liste der Knotenindizes, die NICHT in lager_indizes enthalten sind

freie_indizes = []   # TODO: z.B. mit List-Comprehension

# TODO: DOF-Indizes der freien Knoten sammeln:

# Für jeden freien Knoten n die Indizes 2*n und 2*n+1 aufnehmen

freie_dofs = []      # TODO
freie_dofs = np.array(freie_dofs)

print("Freie Knoten:", freie_indizes)
print("Freie DOFs:", freie_dofs)
```

```{code-cell} python
# --- Reduziertes System lösen ---

# TODO: Reduzierte Steifigkeitsmatrix und reduzierten Kraftvektor extrahieren

# TODO: Mit np.linalg.solve das reduzierte System lösen

# TODO: Vollständigen Verschiebungsvektor zusammensetzen

verschiebung_gesamt = np.zeros(2 * anzahl_knoten)

# TODO: verschiebung_gesamt an den freien DOFs füllen

print("Verschiebungen (in mm):")
for n in range(anzahl_knoten):
    ux = verschiebung_gesamt[2*n]   * 1e3
    uy = verschiebung_gesamt[2*n+1] * 1e3
    print(f"  Knoten {n}: ux = {ux:10.4f} mm, uy = {uy:10.4f} mm")
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
    print(f"  Knoten {n}: ux = {ux:10.4f} mm, uy = {uy:10.4f} mm")
```
````

---

## 6. Lagerkräfte und Gleichgewichtsprüfung

Ein wichtiger Nachbearbeitungsschritt: Aus dem vollständigen
Verschiebungsvektor berechnen wir über $\mathbf{F} = \mathbf{K} \cdot
\mathbf{u}$ den **vollständigen Kraftvektor** zurück. Dieser enthält:

- an den **freien** Knoten die externen Lasten (als Probe),
- an den **Lagerknoten** die Lagerreaktionen.

Prüfen Sie anschließend mit `np.allclose`, ob die berechneten Kräfte an den
freien DOFs mit den externen Lasten übereinstimmen.

```{code-cell} python

# --- Lagerkräfte berechnen ---

# TODO: Vollständigen Kraftvektor über F = K · u berechnen

# reaktion = ...

# --- Probe ---

# TODO: Prüfen, ob die berechneten Kräfte an den freien DOFs
# mit den externen Lasten übereinstimmen:

# np.allclose(reaktion[freie_dofs], kraft_vektor[freie_dofs])

# --- Knotenweise Ergebniszusammenfassung ---
# (nach Berechnung einkommentieren)
# print("\nKnotenweise Zusammenfassung:")
# print(f"{'Knoten':>6}  {'ux (mm)':>10}  {'uy (mm)':>10}  "
# f"{'Fx (N)':>10}  {'Fy (N)':>10}  {'Typ':>8}")
# print("-" * 62)

# for n in range(anzahl_knoten):

# ux = verschiebung_gesamt[2*n]   * 1e3

# uy = verschiebung_gesamt[2*n+1] * 1e3

# fx = reaktion[2*n]

# fy = reaktion[2*n+1]

# typ = "Lager" if n in lager_indizes else "frei"

# print(f"{n:>6}  {ux:>10.4f}  {uy:>10.4f}  "

# f"{fx:>10.2f}  {fy:>10.2f}  {typ:>8}")

```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# --- Lagerkräfte berechnen ---
reaktion = steifigkeit_global @ verschiebung_gesamt

# --- Probe ---
# An den freien DOFs muss F_berechnet ≈ F_extern gelten:
assert np.allclose(reaktion[freie_dofs], kraft_vektor[freie_dofs]), \
    "Gleichgewichtsprobe fehlgeschlagen!"
print("Gleichgewichtsprobe bestanden: F_berechnet ≈ F_extern an freien DOFs ✓")

# --- Knotenweise Ergebniszusammenfassung ---
print("\nKnotenweise Zusammenfassung:")
print(f"{'Knoten':>6}  {'ux (mm)':>10}  {'uy (mm)':>10}  "
      f"{'Fx (N)':>10}  {'Fy (N)':>10}  {'Typ':>8}")
print("-" * 62)
for n in range(anzahl_knoten):
    ux = verschiebung_gesamt[2*n]   * 1e3
    uy = verschiebung_gesamt[2*n+1] * 1e3
    fx = reaktion[2*n]
    fy = reaktion[2*n+1]
    typ = "Lager" if n in lager_indizes else "frei"
    print(f"{n:>6}  {ux:>10.4f}  {uy:>10.4f}  "
          f"{fx:>10.2f}  {fy:>10.2f}  {typ:>8}")
```
````

---

## 7. Stabkräfte berechnen

Berechnen Sie nun für jeden Stab die **Stabkraft** (Normalkraft) aus den
Knotenverschiebungen. Nutzen Sie dazu die Projektion der relativen
Verschiebung auf die Stabachse.

```{admonition} Vorzeichenkonvention
:class: note
Wir verwenden die in der Stabstatik übliche Konvention:
**Zug ist positiv**, **Druck ist negativ**.
```

```{code-cell} python
print("Stabkräfte (positiv = Zug, negativ = Druck):")
print(f"{'Stab':>6}  {'Länge':>8}  {'Kraft in N':>12}  {'Typ':>8}")
print("-" * 42)

for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            # TODO: Differenzvektor, Stablänge, Winkel, Stabsteifigkeit
            # TODO: Einheitsvektor entlang der Stabachse
            # TODO: Relativverschiebung projizieren und Stabkraft berechnen
            #       u_i = verschiebung_gesamt[2*i : 2*(i+1)]
            #       u_j = verschiebung_gesamt[2*j : 2*(j+1)]
            #       delta = np.dot(einheitsvektor, u_j - u_i)
            #       stabkraft = stabsteifigkeit * delta
            pass
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
print("Stabkräfte (positiv = Zug, negativ = Druck):")
print(f"{'Stab':>6}  {'Länge':>8}  {'Kraft in N':>12}  {'Typ':>8}")
print("-" * 42)

for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            differenz   = knoten_pos[j] - knoten_pos[i]
            stablaenge  = np.linalg.norm(differenz)
            winkel      = np.arctan2(differenz[1], differenz[0])
            stabsteifigkeit = elastizitaetsmodul * querschnitt / stablaenge

            # Einheitsvektor entlang der Stabachse
            einheitsvektor = np.array([np.cos(winkel), np.sin(winkel)])

            # Projektion der Relativverschiebung
            u_i = verschiebung_gesamt[2*i : 2*(i+1)]
            u_j = verschiebung_gesamt[2*j : 2*(j+1)]
            delta = np.dot(einheitsvektor, u_j - u_i)

            # Stabkraft
            stabkraft = stabsteifigkeit * delta

            stabtyp = 'Zug' if stabkraft > 0 else 'Druck'
            print(f"{i}-{j:>1}   {stablaenge:>8.3f} m  "
                  f"{stabkraft:>12.2f} N  {stabtyp:>8}")
```
````

---

## 8. Verformung darstellen

Stellen Sie die verformte Lage des Fachwerks mit einem Überhöhungsfaktor dar.

```{code-cell} python
#zeichne_fachwerk(knoten_pos, verbindung, lager_indizes,
#                 verschiebung=verschiebung_gesamt,
#                 skalierung=500,
#                 titel='Verformtes Fachwerk (Überhöhungsfaktor 500)')
```

---

## 9. Vertiefende Aufgaben

```{admonition} Hinweis: Notebook-Reihenfolge
:class: warning
Wenn Sie in den folgenden Aufgaben Parameter ändern (Durchmesser, Stäbe),
arbeiten Sie am besten in **neuen Code-Zellen**, ohne die Originalwerte
oben zu überschreiben. So bleiben die Referenzergebnisse erhalten und Sie
können die Resultate direkt vergleichen.
```

```{admonition} Vertiefung
:class: tip
1. **Stab mit größter Zug- bzw. Druckkraft:**
   Welcher Stab steht am stärksten auf Zug, welcher am stärksten auf Druck?
   Ist dieses Ergebnis für die belastete Seite qualitativ plausibel?
2. **Einfluss des Durchmessers:**
   Verdoppeln Sie den Durchmesser (`durchmesser = 2e-2` m), berechnen Sie
   den Querschnitt und das System neu. Um welchen Faktor ändert sich die
   vertikale Verschiebung von Knoten 2 (`u_y`)? Begründen Sie den Zusammenhang
   mit $k = EA/L$.
3. **Zusätzlicher Stab:**
   Fügen Sie einen weiteren Stab zwischen Knoten 0 und 4 ein und berechnen
   Sie Steifigkeitsmatrix, Verschiebungen und Stabkräfte neu. Wird das
   Fachwerk steifer oder weicher? Vergleichen Sie die maximale Verschiebung
   mit und ohne zusätzlichen Stab.
```

```{code-cell} python
# TODO: Hier Ihren Code für die Vertiefungsaufgaben ergänzen.
```

````{admonition} Lösung Aufgabe 2: Einfluss des Durchmessers
:class: tip
:class: dropdown
```python
# --- Aufgabe 2: Einfluss des Durchmessers ---
durchmesser_neu = 2.0e-2
querschnitt_neu = np.pi * 0.25 * durchmesser_neu**2

# Steifigkeitsmatrix neu aufbauen
steifigkeit_global_a2 = np.zeros((2 * anzahl_knoten, 2 * anzahl_knoten))
for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            differenz   = knoten_pos[j] - knoten_pos[i]
            stablaenge = np.linalg.norm(differenz)
            winkel      = np.arctan2(differenz[1], differenz[0])
            k_stab      = elastizitaetsmodul * querschnitt_neu / stablaenge
            cos_w, sin_w = np.cos(winkel), np.sin(winkel)
            k_el = k_stab * np.array([
                [cos_w**2,      sin_w * cos_w],
                [sin_w * cos_w, sin_w**2     ],
            ])
            steifigkeit_global_a2[2*i:2*(i+1), 2*i:2*(i+1)] += k_el
            steifigkeit_global_a2[2*j:2*(j+1), 2*j:2*(j+1)] += k_el
            steifigkeit_global_a2[2*i:2*(i+1), 2*j:2*(j+1)] -= k_el
            steifigkeit_global_a2[2*j:2*(j+1), 2*i:2*(i+1)] -= k_el

# Reduziertes System lösen (freie_dofs von oben wiederverwenden)
K_red_a2 = steifigkeit_global_a2[freie_dofs, :][:, freie_dofs]
u_red_a2 = np.linalg.solve(K_red_a2, kraft_vektor[freie_dofs])
u_a2     = np.zeros(2 * anzahl_knoten)
u_a2[freie_dofs] = u_red_a2

# Vergleich
u2y_alt = verschiebung_gesamt[2*2 + 1]
u2y_neu = u_a2[2*2 + 1]
print(f"u_y Knoten 2 (d = 1 cm): {u2y_alt*1e3:.4f} mm")
print(f"u_y Knoten 2 (d = 2 cm): {u2y_neu*1e3:.4f} mm")
print(f"Faktor: {u2y_alt / u2y_neu:.1f}")
print()
print("Begründung: A ~ d², also vervierfacht sich der Querschnitt.")
print("Da k = E·A/L und u ~ 1/k, viertelt sich die Verschiebung.")
print("Erwarteter Faktor: 4.")
```
````

````{admonition} Lösung Aufgabe 3: Zusätzlicher Stab
:class: tip
:class: dropdown
```python
# --- Aufgabe 3: Zusätzlicher Stab 0-4 ---
# Konnektivitätsmatrix komplett NEU aufbauen (nicht die alte modifizieren,
# um eine doppelte Symmetrisierung zu vermeiden).
verbindung_a3 = np.zeros((anzahl_knoten, anzahl_knoten))
verbindung_a3[0, 1] = 1
verbindung_a3[1, 2] = 1
verbindung_a3[1, 3] = 1
verbindung_a3[1, 4] = 1
verbindung_a3[2, 4] = 1
verbindung_a3[3, 4] = 1
verbindung_a3[0, 4] = 1   # neuer Stab
verbindung_a3 = verbindung_a3 + verbindung_a3.T

print(f"Anzahl Stäbe: {int(verbindung_a3.sum() / 2)} "
      f"(vorher: {int(verbindung.sum() / 2)})")

# Steifigkeitsmatrix neu aufbauen
steifigkeit_global_a3 = np.zeros((2 * anzahl_knoten, 2 * anzahl_knoten))
for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung_a3[i, j]:
            differenz   = knoten_pos[j] - knoten_pos[i]
            stablaenge  = np.linalg.norm(differenz)
            winkel      = np.arctan2(differenz[1], differenz[0])
            k_stab      = elastizitaetsmodul * querschnitt / stablaenge
            cos_w, sin_w = np.cos(winkel), np.sin(winkel)
            k_el = k_stab * np.array([
                [cos_w**2,      sin_w * cos_w],
                [sin_w * cos_w, sin_w**2     ],
            ])
            steifigkeit_global_a3[2*i:2*(i+1), 2*i:2*(i+1)] += k_el
            steifigkeit_global_a3[2*j:2*(j+1), 2*j:2*(j+1)] += k_el
            steifigkeit_global_a3[2*i:2*(i+1), 2*j:2*(j+1)] -= k_el
            steifigkeit_global_a3[2*j:2*(j+1), 2*i:2*(i+1)] -= k_el

# Lösen
K_red_a3 = steifigkeit_global_a3[freie_dofs, :][:, freie_dofs]
u_red_a3 = np.linalg.solve(K_red_a3, kraft_vektor[freie_dofs])
u_a3     = np.zeros(2 * anzahl_knoten)
u_a3[freie_dofs] = u_red_a3

# Vergleich maximale vertikale Verschiebung
max_uy_alt = np.max(np.abs(verschiebung_gesamt[1::2]))
max_uy_neu = np.max(np.abs(u_a3[1::2]))
print(f"Max |u_y| ohne Stab 0-4: {max_uy_alt*1e3:.4f} mm")
print(f"Max |u_y| mit  Stab 0-4: {max_uy_neu*1e3:.4f} mm")
print(f"Das Fachwerk wird {'steifer' if max_uy_neu < max_uy_alt else 'weicher'}.")

# Visualisierung mit der parametrisierten Plot-Funktion
zeichne_fachwerk(knoten_pos, verbindung_a3, lager_indizes,
                 verschiebung=u_a3, skalierung=500,
                 titel='Verformtes Fachwerk mit Stab 0-4 (Überhöhung 500)')
```
````

---

## 10. Zusammenfassung

In dieser Übung haben Sie

- ein Fachwerk mit fünf Knoten durch Knotenkoordinaten, Lagerknoten,
  Konnektivitätsmatrix und Kraftvektor beschrieben,
- die globale Steifigkeitsmatrix aus den Stäben assembliert,
- das reduzierte LGS gelöst und die Knotenverschiebungen berechnet,
- die **Lagerkräfte** über $\mathbf{F} = \mathbf{K} \cdot \mathbf{u}$
  bestimmt und das Gleichgewicht geprüft,
- aus den Verschiebungen die Stabkräfte (Zug/Druck) bestimmt,
- die verformte Lage mit einem Überhöhungsfaktor visualisiert und
- den Einfluss von Querschnittsänderungen und zusätzlicher Stabverbindung
  qualitativ und quantitativ untersucht.

Der Algorithmus ist derselbe wie beim Dreiecksfachwerk, nur die Größe von
Matrizen und Vektoren ist gewachsen.
