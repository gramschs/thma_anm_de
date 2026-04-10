---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.1 Matrizen und lineare Gleichungssysteme

An einem Obststand kostet ein Apfel 0,30 €, eine Banane 0,20 € und eine
Clementine 0,50 €. Wir kaufen an drei Tagen verschiedene Mengen und bezahlen
2,10 €, 1,70 € und 2,80 €. Morgen soll jemand anderes einkaufen, aber wir kennen
die Einzelpreise nicht mehr, nur noch die Mengen, die wir an den einzelnen Tagen
gekauft haben. *Wie findet man sie aus den Kassenbons zurück?*

Genau das ist die Aufgabe eines **linearen Gleichungssystems (LGS)**. In diesem
Kapitel lernen wir, wie man ein LGS als Matrixgleichung aufschreibt und
prüft, ob es überhaupt eine eindeutige Lösung gibt.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können ein lineares Gleichungssystem in der Matrixform
  $\mathbf{A} \cdot \vec{x} = \vec{b}$ aufschreiben.
* [ ] Sie können eine Matrix als 2D-NumPy-Array anlegen und auf Zeilen,
  Spalten und einzelne Elemente zugreifen.
* [ ] Sie können mit `np.linalg.det()` prüfen, ob ein LGS eine eindeutige
  Lösung hat.
```

+++

## Eine Matrix anlegen

Wir beginnen direkt mit dem Code. Die Einkaufsmengen an drei Tagen
(Zeile = Tag, Spalte = Fruchtsorte) lassen sich als Tabelle darstellen

| | Äpfel | Bananen | Clementinen |
| - | ----- | ------- | ----------- |
| **Tag 1** | 3 | 2 | 1 |
| **Tag 2** | 2 | 3 | 0 |
| **Tag 3** | 1 | 1 | 3 |

bzw. in einer Matrix kurz zusammenfassen:

\begin{equation*}
\mathbf{A} = \begin{pmatrix} 3 & 2 & 1 \\ 2 & 3 & 0 \\ 1 & 1 & 3 \end{pmatrix}.
\end{equation*}

Wir legen Matrizen in NumPy mit `np.array` an und greifen mit `A[i, j]` auf
Einträge zu. Hier nochmal unser Obst‑Beispiel:

```{code-cell} python
import numpy as np

# Koeffizientenmatrix: jede Zeile ist ein Einkaufstag
# Spalten: Anzahl Äpfel, Bananen, Clementinen
A = np.array([
    [3, 2, 1],   # Tag 1: 3 Aepfel, 2 Bananen, 1 Clementine
    [2, 3, 0],   # Tag 2: 2 Aepfel, 3 Bananen, 0 Clementinen
    [1, 1, 3],   # Tag 3: 1 Apfel,  1 Banane,  3 Clementinen
], dtype=float)  # dtype=float: Zahlen werden als Dezimalzahlen gespeichert

print('Matrix A:')
print(A)
print('Form (Zeilen, Spalten):', A.shape)
```

+++

## Vom Gleichungssystem zur Matrixform

Die drei Einkaufstage liefern drei Gleichungen. Dabei sind
$x_A$, $x_B$, $x_C$ die gesuchten Einzelpreise:

$$\begin{align}
3 x_A + 2 x_B + 1 x_C &= 2.10 \\
2 x_A + 3 x_B + 0 x_C &= 1.70 \\
1 x_A + 1 x_B + 3 x_C &= 2.80
\end{align}$$

Diese drei Gleichungen schreiben wir kompakt als **Matrixgleichung**:

\begin{equation*}
\mathbf{A} \cdot \vec{x} = \vec{b},
\end{equation*}

also ausgeschrieben als

\begin{equation*}
\begin{pmatrix} 3 & 2 & 1 \\ 2 & 3 & 0 \\ 1 & 1 & 3 \end{pmatrix}
\cdot \begin{pmatrix} x_A \\ x_B \\ x_C \end{pmatrix} =
\begin{pmatrix} 2.10 \\ 1.70 \\ 2.80 \end{pmatrix}.
\end{equation*}

$\mathbf{A}$ enthält die Koeffizienten (die Einkaufsmengen), $\vec{x}$ die
Unbekannten (die Preise) und $\vec{b}$ die rechten Seiten (die Kassenbons).

```{admonition} Wie ist die Matrix aufgebaut?
:class: note
Jede Zeile von $\mathbf{A}$ entspricht einer Gleichung, jede Spalte einer
Unbekannten. Der Eintrag $A_{ij}$ ist der Koeffizient der $j$-ten
Unbekannten in der $i$-ten Gleichung.
```

In NumPy legen wir die rechte Seite als einfaches 1D-Array an:

```{code-cell} python
# Rechte Seite: bezahlte Gesamtbetraege in Euro
b = np.array([2.10, 1.70, 2.80])

