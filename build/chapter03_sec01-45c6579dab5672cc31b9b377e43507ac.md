---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.1 Lineare Gleichungssysteme mit NumPy lösen

An einem Obststand kosten ein Apfel, eine Banane und eine Clementine je einen
festen Betrag. An drei Tagen kaufen wir verschiedene Mengen und bezahlen jeweils
einen Gesamtbetrag. Die Einzelpreise kennen wir nicht mehr, nur die Mengen und
die Kassenbons. *Wie rechnen wir die Einzelpreise daraus zurück?*

Das ist die Aufgabe eines **linearen Gleichungssystems**, kurz **LGS**. In
diesem Kapitel schreiben wir ein LGS als Matrixgleichung, prüfen mit der
Determinante, ob es eine eindeutige Lösung gibt, und berechnen sie mit einer
einzigen NumPy-Funktion. Die NumPy-Arrays aus Kapitel 2 sind dabei unser
Werkzeug, neu ist nur, dass wir jetzt mit zweidimensionalen Arrays arbeiten.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können ein lineares Gleichungssystem in der Matrixform
  $\mathbf{A} \cdot \vec{x} = \vec{b}$ aufschreiben.
* [ ] Sie können eine Matrix als zweidimensionales NumPy-Array anlegen und mit
  `A[i, j]` auf einzelne Einträge zugreifen.
* [ ] Sie können mit `np.linalg.det()` prüfen, ob ein LGS eine eindeutige
  Lösung hat, und einen `LinAlgError` mit `try`/`except` abfangen.
* [ ] Sie können ein LGS mit `np.linalg.solve()` lösen und das Ergebnis mit
  einer Probe absichern.
```

## Wie schreiben wir ein Gleichungssystem als Matrix?

Die Einkaufsmengen an drei Tagen ordnen wir in einer Tabelle an: Jede Zeile
ist ein Tag, jede Spalte eine Fruchtsorte.

| | Äpfel | Bananen | Clementinen |
| --- | --- | --- | --- |
| **Tag 1** | 3 | 2 | 1 |
| **Tag 2** | 2 | 3 | 0 |
| **Tag 3** | 1 | 1 | 3 |

Mit den unbekannten Einzelpreisen $x_A$, $x_B$, $x_C$ und den bezahlten
Beträgen liefert jeder Tag eine Gleichung:

$$\begin{align}
3 x_A + 2 x_B + 1 x_C &= 1.80 \\
2 x_A + 3 x_B + 0 x_C &= 1.20 \\
1 x_A + 1 x_B + 3 x_C &= 2.00
\end{align}$$

Alle drei linken Seiten haben dieselbe Bauart: Zahlen mal Unbekannte,
aufsummiert. Diese Zahlen, die **Koeffizienten**, schreiben wir in eine Matrix
$\mathbf{A}$, die Unbekannten in einen Vektor $\vec{x}$:

$$\mathbf{A} = \begin{pmatrix} 3 & 2 & 1 \\ 2 & 3 & 0 \\ 1 & 1 & 3 \end{pmatrix},
\qquad
\vec{x} = \begin{pmatrix} x_A \\ x_B \\ x_C \end{pmatrix}$$

Das Produkt $\mathbf{A} \cdot \vec{x}$ ist so definiert, dass genau die linken
Seiten unserer Gleichungen herauskommen. Für jede Zeile von $\mathbf{A}$
multiplizieren wir Eintrag für Eintrag mit $\vec{x}$ und addieren auf:

$$\mathbf{A} \cdot \vec{x} =
\begin{pmatrix} 3 & 2 & 1 \\ 2 & 3 & 0 \\ 1 & 1 & 3 \end{pmatrix}
\cdot \begin{pmatrix} x_A \\ x_B \\ x_C \end{pmatrix}
=
\begin{pmatrix}
3 \cdot x_A + 2 \cdot x_B + 1 \cdot x_C \\
2 \cdot x_A + 3 \cdot x_B + 0 \cdot x_C \\
1 \cdot x_A + 1 \cdot x_B + 3 \cdot x_C
\end{pmatrix}$$

Die erste Zeile von $\mathbf{A}$ trifft von oben nach unten auf $\vec{x}$:
$3$ mal $x_A$, plus $2$ mal $x_B$, plus $1$ mal $x_C$. Das ist genau die linke
Seite der ersten Gleichung. Für die zweite und dritte Zeile gilt dasselbe.

Setzen wir dieses Produkt mit dem Vektor der bezahlten Beträge gleich, steht
Zeile für Zeile wieder unser ursprüngliches Gleichungssystem:

$$\begin{pmatrix} 3 & 2 & 1 \\ 2 & 3 & 0 \\ 1 & 1 & 3 \end{pmatrix}
\cdot \begin{pmatrix} x_A \\ x_B \\ x_C \end{pmatrix}
= \begin{pmatrix} 1.80 \\ 1.20 \\ 2.00 \end{pmatrix}$$

Das ist die **Matrixgleichung** $\mathbf{A} \cdot \vec{x} = \vec{b}$. Die
**Koeffizientenmatrix** $\mathbf{A}$ enthält die Einkaufsmengen, der Vektor
$\vec{x}$ die unbekannten Preise und der Vektor $\vec{b}$ die bezahlten
Beträge. Jede Zeile von $\mathbf{A}$ gehört zu einer Gleichung, jede Spalte zu
einer Unbekannten.

In NumPy legen wir $\mathbf{A}$ als **zweidimensionales Array** an: eine Liste
von Listen, wobei jede innere Liste eine Zeile ist. Mit `dtype=float` werden
alle Einträge als Fließkommazahlen gespeichert, auch wenn wir sie als ganze
Zahlen eintippen.

```{code-cell} python
import numpy as np

