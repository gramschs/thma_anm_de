---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.1 Grundlagen Matrizen und lineare Gleichungssysteme

Im Supermarkt kostet ein Apfel 0,30 €, eine Banane 0,20 € und eine Orange
0,50 €. Wir kaufen an drei verschiedenen Tagen je eine Auswahl dieser Früchte
und bezahlen 2,10 €, 1,70 € und 2,80 €. Morgen soll jemand anderes einkaufen
— aber wir kennen die Einzelpreise nicht mehr, nur noch die Tagesbons. Wie
findet man die Einzelpreise zurück?

Genau das ist die Aufgabenstellung eines **linearen Gleichungssystems (LGS)**:
Mehrere Gleichungen, mehrere Unbekannte, eine kompakte Lösung. In Kapitel 2.2
haben wir das Werkzeug `np.linalg.solve` schon kurz gesehen. In diesem Kapitel
erarbeiten wir die Grundlage: Wie schreibt man ein LGS als Matrix-Gleichung,
und wann hat es überhaupt eine eindeutige Lösung?

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können ein **lineares Gleichungssystem** in der Matrixform
  $\mathbf{A} \cdot \vec{x} = \vec{b}$ aufschreiben.
* [ ] Sie können eine Matrix als 2D-NumPy-Array anlegen und auf einzelne
  Zeilen, Spalten und Elemente zugreifen.
* [ ] Sie können mit `np.linalg.det()` die Determinante berechnen und
  daraus auf die Eindeutigkeit der Lösung schließen.
* [ ] Sie können mit `np.linalg.matrix_rank()` den Rang einer Matrix 
  berechnen und als allgemeineres Lösbarkeitskriterium einsetzen.
```

+++

## Wiederholung: Matrizen als 2D-Arrays

In Kapitel 2.2 haben wir Matrizen bereits als zweidimensionale NumPy-Arrays
kennengelernt. Wir wiederholen die wichtigsten Punkte kurz und direkt am
Obstmarkt-Beispiel.

An drei Einkaufstagen wurden folgende Mengen gekauft (Zeile = Tag,
Spalte = Frucht: Äpfel, Bananen, Orangen):

$$\mathbf{A} = \begin{pmatrix} 3 & 2 & 1 \\ 2 & 3 & 0 \\ 1 & 1 & 3 \end{pmatrix}$$

In NumPy legen wir die Matrix zeilenweise an — jede innere Liste ist eine
Zeile:

```{code-cell} python
import numpy as np

# Koeffizientenmatrix: Zeile = Einkaufstag, Spalte = Fruchtsorte
A = np.array([
    [3, 2, 1],   # Tag 1: 3 Äpfel, 2 Bananen, 1 Orange
    [2, 3, 0],   # Tag 2: 2 Äpfel, 3 Bananen, 0 Orangen
    [1, 1, 3],   # Tag 3: 1 Apfel,  1 Banane,  3 Orangen
], dtype=float)

print('Matrix A:')
print(A)
print(f'Shape: {A.shape}')   # (3, 3): 3 Zeilen, 3 Spalten
```

`.shape` gibt ein Tupel `(Zeilen, Spalten)` zurück. Mit eckigen Klammern
greifen wir auf einzelne Elemente, Zeilen oder Spalten zu — der erste Index
wählt immer die **Zeile**, der zweite die **Spalte**:

```{code-cell} python
print(A[0, 2])    # Zeile 0, Spalte 2 → Anzahl Orangen an Tag 1
print(A[1, :])    # gesamte Zeile 1  → Einkauf Tag 2
print(A[:, 0])    # gesamte Spalte 0 → Äpfel an allen Tagen
```

```{admonition} Mini-Übung
:class: tip
:class: dropdown
Gegeben ist folgende Matrix mit den Noten von 4 Studierenden in 3 Prüfungen:

$$N = \begin{pmatrix} 1.3 & 2.0 & 1.7 \\ 2.3 & 3.0 & 2.7 \\
1.0 & 1.3 & 1.0 \\ 3.7 & 2.3 & 3.0 \end{pmatrix}$$

1. Legen Sie die Matrix als NumPy-Array an und geben Sie ihre Abmessungen aus.
2. Was gibt `N[2, :]` aus? Ermitteln Sie das Ergebnis zuerst im Kopf.
3. Was gibt `N[:, 1]` aus? Ermitteln Sie das Ergebnis zuerst im Kopf.
4. Berechnen Sie den Notendurchschnitt jedes Studierenden mit `np.mean(N,
   axis=1)`.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

N = np.array([
    [1.3, 2.0, 1.7],
    [2.3, 3.0, 2.7],
    [1.0, 1.3, 1.0],
    [3.7, 2.3, 3.0],
])

