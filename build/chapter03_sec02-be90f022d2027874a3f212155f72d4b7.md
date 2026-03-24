---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.2 Matrizen und lineare Gleichungssysteme

In Kapitel 2.1 haben wir das Schwingungssignal eines einzelnen Sensors
modelliert. In der Praxis überwacht man eine Maschine jedoch oft mit mehreren
Sensoren gleichzeitig, zum Beispiel an verschiedenen Lagerstellen. Die
Messwerte aller Sensoren lassen sich dann als **Matrix** organisieren: jede
Zeile entspricht einem Sensor, jede Spalte einem Zeitpunkt. NumPy unterstützt
solche zweidimensionalen Arrays genauso natürlich wie die eindimensionalen
Arrays aus Kapitel 2.1.

Außerdem taucht in der Ingenieurpraxis regelmäßig die Frage auf, welche
Materialeigenschaften oder Kosten hinter einer Reihe von Messungen stecken.
Das führt fast immer auf ein **lineares Gleichungssystem**, das wir mit NumPy
in einer einzigen Zeile lösen können.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können ein zweidimensionales NumPy-Array (Matrix) erzeugen und
  dessen `.shape` interpretieren.
* [ ] Sie können auf einzelne Zeilen, Spalten und Elemente einer Matrix
  zugreifen.
* [ ] Sie können eine **Matrix-Vektor-Multiplikation** mit dem `@`-Operator
  durchführen.
* [ ] Sie können ein lineares Gleichungssystem mit `np.linalg.solve()` lösen.
* [ ] Sie können die Inverse einer Matrix mit `np.linalg.inv()` berechnen.
* [ ] Sie können mit `np.linalg.det()` die Determinante einer Matrix berechnen
  und die Lösbarkeit eines Gleichungssystems beurteilen.
```

## Zweidimensionale Arrays

Wir stellen uns vor, dass drei Beschleunigungssensoren unsere Maschine
gleichzeitig überwachen. Jeder Sensor liefert Messwerte zu denselben vier
Zeitpunkten. Diese Daten speichern wir als zweidimensionales Array:

```{code-cell} python
import numpy as np

# Messwerte: 3 Sensoren, 4 Zeitpunkte (in m/s²)
messungen = np.array([
    [0.3, 1.2, 2.5, 1.8],   # Sensor 1
    [0.5, 0.9, 1.7, 1.1],   # Sensor 2
    [1.2, 2.4, 3.1, 2.0],   # Sensor 3
])

print(messungen)
print(messungen.shape)
```

`.shape` gibt nun ein Tupel mit zwei Werten zurück: `(3, 4)` bedeutet 3 Zeilen
und 4 Spalten. Die erste Zahl ist immer die Zeilenanzahl, die zweite die
Spaltenanzahl.

Auf einzelne Elemente, Zeilen und Spalten greifen wir mit eckigen Klammern zu:

```{code-cell} python
print(messungen[0, 2])    # Zeile 0, Spalte 2: Sensor 1 zum dritten Zeitpunkt
print(messungen[1, :])    # Gesamte Zeile 1: alle Werte von Sensor 2
print(messungen[:, 0])    # Gesamte Spalte 0: alle Sensoren zum ersten Zeitpunkt
```

Der Doppelpunkt `:` bedeutet "alle Einträge in dieser Dimension". Das ist
dasselbe Slicing-Prinzip, das wir von Python-Listen kennen, jetzt aber in
zwei Dimensionen.

```{admonition} Zeilen und Spalten
:class: note
Bei einem zweidimensionalen Array gilt: Der erste Index wählt die **Zeile**,
der zweite die **Spalte**. Eine gute Eselsbrücke ist "Row before Column",
also erst die Zeile, dann die Spalte. `messungen[i, j]` greift auf den Wert
in Zeile `i` und Spalte `j` zu.
```

````{admonition} Mini-Übung
:class: tip
Gegeben ist folgende Matrix mit Schwingungsamplituden (in mm) von vier
Sensoren an drei verschiedenen Messpunkten:

\begin{equation*}
A = \begin{pmatrix}
0.12 & 0.34 & 0.21 \\
0.45 & 0.11 & 0.38 \\
0.29 & 0.52 & 0.17 \\
0.08 & 0.43 & 0.31 \\
\end{pmatrix}
\end{equation*}

1. Erzeugen Sie die Matrix in Python und geben Sie die Zeilen- und Spaltenanzahl
   aus.
2. Geben Sie die Amplituden von Sensor 3 (Zeile 2) an allen Messpunkten aus.
3. Geben Sie die Amplituden aller Sensoren am zweiten Messpunkt (Spalte 1) aus.
4. Was liefert `amplituden[2, 1]`? Berechnen Sie das Ergebnis zuerst im Kopf.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

amplituden = np.array([
    [0.12, 0.34, 0.21],
    [0.45, 0.11, 0.38],
    [0.29, 0.52, 0.17],
    [0.08, 0.43, 0.31],
])

print(amplituden.shape)      # (4, 3)
print(amplituden[2, :])      # [0.29 0.52 0.17]
print(amplituden[:, 1])      # [0.34 0.11 0.52 0.43]
print(amplituden[2, 1])      # 0.52
```