# Koeffizientenmatrix: jede Zeile ist ein Einkaufstag,
# die Spalten stehen für Äpfel, Bananen, Clementinen
A = np.array([
    [3, 2, 1],
    [2, 3, 0],
    [1, 1, 3],
], dtype=float)

b = np.array([1.80, 1.20, 2.00])

print(A)
print('Form (Zeilen, Spalten):', A.shape)
```

`A.shape` gibt `(3, 3)` zurück, also drei Zeilen und drei Spalten. Auf einen
einzelnen Eintrag greifen wir mit zwei Indizes in eckigen Klammern zu:
`A[zeile, spalte]`, also zuerst die Zeile, dann die Spalte.

```{code-cell} python
print('Zeile 0, Spalte 2:', A[0, 2])   # Clementinen an Tag 1
print('Zeile 1, Spalte 0:', A[1, 0])   # Äpfel an Tag 2
```

```{admonition} Mini-Übung (✩)
:class: tip
Ein Betrieb stellt aus Stahl und Aluminium zwei Produkte her. Mit den
unbekannten Stückzahlen $n_1$ und $n_2$ lautet das Gleichungssystem:

$$\begin{align}
3 n_1 + 2 n_2 &= 12 \qquad \text{(Stahl in kg)} \\
1 n_1 + 4 n_2 &= 9 \qquad \text{(Aluminium in kg)}
\end{align}$$

1. Schreiben Sie die Koeffizientenmatrix `A_prod` und die rechte Seite
   `b_prod` als NumPy-Arrays.
2. Geben Sie `A_prod.shape` aus.
3. Beantworten Sie ohne Code: Was bedeuten die Einträge `A_prod[1, 0]` und
   `b_prod[1]` inhaltlich?
4. Rechnen Sie $\mathbf{A} \cdot \vec{n}$ von Hand aus und prüfen Sie, dass
   Zeile für Zeile wieder das obige Gleichungssystem entsteht.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

A_prod = np.array([
    [3, 2],    # Stahl:     3 kg pro Stück Produkt 1, 2 kg pro Stück Produkt 2
    [1, 4],    # Aluminium: 1 kg pro Stück Produkt 1, 4 kg pro Stück Produkt 2
], dtype=float)