print('Koeffizientenmatrix A:')
print(A)
print('Rechte Seite b:', b)
```

+++

## Hat das System eine eindeutige Lösung?

Nicht jedes LGS hat genau eine Lösung. Es gibt drei Möglichkeiten:

* genau eine Lösung (Normalfall)
* keine Lösung (widersprüchliche Gleichungen)
* unendlich viele Lösungen (eine Gleichung ist doppelt)

Für quadratische Matrizen (gleich viele Gleichungen wie Unbekannte) prüfen
wir das schnell mit der **Determinante** $\det(\mathbf{A})$:

\begin{equation*}
\det(\mathbf{A}) \neq 0 \quad \Longleftrightarrow \quad \text{genau eine Lösung}.
\end{equation*}

Wir prüfen nun die Determinante für den Obsteinkauf.

```{code-cell} python
# Determinante berechnen
det_A = np.linalg.det(A)
print(f'Determinante: {det_A:.4f}')

# np.isclose prueft, ob ein Wert nahe bei 0 liegt
# (besser als == 0, weil Fließkommazahlen nie exakt sind)
if np.isclose(det_A, 0.0):
    print('det(A) = 0: keine eindeutige Lösung.')
else:
    print('det(A) ≠ 0: genau eine Lösung vorhanden.')
```

Zum Vergleich: eine singuläre Matrix, bei der eine Gleichung keine neue
Information liefert:

```{code-cell} python
# Zeile 2 = Zeile 0 + Zeile 1 -> keine neue Information
A_singular = np.array([
    [3, 2, 1],
    [2, 3, 0],
    [5, 5, 1],   # = Zeile 0 + Zeile 1
], dtype=float)

det_singular = np.linalg.det(A_singular)
print(f'Determinante der singulären Matrix: {det_singular:.2e}')
# Ergebnis ist nahe 0 (winzige Abweichung durch Fließkomma-Rechnung)
```

```{admonition} Mini-Übung
:class: tip
1. Warum ist `det_singular` nicht exakt null, obwohl die dritte Zeile
   mathematisch genau die Summe der ersten beiden ist? Lesen Sie den
   Kommentar im Code und formulieren Sie eine Antwort in eigenen Worten.
2. Wir haben `np.isclose(det_A, 0.0)` statt `det_A == 0.0` verwendet.
   Was würde passieren, wenn wir `== 0.0` auf `det_singular` anwenden?
   Testen Sie es.
```

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# Frage 1: Erklärung im Text
# Fließkommazahlen werden im Computer nur näherungsweise gespeichert.
# Berechnungen akkumulieren winzige Rundungsfehler, daher ist das Ergebnis
# z.B. 1.67e-15 statt exakt 0.

# Frage 2: == 0.0 schlägt fehl, weil der Wert nicht exakt null ist
A_singular = np.array([
    [3, 2, 1],
    [2, 3, 0],
    [5, 5, 1],
], dtype=float)
det_singular = np.linalg.det(A_singular)
print('== 0.0:', det_singular == 0.0)             # False (obwohl mathematisch 0)
print('isclose:', np.isclose(det_singular, 0.0))  # True (korrekt)
```