`amplituden[2, 1]` greift auf Zeile 2 (Sensor 3) und Spalte 1 (zweiter
Messpunkt) zu. Das ist der Wert 0.52 mm. Die Form `(4, 3)` bestätigt:
4 Sensoren, 3 Messpunkte.
````

## Matrix-Vektor-Multiplikation mit `@`

Eine der wichtigsten Operationen in der linearen Algebra ist die
Matrix-Vektor-Multiplikation. Angenommen, wir kennen die Empfindlichkeit
jedes Sensors als Kalibrierungsfaktor und wollen daraus die tatsächlichen
physikalischen Größen berechnen. Das entspricht einer Multiplikation der
Kalibrierungsmatrix mit dem Messvektor.

In NumPy verwenden wir dafür den `@`-Operator:

```{code-cell} python
# Kalibrierungsmatrix K (3x3): Umrechnungsfaktoren zwischen Sensoren
K = np.array([
    [2.0, 0.5, 0.0],
    [0.3, 1.8, 0.2],
    [0.0, 0.4, 2.1],
])

# Messvektor: Rohwerte der drei Sensoren zum selben Zeitpunkt (in V)
u = np.array([1.0, 0.8, 1.2])

# Kalibrierte Beschleunigungen (in m/s²)
a = K @ u
print(a)
```

Der `@`-Operator führt die Matrixmultiplikation durch. `K @ u` bedeutet:
multipliziere die Matrix `K` mit dem Vektor `u` und liefere den
Ergebnisvektor `a`. Das Ergebnis hat dieselbe Anzahl von Zeilen wie `K`.

```{admonition} Unterschiede Multiplikationsoperatoren
:class: note
Der `*`-Operator multipliziert Arrays **elementweise**, der `@`-Operator
führt die **Matrixmultiplikation** durch. `K * u` wäre hier falsch, weil es
nicht der linearen Algebra entspricht. Für zwei Matrizen `A` und `B` gilt:
`A @ B` ist die Matrixmultiplikation, `A * B` ist die elementweise
Multiplikation.
```

````{admonition} Mini-Übung
:class: tip
Die Steifigkeitsmatrix eines einfachen Feder-Masse-Systems lautet:

```python
K = np.array([
    [ 3.0, -1.0],
    [-1.0,  2.0],
])
```

Der Verschiebungsvektor der beiden Massen beträgt `u = np.array([0.5, 0.3])`
in mm.

1. Berechnen Sie den Kraftvektor `F = K @ u`.
2. Geben Sie `F` aus und überprüfen Sie das Ergebnis für die erste Komponente
   von Hand: $F_1 = 3.0 \cdot 0.5 + (-1.0) \cdot 0.3$.
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
    [ 3.0, -1.0],
    [-1.0,  2.0],
])
u = np.array([0.5, 0.3])

F = K @ u
print(F)
```

Ausgabe:
```
[1.2 0.1]
```

Die erste Komponente: $3.0 \cdot 0.5 + (-1.0) \cdot 0.3 = 1.5 - 0.3 = 1.2$ N.
Die zweite Komponente: $(-1.0) \cdot 0.5 + 2.0 \cdot 0.3 = -0.5 + 0.6 = 0.1$ N.
````

## Lineare Gleichungssysteme lösen mit `np.linalg.solve()`

Wir kehren zu unserer Maschine zurück. Drei Sensoren liefern Messwerte, aber
ihre Kalibrierungsfaktoren sind unbekannt. Aus drei Referenzmessungen mit
bekannten Beschleunigungen lässt sich ein lineares Gleichungssystem aufstellen:

$$\mathbf{A} \cdot \vec{x} = \vec{b}$$

Dabei enthält die Matrix $\mathbf{A}$ die Rohspannungen der Sensoren bei den
Referenzmessungen, $\vec{b}$ die bekannten Referenzbeschleunigungen und
$\vec{x}$ die gesuchten Kalibrierungsfaktoren.

```{code-cell} python
# Rohspannungen der 3 Sensoren bei 3 Referenzmessungen (in V)
A = np.array([
    [1.0, 0.0, 0.5],
    [0.0, 2.0, 1.0],
    [1.5, 1.0, 0.0],
])

# Bekannte Referenzbeschleunigungen (in m/s²)
b = np.array([2.5, 4.0, 3.0])