b_prod = np.array([12.0, 9.0])

print(A_prod.shape)
```
`A_prod.shape` ist `(2, 2)`. Der Eintrag `A_prod[1, 0]` steht in Zeile 1
(Aluminium) und Spalte 0 (Produkt 1), er ist also der Aluminiumbedarf pro
Stück von Produkt 1, hier 1 kg. `b_prod[1]` gehört zur zweiten Gleichung und
ist die insgesamt verfügbare Aluminiummenge, hier 9 kg.

Das Produkt von Hand:

$$\mathbf{A} \cdot \vec{n} =
\begin{pmatrix} 3 & 2 \\ 1 & 4 \end{pmatrix}
\cdot \begin{pmatrix} n_1 \\ n_2 \end{pmatrix}
= \begin{pmatrix} 3 n_1 + 2 n_2 \\ 1 n_1 + 4 n_2 \end{pmatrix}$$

Gleichgesetzt mit $\vec{b} = \begin{pmatrix} 12 \\ 9 \end{pmatrix}$ ergibt das
Zeile für Zeile genau die beiden Ausgangsgleichungen.
````

## Hat das System eine eindeutige Lösung?

Nicht jedes LGS hat genau eine Lösung. Drei Fälle sind möglich:

* genau eine Lösung (der Normalfall)
* keine Lösung (die Gleichungen widersprechen sich)
* unendlich viele Lösungen (eine Gleichung liefert keine neue Information)

Für quadratische Systeme, also gleich viele Gleichungen wie Unbekannte, prüfen
wir das mit der **Determinante** $\det(\mathbf{A})$, die NumPy mit
`np.linalg.det()` berechnet. Es gilt: Ist die Determinante ungleich null, hat
das System genau eine Lösung. Ob ein Wert nahe bei null liegt, prüfen wir mit
`np.isclose()`.

```{code-cell} python
det_A = np.linalg.det(A)
print(f'Determinante: {det_A:.4f}')

# np.isclose prüft, ob ein Wert nahe bei null liegt.
# Das ist zuverlässiger als == 0, weil Rechnungen mit Fließkommazahlen
# kleine Rundungsfehler erzeugen können.
if np.isclose(det_A, 0.0):
    print('det(A) = 0: keine eindeutige Lösung.')
else:
    print('det(A) ungleich 0: genau eine Lösung.')
```

Die Determinante unserer Obststand-Matrix ist 14, also deutlich von null
verschieden. Das System hat genau eine Lösung.

Zum Vergleich eine Matrix, deren dritte Zeile die Summe der ersten beiden ist
und damit keine neue Information liefert. Ihre Determinante geben wir mit
`:.2e` in wissenschaftlicher Schreibweise aus, also als Zahl mal Zehnerpotenz.

```{code-cell} python
A_singular = np.array([
    [3, 2, 1],
    [2, 3, 0],
    [5, 5, 1],   # Zeile 3 = Zeile 1 + Zeile 2
], dtype=float)

print(f'Determinante: {np.linalg.det(A_singular):.2e}')
```

Das Ergebnis ist nicht exakt null, sondern eine winzige Zahl wie `1e-15` oder
`1e-16`, deren genauer Wert vom Rechner abhängt. Die Einträge der Matrix sind
ganze Zahlen und werden exakt gespeichert. NumPy berechnet die Determinante
aber in mehreren Zwischenschritten mit Divisionen, und nicht jedes
Zwischenergebnis lässt sich exakt als Fließkommazahl speichern. Diese kleinen
Rundungsfehler sind der Grund, warum wir mit `np.isclose` vergleichen und
nicht mit `== 0`.

````{admonition} Was passiert bei einer singulären Matrix?
:class: warning
Die Lösungsfunktion `np.linalg.solve` meldet einen `LinAlgError` nur dann,
wenn bei der Elimination eine exakte Null auftritt. Wegen der Rundungsfehler
passiert das bei `A_singular` nicht: `np.linalg.solve(A_singular, b)` liefert
ohne jede Warnung eine vermeintliche Lösung mit Einträgen in der
Größenordnung $10^{15}$. Deshalb prüfen wir die Determinante vor dem Lösen.

Tritt der Fehler doch auf, zum Beispiel bei einer Matrix, deren zweite Zeile
genau das Doppelte der ersten ist, fangen wir ihn mit `try` und `except` ab,
statt das Programm abstürzen zu lassen:

```python
A_klein = np.array([
    [1.0, 2.0],
    [2.0, 4.0],   # Zeile 2 = 2 * Zeile 1
])