print(N.shape)            # (4, 3): 4 Studierende, 3 Prüfungen
print(N[2, :])            # [1.  1.3  1. ] Noten von Studierendem 3
print(N[:, 1])            # [2.  3.  1.3  2.3] Noten der 2. Prüfung
print(np.mean(N, axis=1)) # Durchschnitt pro Studierendem
```

`axis=1` bedeutet: Mittelwert entlang der Spaltenrichtung, also für jede
Zeile separat. Das Ergebnis ist ein Wert pro Studierendem.
````

+++

## Vom Gleichungssystem zur Matrixform

Zurück zum Obstmarkt. Die drei Einkaufstage liefern drei Gleichungen:

$$\begin{align}
3 x_A + 2 x_B + 1 x_O &= 2.10 \\
2 x_A + 3 x_B + 0 x_O &= 1.70 \\
1 x_A + 1 x_B + 3 x_O &= 2.80
\end{align}$$

Dabei sind $x_A$, $x_B$, $x_O$ die unbekannten Einzelpreise für Äpfel, Bananen
und Orangen in Euro.

Diese drei Gleichungen lassen sich kompakt als **Matrixgleichung** schreiben:

$$\mathbf{A} \cdot \vec{x} = \vec{b}$$

Die Koeffizientenmatrix $\mathbf{A}$ enthält die Einkaufsmengen, der
Lösungsvektor $\vec{x}$ die unbekannten Preise und der rechte Seiten-Vektor
$\vec{b}$ die bezahlten Gesamtbeträge:

$$\underbrace{\begin{pmatrix} 3 & 2 & 1 \\ 2 & 3 & 0 \\ 1 & 1 & 3 \end{pmatrix}}_{\mathbf{A}}
\cdot
\underbrace{\begin{pmatrix} x_A \\ x_B \\ x_O \end{pmatrix}}_{\vec{x}}
=
\underbrace{\begin{pmatrix} 2.10 \\ 1.70 \\ 2.80 \end{pmatrix}}_{\vec{b}}$$

In NumPy:

```{code-cell} python
# Koeffizientenmatrix
A = np.array([
    [3, 2, 1],
    [2, 3, 0],
    [1, 1, 3],
], dtype=float)

# Rechte Seite: bezahlte Gesamtbeträge in Euro
b = np.array([2.10, 1.70, 2.80])

print('Koeffizientenmatrix A:')
print(A)
print(f'\nRechte Seite b: {b}')
```

```{admonition} Aufbau der Matrixform
:class: note
Die Zeilen von $\mathbf{A}$ entsprechen den **Gleichungen**, die Spalten
den **Unbekannten**. Jeder Eintrag $A_{ij}$ ist der Koeffizient der
$j$-ten Unbekannten in der $i$-ten Gleichung. Der Vektor $\vec{b}$ enthält
die rechten Seiten aller Gleichungen.
```

````{admonition} Mini-Übung
:class: tip
Ein Café verkauft Kaffee (K), Tee (T) und Kuchen (C). An drei Tagen wurden
folgende Bestellungen aufgenommen und folgende Umsätze erzielt:

| Tag | Kaffee | Tee | Kuchen | Umsatz (€) |
|-----|--------|-----|--------|------------|
| Mo  | 5      | 2   | 3      | 24,50      |
| Di  | 3      | 4   | 1      | 17,30      |
| Mi  | 6      | 1   | 4      | 28,10      |

1. Schreiben Sie das LGS als Matrixgleichung $\mathbf{A} \cdot \vec{x} = \vec{b}$
   auf (auf Papier oder als Kommentar).
2. Legen Sie `A` und `b` als NumPy-Arrays an.
3. Geben Sie die Anzahl der Gleichungen und Unbekannten aus.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# Gleichungen:
#   5*K + 2*T + 3*C = 24.50
#   3*K + 4*T + 1*C = 17.30
#   6*K + 1*T + 4*C = 28.10

A = np.array([
    [5, 2, 3],
    [3, 4, 1],
    [6, 1, 4],
], dtype=float)

b = np.array([24.50, 17.30, 28.10])

n_gleichungen, n_unbekannte = A.shape
print(f'Anzahl Gleichungen: {n_gleichungen}')
print(f'Anzahl Unbekannte:  {n_unbekannte}')
```

Da die Anzahl der Gleichungen gleich der Anzahl der Unbekannten ist
(quadratische Matrix), kann das System prinzipiell eine eindeutige Lösung
haben. Ob das tatsächlich der Fall ist, prüfen wir im nächsten Abschnitt.
````

+++

## Hat das System eine eindeutige Lösung? Die Determinante

Nicht jedes LGS hat genau eine Lösung. Es gibt drei Möglichkeiten:

- **Genau eine Lösung** (Regelfall, den wir anstreben)
- **Keine Lösung** (widersprüchliche Gleichungen)
- **Unendlich viele Lösungen** (eine Gleichung ist überflüssig)

