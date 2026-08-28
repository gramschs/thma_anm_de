---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.1 Lineare Gleichungssysteme mit NumPy lösen

An einem Obststand kostet ein Apfel, eine Banane und eine Clementine je einen
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

Die Einkaufsmengen an drei Tagen ordnen wir in einer Tabelle: Jede Zeile ist
ein Tag, jede Spalte eine Fruchtsorte.

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

Setzen wir dieses Produkt mit dem Vektor der Kassenbons gleich, steht Zeile für
Zeile wieder unser ursprüngliches Gleichungssystem:

$$\begin{pmatrix} 3 & 2 & 1 \\ 2 & 3 & 0 \\ 1 & 1 & 3 \end{pmatrix}
\cdot \begin{pmatrix} x_A \\ x_B \\ x_C \end{pmatrix}
= \begin{pmatrix} 1.80 \\ 1.20 \\ 2.00 \end{pmatrix}$$

Das ist die **Matrixgleichung** $\mathbf{A} \cdot \vec{x} = \vec{b}$. Die
**Koeffizientenmatrix** $\mathbf{A}$ enthält die Einkaufsmengen, der Vektor
$\vec{x}$ die unbekannten Preise und der Vektor $\vec{b}$ die Kassenbons. Jede
Zeile von $\mathbf{A}$ gehört zu einer Gleichung, jede Spalte zu einer
Unbekannten.

In NumPy legen wir $\mathbf{A}$ als **zweidimensionales Array** an: eine Liste
von Listen, wobei jede innere Liste eine Zeile ist.

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
einzelnen Eintrag greifen wir mit zwei Indizes zu: zuerst die Zeile, dann die
Spalte.

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

1. Schreiben Sie die Koeffizientenmatrix `A` und die rechte Seite `b` als
   NumPy-Arrays.
2. Geben Sie `A.shape` aus.
3. Beantworten Sie ohne Code: Was bedeuten die Einträge `A[1, 0]` und `b[1]`
   inhaltlich?
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

A = np.array([
    [3, 2],    # Stahl:     3 kg pro Stück Produkt 1, 2 kg pro Stück Produkt 2
    [1, 4],    # Aluminium: 1 kg pro Stück Produkt 1, 4 kg pro Stück Produkt 2
], dtype=float)

b = np.array([12.0, 9.0])

print(A.shape)
```
`A.shape` ist `(2, 2)`. Der Eintrag `A[1, 0]` steht in Zeile 1 (Aluminium) und
Spalte 0 (Produkt 1), er ist also der Aluminiumbedarf pro Stück von Produkt 1,
hier 1 kg. `b[1]` gehört zur zweiten Gleichung und ist die insgesamt verfügbare
Aluminiummenge, hier 9 kg.

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
wir das mit der **Determinante** $\det(\mathbf{A})$. Es gilt: Ist die
Determinante ungleich null, hat das System genau eine Lösung.

```{code-cell} python
det_A = np.linalg.det(A)
print(f'Determinante: {det_A:.4f}')

# np.isclose prüft, ob ein Wert nahe bei null liegt.
# Das ist zuverlässiger als == 0, weil Fließkommazahlen nie exakt sind.
if np.isclose(det_A, 0.0):
    print('det(A) = 0: keine eindeutige Lösung.')
else:
    print('det(A) ungleich 0: genau eine Lösung.')
```

Zum Vergleich eine Matrix, deren dritte Zeile die Summe der ersten beiden ist
und damit keine neue Information liefert:

```{code-cell} python
A_singular = np.array([
    [3, 2, 1],
    [2, 3, 0],
    [5, 5, 1],   # Zeile 3 = Zeile 1 + Zeile 2
], dtype=float)

print(f'Determinante: {np.linalg.det(A_singular):.2e}')
```

Das Ergebnis ist nicht exakt null, sondern eine winzige Zahl in der
Größenordnung `1e-16`. Fließkommazahlen werden im Rechner nur näherungsweise
gespeichert, und jede Rechnung sammelt kleine Rundungsfehler an. Genau deshalb
vergleichen wir mit `np.isclose` und nicht mit `== 0`.

````{admonition} Was passiert bei einer singulären Matrix?
:class: warning
Übergeben wir eine singuläre Matrix an die Lösungsfunktion `np.linalg.solve`,
bricht das Programm mit einem `LinAlgError` ab. Solche Fehler fangen wir mit
`try` und `except` ab, statt das Programm abstürzen zu lassen:

```python
try:
    x = np.linalg.solve(A_singular, b)