# Gesuchte Kalibrierungsfaktoren
x = np.linalg.solve(A, b)
print(f"Kalibrierungsfaktoren: {x}")
```

`np.linalg.solve(A, b)` löst das Gleichungssystem $\mathbf{A} \cdot \vec{x} =
\vec{b}$ direkt. Intern verwendet NumPy dabei eine sogenannte LU-Zerlegung, die
deutlich effizienter ist als das von Hand durchgeführte Gaußsche
Eliminationsverfahren.

Wir können das Ergebnis überprüfen, indem wir $\mathbf{A} \cdot \vec{x}$
berechnen und mit $\vec{b}$ vergleichen:

```{code-cell} python
b_probe = A @ x
print(f"Probe A @ x: {b_probe}")
print(f"Original  b: {b}")
```

Die Werte stimmen überein, das Gleichungssystem ist korrekt gelöst.

## Die Inverse einer Matrix

Eine alternative Methode zum Lösen von $\mathbf{A} \cdot \vec{x} = \vec{b}$
nutzt die Inverse $\mathbf{A}^{-1}$:

$$\vec{x} = \mathbf{A}^{-1} \cdot \vec{b}$$

In NumPy berechnen wir die Inverse mit `np.linalg.inv()`:

```{code-cell} python
A_inv = np.linalg.inv(A)
x_inv = A_inv @ b
print(f"Lösung via Inverse: {x_inv}")
```

Das Ergebnis ist dasselbe wie mit `np.linalg.solve()`. In der Praxis ist
`np.linalg.solve()` jedoch vorzuziehen, weil es schneller und numerisch
stabiler ist. Die Berechnung der Inversen ist aufwändiger und anfälliger für
Rundungsfehler. Die Inverse ist dann sinnvoll, wenn man dasselbe
Gleichungssystem für viele verschiedene rechte Seiten $\vec{b}$ lösen möchte,
die Koeffizientenmatrix $\mathbf{A}$ aber immer gleich bleibt.

````{admonition} Mini-Übung
:class: tip
In einer Fabrik werden drei Produkte A, B und C hergestellt. Die
Produktionskosten hängen von den Preisen dreier Materialien ab. Aus drei
bekannten Produkten lässt sich ein lineares Gleichungssystem aufstellen:

$$\begin{pmatrix} 3 & 2 & 1 \\ 0 & 1 & 1 \\ 3 & 1 & 2 \end{pmatrix}
\cdot \begin{pmatrix} M_1 \\ M_2 \\ L \end{pmatrix}
= \begin{pmatrix} 100 \\ 50 \\ 70 \end{pmatrix}$$

$M_1$, $M_2$ sind die Materialpreise in €/Einheit, $L$ ist der Lohnkostensatz
in €/Stunde.

1. Lösen Sie das Gleichungssystem mit `np.linalg.solve()`.
2. Lösen Sie es erneut mit `np.linalg.inv()`.
3. Überprüfen Sie beide Ergebnisse mit einer Probe.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

A = np.array([
    [3, 2, 1],
    [0, 1, 1],
    [3, 1, 2],
], dtype=float)

b = np.array([100, 50, 70], dtype=float)

# Lösung mit solve
x = np.linalg.solve(A, b)
print(f"Lösung (solve): M1 = {x[0]:.2f} €, M2 = {x[1]:.2f} €, L = {x[2]:.2f} €")

# Lösung mit Inverse
x_inv = np.linalg.inv(A) @ b
print(f"Lösung (inv):   M1 = {x_inv[0]:.2f} €, M2 = {x_inv[1]:.2f} €, L = {x_inv[2]:.2f} €")

# Probe
print(f"Probe: {A @ x}")
```

Ausgabe:
```
Lösung (solve): M1 = 10.00 €, M2 = 20.00 €, L = 30.00 €
Lösung (inv):   M1 = 10.00 €, M2 = 20.00 €, L = 30.00 €
Probe: [100.  50.  70.]
```

Beide Methoden liefern dasselbe Ergebnis. Eine Einheit Material 1 kostet
10 €, eine Einheit Material 2 kostet 20 € und eine Arbeitsstunde kostet
30 €. Die Probe bestätigt, dass $\mathbf{A} \cdot \vec{x} = \vec{b}$ erfüllt
ist. Dieses Gleichungssystem entspricht genau Übung 7 auf Übungsblatt 1.
````

## Lösbarkeit prüfen mit `np.linalg.det()`

Bevor wir ein lineares Gleichungssystem lösen, lohnt es sich zu prüfen, ob
überhaupt eine eindeutige Lösung existiert. Das ist genau dann der Fall, wenn
die Koeffizientenmatrix regulär ist, also eine von null verschiedene
**Determinante** besitzt.

In NumPy berechnen wir die Determinante mit `np.linalg.det()`:

```{code-cell} python
A_regulaer = np.array([
    [1.0, 0.0, 0.5],
    [0.0, 2.0, 1.0],
    [1.5, 1.0, 0.0],
])

A_singulaer = np.array([
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
    [7.0, 8.0, 9.0],
])

print(f"det(A_regulaer):  {np.linalg.det(A_regulaer):.4f}")
print(f"det(A_singulaer): {np.linalg.det(A_singulaer):.4f}")
```

Die erste Matrix hat eine Determinante ungleich null und ist damit regulär:
das Gleichungssystem hat genau eine Lösung. Die zweite Matrix hat die
Determinante null und ist **singulär**: die Zeilen sind linear abhängig und
das Gleichungssystem hat entweder keine oder unendlich viele Lösungen. Ein
Aufruf von `np.linalg.solve()` würde in diesem Fall einen Fehler liefern.

```{admonition} Determinante und Lösbarkeit
:class: note
Die Determinante einer Matrix ist ein skalarer Wert, der angibt, ob die
Matrix regulär (invertierbar) oder singulär (nicht invertierbar) ist. Für
das lineare Gleichungssystem $\mathbf{A} \cdot \vec{x} = \vec{b}$ gilt:
Ist $\det(\mathbf{A}) \neq 0$, so existiert genau eine Lösung. Ist
$\det(\mathbf{A}) = 0$, so ist die Matrix singulär und `np.linalg.solve()`
wirft einen `LinAlgError`. In der Praxis prüft man daher die Determinante
vor dem Lösen, wenn die Regularität der Matrix nicht garantiert ist.
```

````{admonition} Mini-Übung
:class: tip
Gegeben sind drei Matrizen:

```python
M1 = np.array([[2.0, 1.0], [5.0, 3.0]])
M2 = np.array([[1.0, 2.0], [2.0, 4.0]])
M3 = np.array([[3.0, 0.0, 1.0],
               [0.0, 2.0, 0.0],
               [1.0, 0.0, 3.0]])
```

1. Berechnen Sie die Determinante jeder Matrix.
2. Welche Matrizen sind regulär, welche singulär?
3. Versuchen Sie, für die singuläre Matrix das Gleichungssystem
   $\mathbf{M} \cdot \vec{x} = \vec{b}$ mit `b = np.array([1.0, 2.0])`
   zu lösen. Fangen Sie den Fehler mit `try/except` ab und geben Sie eine
   sinnvolle Fehlermeldung aus.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

M1 = np.array([[2.0, 1.0], [5.0, 3.0]])
M2 = np.array([[1.0, 2.0], [2.0, 4.0]])
M3 = np.array([[3.0, 0.0, 1.0],
               [0.0, 2.0, 0.0],
               [1.0, 0.0, 3.0]])

print(f"det(M1) = {np.linalg.det(M1):.4f}")   # regulaer
print(f"det(M2) = {np.linalg.det(M2):.4f}")   # singulaer
print(f"det(M3) = {np.linalg.det(M3):.4f}")   # regulaer

b = np.array([1.0, 2.0])
try:
    x = np.linalg.solve(M2, b)
except np.linalg.LinAlgError as e:
    print(f"Fehler: Matrix ist singulaer ({e})")
```

Ausgabe:
```
det(M1) =  1.0000
det(M2) =  0.0000
det(M3) = 16.0000
Fehler: Matrix ist singulaer (Singular matrix)
```

M2 ist singulär, weil die zweite Zeile das Doppelte der ersten ist: die
beiden Gleichungen sind linear abhängig und beschreiben dieselbe Gerade.
Das Gleichungssystem hat daher keine eindeutige Lösung. Das entspricht
genau der Situation in Übung 9 auf Übungsblatt 1, wo die Determinante
geprüft werden soll.
````

## Zusammenfassung und Ausblick

Wir haben zweidimensionale NumPy-Arrays als Matrizen kennengelernt und gelernt,
mit dem `@`-Operator Matrixmultiplikationen durchzuführen. Mit
`np.linalg.solve()` lösen wir lineare Gleichungssysteme direkt, ohne das
Gaußsche Eliminationsverfahren von Hand durchzuführen. Die Inverse einer
Matrix lässt sich mit `np.linalg.inv()` berechnen, ist aber in der Regel
nicht die effizienteste Wahl. Mit `np.linalg.det()` prüfen wir vorab, ob ein
Gleichungssystem überhaupt eindeutig lösbar ist.

Das Schwingungssignal unserer Maschine aus Kapitel 3.1 haben wir in diesem
Kapitel um eine Kalibrierungsanalyse erweitert. Im nächsten Kapitel nutzen wir
NumPy-Werkzeuge wie `np.mean()`, `np.std()` und `np.random`, um das Signal
statistisch zu beschreiben und mit realistischem Messrauschen zu versehen.