try:
    x = np.linalg.solve(A_klein, np.array([1.0, 2.0]))
except np.linalg.LinAlgError:
    print('Matrix ist singulär, das System hat keine eindeutige Lösung.')
```
````

Ist die Determinante null, hat das System entweder keine oder unendlich viele
Lösungen. Welcher Fall vorliegt und wie wir die Lösbarkeit auch bei mehr
Gleichungen als Unbekannten prüfen, klären wir mit dem **Rang** im Exkurs zur
Messbrücke.

```{admonition} Wie zuverlässig ist der Determinantentest?
:class: warning
Die Regel "Determinante ungleich null, genau eine Lösung" gilt in der exakten
Mathematik. Im Rechner hängt der Wert der Determinante aber auch von der
Skalierung der Matrix ab. Multiplizieren wir die $20 \times 20$-Einheitsmatrix
mit $0.001$, sinkt ihre Determinante auf $10^{-60}$ und `np.isclose` meldet
null, obwohl sich das System genauso leicht lösen lässt wie vorher. Für unsere
kleinen, gut skalierten Beispiele reicht der Determinantentest aus.
Zuverlässigere numerische Kriterien sind der Rang und die **Konditionszahl**
(`np.linalg.cond()`), die angibt, wie stark sich kleine Fehler in den Daten
auf die Lösung auswirken.
```

```{admonition} Mini-Übung (✩)
:class: tip
Gegeben ist die Matrix

$$\mathbf{M} = \begin{pmatrix} 2 & 1 & 1 \\ 4 & 2 & 2 \\ 1 & 0 & 3 \end{pmatrix}.$$

1. Beantworten Sie ohne Code: Wird `np.linalg.det(M)` nahe bei null liegen?
   Schauen Sie sich die ersten beiden Zeilen genau an.
2. Legen Sie die Matrix an und prüfen Sie Ihre Vermutung mit
   `np.linalg.det()` und `np.isclose()`.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

M = np.array([
    [2, 1, 1],
    [4, 2, 2],
    [1, 0, 3],
], dtype=float)

det_M = np.linalg.det(M)
print(f'Determinante: {det_M:.2e}')
print('nahe null:', np.isclose(det_M, 0.0))
```
Die zweite Zeile ist genau das Doppelte der ersten Zeile und enthält daher
keine neue Information. Die Determinante ist deshalb null. Je nach
Rundungsfehlern zeigt der Rechner sie als exakt `0.00e+00` oder als winzige
Zahl nahe null an. In beiden Fällen gibt `np.isclose` den Wert `True` zurück,
das System hat keine eindeutige Lösung.
````

## Das System lösen und die Probe

Wenn die Determinante ungleich null ist, berechnen wir die Lösung mit
`np.linalg.solve`. Die Funktion erwartet zuerst die Matrix, dann die rechte
Seite.

```{code-cell} python
x = np.linalg.solve(A, b)

print(f'Preis Apfel:      {x[0]:.2f} Euro')
print(f'Preis Banane:     {x[1]:.2f} Euro')
print(f'Preis Clementine: {x[2]:.2f} Euro')
```

Ein Apfel kostet 0.30 Euro, eine Banane 0.20 Euro und eine Clementine 0.50
Euro.

*Woher wissen wir, dass dieses Ergebnis stimmt?* Wenn $\vec{x}$ die richtige
Lösung ist, muss das Matrixprodukt $\mathbf{A} \cdot \vec{x}$ wieder den Vektor
$\vec{b}$ ergeben. Das ist die **Probe**. Für das Matrixprodukt verwenden wir
den Operator `@`, nicht `*`. `np.allclose()` vergleicht anschließend alle
Einträge zweier Arrays bis auf winzige Rundungsfehler.

```{code-cell} python
b_probe = A @ x

