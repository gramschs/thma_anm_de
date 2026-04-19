---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.3 Stabkräfte und Visualisierung

In Kapitel 4.2 haben wir das LGS $\mathbf{K} \cdot \vec{u} = \vec{F}$
gelöst und die Knotenverschiebungen sowie Lagerreaktionen berechnet. Jetzt
werten wir diese Ergebnisse weiter aus. Wir berechnen die **Stabkräfte**
und bestimmen, ob die einzelnen Stäbe auf Zug oder Druck beansprucht werden.
Anschließend stellen wir die verformte Lage und die Stabkräfte grafisch dar.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können die **Stabkraft** aus den Knotenverschiebungen berechnen
  und als Zug oder Druck interpretieren.
* [ ] Sie können die verformte Lage eines Fachwerks mit einem
  **Überhöhungsfaktor** visualisieren und den Faktor begründen.
* [ ] Sie können Stabkräfte farblich darstellen und aus der Darstellung
  ablesen, welche Stäbe auf Zug und welche auf Druck beansprucht werden.
```

## Stabkräfte berechnen

Die Knotenverschiebungen aus Kapitel 4.2 sagen uns, wie stark sich das
Fachwerk verformt. Sie sagen uns aber noch nicht, wie stark die einzelnen
Stäbe beansprucht werden. Dazu berechnen wir die **Stabkraft** $F_{ij}$
für jeden Stab.

```{admonition} Vorgehen
:class: note
1. **Relativverschiebung projizieren**: Den parallelen Anteil
   $u^{\parallel}$ der Relativverschiebung $\vec{u}_j - \vec{u}_i$
   entlang der Stabachse $\vec{e}$ berechnen.
2. **Stabkraft berechnen**: $F_{ij} = k \cdot u^{\parallel}$.
3. **Vorzeichen interpretieren**: $F_{ij} > 0$ bedeutet Zug,
   $F_{ij} < 0$ bedeutet Druck.
```

**Schritt 1 - Relativverschiebung projizieren:**
Wir kennen den Einheitsvektor $\vec{e}$ entlang der Stabachse aus
Kapitel 4.2. Die Projektion der Relativverschiebung auf die Stabachse
gibt die Längenänderung des Stabs:

\begin{equation*}
u^{\parallel} = \vec{e}^\top (\vec{u}_j - \vec{u}_i).
\end{equation*}

```{code-cell} python
# Stabkräfte berechnen
print("Stab  Länge      u_parallel    Stabkraft    Typ")
print("-" * 50)

for i in range(anzahl_knoten):
    for j in range(i + 1, anzahl_knoten):
        if verbindung[i, j]:
            # Geometrie
            differenz        = knoten_pos[:, j] - knoten_pos[:, i]
            staeblaenge      = np.linalg.norm(differenz)
            winkel           = np.arctan2(differenz[1], differenz[0])
            staebsteifigkeit = elastizitaetsmodul * querschnitt / staeblaenge

            # Einheitsvektor entlang der Stabachse
            einheitsvektor = np.array([np.cos(winkel), np.sin(winkel)])

            # Verschiebungen der Endknoten
            u_i = verschiebung_gesamt[2 * i : 2 * (i + 1)]
            u_j = verschiebung_gesamt[2 * j : 2 * (j + 1)]

            # Schritt 1: Projektion der Relativverschiebung
            u_parallel = np.dot(einheitsvektor, u_j - u_i)

            # Schritt 2: Stabkraft
            stabkraft = staebsteifigkeit * u_parallel

            # Schritt 3: Vorzeichen
            stabtyp = 'Zug' if stabkraft > 0 else 'Druck'

            print(f"  {i}-{j}   {staeblaenge:.3f} m   "
                  f"{u_parallel*1e3:.4f} mm   "
                  f"{stabkraft:.2f} N   {stabtyp}")
```

**Schritt 2 und 3 – Stabkraft und Vorzeichen:**
Die Stabkraft $F_{ij} = k \cdot u^{\parallel}$ hat dasselbe Vorzeichen
wie die Projektion $u^{\parallel}$:

- $F_{ij} > 0$: Der Stab wird gestreckt — er steht unter **Zug**.
- $F_{ij} < 0$: Der Stab wird gestaucht — er steht unter **Druck**.

Gedrückte Stäbe müssen auf Knicken ausgelegt werden, was eine eigene
Bemessungsaufgabe ist.

```{admonition} Mini-Übung
:class: tip

1. Stab 0–1 und Stab 1–2 haben denselben Betrag der Stabkraft. Warum

   ist das für dieses Fachwerk erwartet? Begründen Sie in einem Satz
   ohne Code.

2. Stehen die Stäbe unter Zug oder Druck? Überprüfen Sie das qualitativ:

   In welche Richtung zeigt die Last, und wie müssen sich die Stäbe
   verhalten, damit Knoten 1 im Gleichgewicht bleibt?

3. Verdoppeln Sie die Last auf $-10\,000\,\text{N}$ und berechnen Sie

   die Stabkräfte neu. Um welchen Faktor ändern sie sich, und warum?
```

```{code-cell} python

### Hier Ihren Code eingeben

```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Frage 3: doppelte Last
kraft_knoten_2 = np.zeros((anzahl_knoten, 2))
kraft_knoten_2[1, 1] = -10000.
kraft_vektor_2 = kraft_knoten_2.flatten()
kraft_red_2    = kraft_vektor_2[freie_dofs]
u_red_2        = np.linalg.solve(steifigkeit_reduziert, kraft_red_2)
verschiebung_2 = np.zeros(2 * anzahl_knoten)
verschiebung_2[freie_dofs] = u_red_2
```

Stab 0–1 und Stab 1–2 sind gleich lang und schließen denselben Winkel
mit der Horizontalen ein. Da die Last mittig an Knoten 1 angreift und
die Geometrie symmetrisch ist, tragen beide Stäbe gleich viel.

Die Last zeigt nach unten und zieht Knoten 1 nach unten. Die Stäbe
werden dabei gestreckt, weil ihre Endknoten 0 und 2 fest sind — sie
stehen also unter Zug.

Bei doppelter Last verdoppeln sich alle Verschiebungen (Linearität des
LGS) und damit auch alle Stabkräfte: $F_{ij} = k \cdot u^{\parallel}$,
und $u^{\parallel}$ ist proportional zur Last.
````