`== 0.0` gibt `False` zurück, obwohl die Determinante mathematisch null
ist, weil der Rechenwert z.B. `1.67e-15` lautet. `np.isclose` toleriert
solche winzigen Abweichungen.
````

+++

## Der Rang: das allgemeinere Kriterium

Die Determinante funktioniert nur für quadratische Matrizen. In der Praxis
gibt es aber auch Fälle mit mehr Gleichungen als Unbekannten. Das
allgemeinere Kriterium ist der **Rang** einer Matrix. Der Rang gibt an, wie
viele Gleichungen wirklich unabhängige Informationen enthalten.

Für ein System mit $n$ Unbekannten gilt:

$$\text{rang}(\mathbf{A}) = \text{rang}([\mathbf{A}|\vec{b}]) = n
\quad \Longleftrightarrow \quad \text{genau eine Lösung}$$

Dabei ist $[\mathbf{A}|\vec{b}]$ die **erweiterte Koeffizientenmatrix**:
$\mathbf{A}$ mit $\vec{b}$ als zusätzliche Spalte angehängt. Wir brauchen
`np.column_stack`, um diese erweiterte Matrix zu erzeugen:

```{code-cell} python
# Obstmarkt-System
A = np.array([[3,2,1],[2,3,0],[1,1,3]], dtype=float)
b = np.array([2.10, 1.70, 2.80])

# Rang von A allein
rang_A = np.linalg.matrix_rank(A)

# np.column_stack hängt b als neue Spalte rechts an A an
# Ergebnis ist die erweiterte Koeffizientenmatrix [A|b]
Ab = np.column_stack((A, b))
rang_Ab = np.linalg.matrix_rank(Ab)

n = A.shape[1]   # Anzahl Unbekannte = Anzahl Spalten von A

print(f'Rang von A:     {rang_A}')
print(f'Rang von [A|b]: {rang_Ab}')
print(f'Anzahl Unbekannte n = {n}')

if rang_A == rang_Ab == n:
    print('genau eine Lösung.')
elif rang_A == rang_Ab < n:
    print('unendlich viele Lösungen.')
else:
    print('keine Lösung.')
```

Die möglichen Fälle sind in der folgenden Tabelle zusammengefasst:

| $\text{rang}(\mathbf{A})$ | $\text{rang}([\mathbf{A}\|\vec{b}])$ | Lösbarkeit |
|:---:|:---:|:---|
| $= n$ | $= n$ | genau eine Lösung |
| $< n$ | $= \text{rang}(\mathbf{A})$ | unendlich viele Lösungen |
| $< n$ | $> \text{rang}(\mathbf{A})$ | keine Lösung |

```{admonition} Determinante vs. Rang
:class: note
Für quadratische Matrizen sind beide Kriterien gleichwertig: $\det(\mathbf{A})
\neq 0$ genau dann, wenn $\text{rang}(\mathbf{A}) = n$. Der Rang ist aber
allgemeiner, weil er auch für nicht-quadratische Matrizen definiert ist. Für
kleine Systeme und zum theoretischen Verständnis reicht der Determinanten-Test.
In der praktischen Numerik werden Determinanten kaum zur Lösbarkeitsprüfung
großer Systeme verwendet; dort arbeitet man eher mit dem Rang bzw. direkt mit
Lösungsverfahren wie `np.linalg.solve`.
```

+++

## Zusammenfassung und Ausblick

Ein LGS lässt sich immer als Matrixgleichung $\mathbf{A} \cdot \vec{x} = \vec{b}$
schreiben. Die Koeffizientenmatrix $\mathbf{A}$ legen wir als 2D-NumPy-Array
an, den rechten Seiten-Vektor $\vec{b}$ als 1D-Array. Mit
`np.linalg.det()` und `np.isclose()` prüfen wir, ob eine eindeutige Lösung
existiert, bevor wir versuchen, das System zu lösen.

Im nächsten Kapitel lernen wir `np.linalg.solve`, das die Lösung in einer
einzigen Zeile berechnet, und sichern das Ergebnis immer mit einer Probe ab.