Für **quadratische** Matrizen gibt es einen schnellen Test: die
**Determinante** $\det(\mathbf{A})$.

$$\det(\mathbf{A}) \neq 0 \quad \Longleftrightarrow \quad \text{genau eine Lösung}$$

Anschaulich: Jede Gleichung in unserem System beschreibt eine Ebene im
Raum. Wenn alle drei Ebenen sich in genau einem Punkt schneiden,
ist $\det(\mathbf{A}) \neq 0$. Wenn zwei Ebenen parallel sind oder alle
drei eine gemeinsame Schnittgerade haben, ist $\det(\mathbf{A}) = 0$.

In NumPy:

```{code-cell} python
det_A = np.linalg.det(A)
print(f'Determinante: {det_A:.4f}')

if np.isclose(det_A, 0.0):
    print('→ det(A) = 0: Das System hat keine eindeutige Lösung.')
else:
    print('→ det(A) ≠ 0: Das System hat genau eine Lösung.')
```

```{admonition} Warum np.isclose statt == 0?
:class: note
Fließkommazahlen werden im Computer nur näherungsweise dargestellt.
Ein Vergleich `det_A == 0.0` schlägt oft fehl, selbst wenn die
Determinante mathematisch exakt null ist — weil das Rechenergebnis
z.B. `1.3e-15` lauten kann. `np.isclose(det_A, 0.0)` prüft stattdessen,
ob der Wert nahe genug bei null liegt. Das ist der robustere Weg.
```

Schauen wir uns einen Fall an, bei dem die Determinante tatsächlich null ist.
Das passiert zum Beispiel, wenn eine Gleichung eine Linearkombination der
anderen ist — also keine neue Information liefert:

```{code-cell} python
# Dritte Zeile = Summe der ersten beiden → überflüssige Gleichung
A_singular = np.array([
    [3, 2, 1],
    [2, 3, 0],
    [5, 5, 1],   # = Zeile 0 + Zeile 1
], dtype=float)

det_singular = np.linalg.det(A_singular)
print(f'Determinante der singulären Matrix: {det_singular:.2e}')
```

````{admonition} Mini-Übung
:class: tip
Prüfen Sie für die Café-Matrix aus der vorigen Mini-Übung, ob das System
eine eindeutige Lösung hat. Geben Sie die Determinante aus und formulieren
Sie eine Aussage als `if`-Bedingung.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip, dropdown
```python
import numpy as np

A = np.array([
    [5, 2, 3],
    [3, 4, 1],
    [6, 1, 4],
], dtype=float)

det_A = np.linalg.det(A)
print(f'Determinante: {det_A:.4f}')

if np.isclose(det_A, 0.0):
    print('→ Keine eindeutige Lösung.')
else:
    print('→ Genau eine Lösung.')
```

Die Determinante ist ungleich null — das System hat eine eindeutige Lösung.
Die Einzelpreise für Kaffee, Tee und Kuchen lassen sich also eindeutig
bestimmen.
````

+++

## Der Rang: das allgemeinere Kriterium

Die Determinante funktioniert nur für **quadratische** Matrizen (gleich viele
Gleichungen wie Unbekannte). In der Praxis kommt es aber auch vor, dass
mehr Gleichungen als Unbekannte vorliegen (überbestimmtes System) oder
umgekehrt.

Das allgemeinere Kriterium ist der **Rang** einer Matrix. Der Rang gibt an,
wie viele Gleichungen wirklich unabhängige Informationen enthalten:

$$\text{rang}(\mathbf{A}) = \text{rang}([\mathbf{A}|\vec{b}]) = n
\quad \Longleftrightarrow \quad \text{genau eine Lösung}$$

Dabei ist $n$ die Anzahl der Unbekannten und $[\mathbf{A}|\vec{b}]$ die
**erweiterte Koeffizientenmatrix** — also $\mathbf{A}$ mit $\vec{b}$ als
zusätzliche Spalte angehängt.

In NumPy:

```{code-cell} python
# Obstmarkt-System
A = np.array([[3,2,1],[2,3,0],[1,1,3]], dtype=float)
b = np.array([2.10, 1.70, 2.80])

# Rang von A allein
rang_A = np.linalg.matrix_rank(A)

# Rang der erweiterten Matrix [A|b]
Ab = np.column_stack((A, b))   # b als zusätzliche Spalte anhängen
rang_Ab = np.linalg.matrix_rank(Ab)

n = A.shape[1]   # Anzahl Unbekannte

print(f'Rang von A:     {rang_A}')
print(f'Rang von [A|b]: {rang_Ab}')
print(f'Anzahl Unbekannte n = {n}')

if rang_A == rang_Ab == n:
    print('→ Genau eine Lösung.')
elif rang_A == rang_Ab < n:
    print('→ Unendlich viele Lösungen.')
else:
    print('→ Keine Lösung (widersprüchliches System).')
```