except np.linalg.LinAlgError:
    print('Matrix ist singulär, das System hat keine eindeutige Lösung.')
```
````

Ist die Determinante null, hat das System entweder keine oder unendlich viele
Lösungen. Welcher Fall vorliegt und wie man die Lösbarkeit auch bei mehr
Gleichungen als Unbekannten prüft, klären wir mit dem **Rang** im Exkurs zur
Messbrücke.

```{admonition} Mini-Übung (✩)
:class: tip
Gegeben ist die Matrix

$$\mathbf{A} = \begin{pmatrix} 2 & 1 & 1 \\ 4 & 2 & 2 \\ 1 & 0 & 3 \end{pmatrix}.$$

1. Beantworten Sie ohne Code: Wird `np.linalg.det(A)` nahe bei null liegen?
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

A = np.array([
    [2, 1, 1],
    [4, 2, 2],
    [1, 0, 3],
], dtype=float)

det_A = np.linalg.det(A)
print(f'Determinante: {det_A:.2e}')
print('nahe null:', np.isclose(det_A, 0.0))
```
Die zweite Zeile ist genau das Doppelte der ersten Zeile und enthält daher
keine neue Information. Die Determinante ist deshalb null, im Rechner
erscheint sie als winzige Zahl nahe null. `np.isclose` gibt `True` zurück,
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

*Woher wissen wir, dass dieses Ergebnis stimmt?* Wenn $\vec{x}$ die richtige
Lösung ist, muss das Matrixprodukt $\mathbf{A} \cdot \vec{x}$ wieder den Vektor
$\vec{b}$ ergeben. Das ist die **Probe**. Für das Matrixprodukt verwenden wir
den Operator `@`, nicht `*`.

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

1. Legen Sie `A` und `b` an, prüfen Sie die Determinante, lösen Sie mit
   `np.linalg.solve` und sichern Sie das Ergebnis mit einer Probe ab.
2. Beantworten Sie ohne Code: Die Probe `np.allclose(A @ x, b)` ergibt `True`.
   Heißt das, dass `A` und `b` garantiert richtig aufgestellt wurden?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

A = np.array([
    [5, 2, 3],
    [3, 4, 1],
    [4, 2, 3],
], dtype=float)

b = np.array([25.60, 17.60, 23.30])

print(f'Determinante: {np.linalg.det(A):.2f}')

x = np.linalg.solve(A, b)
print(f'Kaffee: {x[0]:.2f} Euro')
print(f'Tee:    {x[1]:.2f} Euro')
print(f'Eis:    {x[2]:.2f} Euro')

print('Probe bestanden:', np.allclose(A @ x, b))
```
Die Determinante ist 10.0, das System hat also eine eindeutige Lösung: Kaffee
2.30 Euro, Tee 1.80 Euro, Eis 3.50 Euro. Die Probe bestätigt nur, dass `x` zu
dem aufgestellten `A` und `b` passt, nicht, dass `A` und `b` selbst korrekt
sind. Ein Tippfehler in `A` würde eine Lösung liefern, die die Probe ebenfalls
besteht. Deshalb lohnt es sich, `A` und `b` vor dem Lösen noch einmal
auszugeben und zu kontrollieren.
````

## Zusammenfassung und Ausblick

Ein lineares Gleichungssystem schreiben wir als Matrixgleichung
$\mathbf{A} \cdot \vec{x} = \vec{b}$. Die Koeffizientenmatrix $\mathbf{A}$
legen wir als zweidimensionales NumPy-Array an, die rechte Seite $\vec{b}$ als
eindimensionales Array. Mit `np.linalg.det()` und `np.isclose()` prüfen wir
vorab, ob eine eindeutige Lösung existiert, und fangen den `LinAlgError` mit
`try`/`except` ab. Die Lösung selbst liefert `np.linalg.solve(A, b)` in einer
Zeile, abgesichert durch die Probe `np.allclose(A @ x, b)`.

Im nächsten Kapitel verlassen wir den Obststand und stellen ein LGS aus einem
Maschinenbau-Problem auf: dem statischen Gleichgewicht an einem belasteten
Rahmen. Das Vorgehen bleibt gleich, nur kommen die Gleichungen jetzt aus der
Mechanik.