print('A @ x:', b_probe)
print('b:    ', b)

# np.allclose prüft, ob alle Einträge bis auf winzige Rundungsfehler gleich sind
print('Probe bestanden:', np.allclose(b_probe, b))
```

```{admonition} Mini-Übung (✩)
:class: tip
Ein Café verkauft an drei Tagen Kaffee, Tee und Eis und notiert den
Tagesumsatz in Euro:

| Tag | Kaffee | Tee | Eis | Umsatz |
| --- | --- | --- | --- | --- |
| Mo | 5 | 2 | 3 | 25.60 |
| Di | 3 | 4 | 1 | 17.60 |
| Mi | 4 | 2 | 3 | 23.30 |

1. Legen Sie `A_cafe` und `b_cafe` an, prüfen Sie die Determinante, lösen Sie
   mit `np.linalg.solve` und sichern Sie das Ergebnis mit einer Probe ab.
2. Beantworten Sie ohne Code: Die Probe
   `np.allclose(A_cafe @ x_cafe, b_cafe)` ergibt `True`. Heißt das, dass
   `A_cafe` und `b_cafe` garantiert richtig aufgestellt wurden?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

A_cafe = np.array([
    [5, 2, 3],
    [3, 4, 1],
    [4, 2, 3],
], dtype=float)

b_cafe = np.array([25.60, 17.60, 23.30])

print(f'Determinante: {np.linalg.det(A_cafe):.2f}')

x_cafe = np.linalg.solve(A_cafe, b_cafe)
print(f'Kaffee: {x_cafe[0]:.2f} Euro')
print(f'Tee:    {x_cafe[1]:.2f} Euro')
print(f'Eis:    {x_cafe[2]:.2f} Euro')

print('Probe bestanden:', np.allclose(A_cafe @ x_cafe, b_cafe))
```
Die Determinante ist 10.0, das System hat also eine eindeutige Lösung: Kaffee
2.30 Euro, Tee 1.80 Euro, Eis 3.50 Euro. Die Probe bestätigt nur, dass
`x_cafe` zu dem aufgestellten `A_cafe` und `b_cafe` passt, nicht, dass `A_cafe`
und `b_cafe` selbst korrekt sind. Ein Tippfehler in `A_cafe` würde eine Lösung
liefern, die die Probe ebenfalls besteht. Deshalb lohnt es sich, `A_cafe` und
`b_cafe` vor dem Lösen noch einmal auszugeben und zu kontrollieren.
````

## Zusammenfassung und Ausblick

Ein lineares Gleichungssystem schreiben wir als Matrixgleichung
$\mathbf{A} \cdot \vec{x} = \vec{b}$. Die Koeffizientenmatrix $\mathbf{A}$
legen wir als zweidimensionales NumPy-Array an, die rechte Seite $\vec{b}$ als
eindimensionales Array. Bei kleinen, gut skalierten Matrizen prüfen wir mit
`np.linalg.det()` und `np.isclose()` vorab, ob eine eindeutige Lösung
existiert, und fangen einen `LinAlgError` mit `try`/`except` ab. Die Lösung
selbst liefert `np.linalg.solve(A, b)` in einer Zeile, abgesichert durch die
Probe `np.allclose(A @ x, b)`.

Im nächsten Kapitel verlassen wir den Obststand und stellen ein LGS aus einem
Maschinenbau-Problem auf: dem statischen Gleichgewicht an einem belasteten
Träger. Das Vorgehen bleibt gleich, nur kommen die Gleichungen jetzt aus der
Mechanik.