```{admonition} Determinante vs. Rang
:class: note
Für quadratische Matrizen sind Determinante und Rang gleichwertige
Kriterien: $\det(\mathbf{A}) \neq 0$ genau dann, wenn
$\text{rang}(\mathbf{A}) = n$. Der Rang ist aber allgemeiner, weil er
auch für nicht-quadratische Matrizen definiert ist. In der Praxis reicht
für die meisten Ingenieuraufgaben der Determinanten-Test.
```

Zur Illustration: dasselbe System mit einer überflüssigen (linear abhängigen)
dritten Gleichung:

```{code-cell} python
# Dritte Gleichung = Summe der ersten beiden → keine neue Information
A_abh = np.array([[3,2,1],[2,3,0],[5,5,1]], dtype=float)
b_abh = np.array([2.10, 1.70, 3.80])   # 2.10 + 1.70 = 3.80

Ab_abh = np.column_stack((A_abh, b_abh))

print(f'Rang A:     {np.linalg.matrix_rank(A_abh)}')
print(f'Rang [A|b]: {np.linalg.matrix_rank(Ab_abh)}')
print(f'n = {A_abh.shape[1]}')
```

````{admonition} Mini-Übung
:class: tip
Gegeben sind folgende drei Systeme. Prüfen Sie jeweils mit Determinante
**und** Rang, ob eine eindeutige Lösung existiert. Versuchen Sie, das
Ergebnis vorher durch Inspektion der Zeilen zu erahnen.

**System 1:**
$$\mathbf{A_1} = \begin{pmatrix} 1 & 2 \\ 3 & 4 \end{pmatrix}, \quad
\vec{b_1} = \begin{pmatrix} 5 \\ 6 \end{pmatrix}$$

**System 2:**
$$\mathbf{A_2} = \begin{pmatrix} 1 & 2 \\ 2 & 4 \end{pmatrix}, \quad
\vec{b_2} = \begin{pmatrix} 5 \\ 10 \end{pmatrix}$$

**System 3:**
$$\mathbf{A_3} = \begin{pmatrix} 1 & 2 \\ 2 & 4 \end{pmatrix}, \quad
\vec{b_3} = \begin{pmatrix} 5 \\ 11 \end{pmatrix}$$
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip, dropdown
```python
import numpy as np

systeme = [
    (np.array([[1,2],[3,4]], dtype=float), np.array([5., 6.])),
    (np.array([[1,2],[2,4]], dtype=float), np.array([5., 10.])),
    (np.array([[1,2],[2,4]], dtype=float), np.array([5., 11.])),
]

for i, (A, b) in enumerate(systeme, start=1):
    n      = A.shape[1]
    det    = np.linalg.det(A)
    rA     = np.linalg.matrix_rank(A)
    rAb    = np.linalg.matrix_rank(np.column_stack((A, b)))

    if rA == rAb == n:
        loesung = 'Genau eine Lösung'
    elif rA == rAb < n:
        loesung = 'Unendlich viele Lösungen'
    else:
        loesung = 'Keine Lösung'

    print(f'System {i}: det={det:.1f}, rang(A)={rA}, rang([A|b])={rAb} → {loesung}')
```

Ausgabe:
```
System 1: det=-2.0, rang(A)=2, rang([A|b])=2 → Genau eine Lösung
System 2: det=0.0,  rang(A)=1, rang([A|b])=1 → Unendlich viele Lösungen
System 3: det=0.0,  rang(A)=1, rang([A|b])=2 → Keine Lösung
```

System 2: Die zweite Gleichung ($2x + 4y = 10$) ist das Doppelte der
ersten ($x + 2y = 5$) — sie enthält keine neue Information, daher
unendlich viele Lösungen. System 3: Die zweite Gleichung widerspricht der
ersten ($2 \cdot 5 = 10 \neq 11$) — kein gemeinsamer Schnittpunkt.
````

+++

## Zusammenfassung und Ausblick

Wir haben die Grundlagen für das Lösen linearer Gleichungssysteme gelegt.
Ein LGS lässt sich immer als Matrixgleichung $\mathbf{A} \cdot \vec{x} = \vec{b}$
schreiben, wobei die Zeilen von $\mathbf{A}$ den Gleichungen und die Spalten
den Unbekannten entsprechen. Mit `np.linalg.det()` und `np.linalg.matrix_rank()`
prüfen wir, ob das System eine eindeutige Lösung hat — bevor wir versuchen,
es zu lösen.

Im nächsten Notebook lernen wir die eigentliche Lösung: `np.linalg.solve`
und die Inverse einer Matrix.
