---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.2 LGS lösen mit NumPy

Im letzten Kapitel haben wir das Obstmarkt-Problem als Matrixgleichung
$\mathbf{A} \cdot \vec{x} = \vec{b}$ aufgeschrieben und mit der Determinante
geprüft, ob eine eindeutige Lösung existiert. Jetzt berechnen wir sie.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können ein LGS mit `np.linalg.solve(A, b)` lösen.
* [ ] Sie können das Ergebnis mit einer Probe absichern.
* [ ] Sie wissen, wann `np.linalg.inv()` eine sinnvolle Alternative ist.
```

+++

## Lösen mit `np.linalg.solve`

Wir starten sofort mit dem Code. Das Obstmarkt-System aus Kapitel 3.1 kennen
wir bereits:

```{code-cell} python
import numpy as np

# Koeffizientenmatrix: Zeile = Einkaufstag, Spalte = Fruchtsorte
A = np.array([
    [3, 2, 1],
    [2, 3, 0],
    [1, 1, 3],
], dtype=float)

# Rechte Seite: bezahlte Gesamtbetraege in Euro
b = np.array([2.10, 1.70, 2.80])

# np.linalg.solve loest A * x = b in einer einzigen Zeile
# Intern verwendet NumPy eine LU-Zerlegung (schneller als Gauss von Hand)
# Wichtig: solve erwartet immer zuerst A, dann b
x = np.linalg.solve(A, b)

# x ist ein 1D-Array: x[0]=Apfel, x[1]=Banane, x[2]=Clementinen
print(f'Preis Apfel:  {x[0]:.2f} €')
print(f'Preis Banane: {x[1]:.2f} €')
print(f'Preis Clementine: {x[2]:.2f} €')
```

+++

## Das Ergebnis immer prüfen: die Probe

*Woher wissen wir, dass das Ergebnis stimmt?* Wenn $\vec{x}$ die richtige
Lösung ist, muss das Matrixprodukt $\mathbf{A} \cdot \vec{x}$ exakt den
Vektor $\vec{b}$ reproduzieren. Das ist unsere **Probe**:

```{code-cell} python
# Probe: A mal x muss gleich b ergeben
# @ ist der Matrixmultiplikations-Operator in NumPy (nicht * verwenden!)
b_probe = A @ x

print('Ergebnis A @ x:', b_probe)
print('Soll-Wert   b: ', b)

# np.allclose prueft, ob alle Einträge bis auf winzige Rundungsfehler gleich sind
# Hintergrund: Fließkommazahlen sind nie exakt, daher schlägt == oft fehl
# allclose toleriert Abweichungen kleiner als 1e-8 (Standard-Schwellenwert)
print('Probe bestanden:', np.allclose(b_probe, b))
```

## Arrays kopieren mit `.copy()`

Bevor wir die Probe genauer untersuchen, brauchen wir ein weiteres Konzept.
*Was passiert, wenn wir ein Array einer neuen Variable zuweisen und dann
einen Eintrag ändern?*

```{code-cell} python
# ACHTUNG: einfache Zuweisung erstellt keine Kopie, sondern eine zweite
# Referenz auf dasselbe Array im Speicher
A_original = np.array([[3, 2, 1], [2, 3, 0], [1, 1, 3]], dtype=float)
A_referenz  = A_original          # keine Kopie!
A_referenz[0, 0] = 99.0           # aendert auch A_original mit

print('A_original[0,0]:', A_original[0, 0])  # 99.0 -> unbeabsichtigt geaendert!

# RICHTIG: .copy() erstellt eine echte, unabhaengige Kopie
A_original = np.array([[3, 2, 1], [2, 3, 0], [1, 1, 3]], dtype=float)
A_kopie    = A_original.copy()    # echte Kopie
A_kopie[0, 0] = 99.0              # aendert nur A_kopie

print('A_original[0,0]:', A_original[0, 0])  # 3.0 -> unveraendert
print('A_kopie[0,0]:   ', A_kopie[0, 0])     # 99.0
```

`.copy()` ist immer dann nötig, wenn wir ein Array verändern wollen, ohne
das Original zu beschädigen. Das ist ein häufiger Fehler, der schwer zu
finden ist, weil kein Fehler geworfen wird.

```{admonition} Mini-Übung
:class: tip
Legen Sie mit `.copy()` eine Kopie von `A` an, nennen Sie sie `A_falsch`,
und ersetzen Sie darin `A_falsch[0, 0]` durch `4` statt `3`. Lösen Sie das
System mit `A_falsch` und führen Sie die Probe durch. Was gibt
`np.allclose(A_falsch @ x_falsch, b)` aus – und was bedeutet das für die
Verlässlichkeit der Probe?
```

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

A = np.array([[3, 2, 1], [2, 3, 0], [1, 1, 3]], dtype=float)
b = np.array([2.10, 1.70, 2.80])

A_falsch = A.copy()        # Kopie anlegen, um A nicht zu veraendern
A_falsch[0, 0] = 4.0       # absichtlicher Tippfehler
x_falsch = np.linalg.solve(A_falsch, b)

print('Probe mit falschem A:', np.allclose(A_falsch @ x_falsch, b))  # True!
print('Probe mit richtigem A:', np.allclose(A @ x_falsch, b))        # False
```

Die Probe mit `A_falsch` besteht, weil `x_falsch` zur *falschen* Matrix
passt – nicht zur richtigen. Die Probe prüft nur Konsistenz, nicht
Korrektheit. Deshalb lohnt es sich, `A` und `b` vor dem Lösen nochmals
auszugeben und visuell zu kontrollieren.
````

+++

## Die Inverse: eine Alternative für Spezialfälle (optional)

```{admonition} Optionaler Abschnitt
:class: note
Dieser Abschnitt ist für das Verständnis von Kapitel 3.3 nicht erforderlich.
Er lohnt sich, wenn Sie verstehen möchten, wann `np.linalg.inv` gegenüber
`np.linalg.solve` sinnvoll ist.
```

Es gibt einen zweiten Weg zur Lösung. Aus
$\mathbf{A} \cdot \vec{x} = \vec{b}$ folgt durch Multiplikation mit der
**inversen Matrix** $\mathbf{A}^{-1}$ von links:

$$\vec{x} = \mathbf{A}^{-1} \cdot \vec{b}$$

Die Inverse ist so definiert, dass $\mathbf{A}^{-1} \cdot \mathbf{A}$
die **Einheitsmatrix** $\mathbf{I}$ ergibt: Einsen auf der Diagonale,
Nullen überall sonst. Das ist das Matrix-Äquivalent der Aussage
$\frac{1}{a} \cdot a = 1$:

```{code-cell} python
# Inverse berechnen
A_inv = np.linalg.inv(A)

# Probe: A_inv @ A soll die Einheitsmatrix ergeben
# np.round auf 10 Stellen, damit winzige Rundungsfehler nicht stoeren
print('A_inv @ A:')
print(np.round(A_inv @ A, 10))

# Loesung ueber die Inverse: dieselbe wie mit solve
x_via_inv = A_inv @ b
print('Loesung via Inverse:', np.round(x_via_inv, 2))
```

```{admonition} solve oder inv?
:class: note
**Normalfall: `np.linalg.solve`.** Es ist schneller und numerisch stabiler,
weil es die Matrix nicht explizit invertiert.

**Ausnahme: `np.linalg.inv`** lohnt sich, wenn dasselbe System
$\mathbf{A} \cdot \vec{x} = \vec{b}$ für viele verschiedene rechte Seiten
$\vec{b}$ gelöst werden muss, die Matrix $\mathbf{A}$ aber immer gleich
bleibt. Dann berechnet man $\mathbf{A}^{-1}$ einmal und multipliziert
danach nur noch.
```

```{admonition} Mini-Übung
:class: tip
1. Die Einheitsmatrix hat Einsen auf der Diagonale und Nullen sonst.
   Erzeugen Sie sie direkt mit `np.eye(3)` und vergleichen Sie sie mit
   dem Ergebnis von `A_inv @ A`. Stimmen sie überein?
2. Ein Café betreibt drei Filialen. In jeder Filiale gelten dieselben
   Preise (dieselbe Matrix `A`), aber die Tagesumsätze `b` sind
   verschieden. Nennen Sie in einem Satz, welche Methode hier effizienter
   ist: `solve` dreimal aufrufen oder `inv` einmal berechnen.
3. Was gibt `np.linalg.inv(A_sing)` aus, wenn `A_sing` singulär ist?
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

A = np.array([[3, 2, 1], [2, 3, 0], [1, 1, 3]], dtype=float)
A_inv = np.linalg.inv(A)

# Frage 1: np.eye erzeugt die Einheitsmatrix
einheitsmatrix = np.eye(3)
print(einheitsmatrix)
print('Gleich?', np.allclose(A_inv @ A, einheitsmatrix))  # True

# Frage 3: Inverse einer singulaeren Matrix
# try/except wie oben eingefuehrt: LinAlgError bei singulaerer Matrix
A_sing = np.array([[1, 2], [2, 4]], dtype=float)
try:
    print(np.linalg.inv(A_sing))
except np.linalg.LinAlgError:
    print('Fehler: Matrix ist singulaer.')   # Singular matrix
```

Für die drei Filialen mit derselben Matrix `A` ist `inv` effizienter:
$\mathbf{A}^{-1}$ wird einmal berechnet, danach reicht für jede Filiale
eine einfache Matrix-Vektor-Multiplikation. Für singuläre Matrizen wirft
auch `inv` einen `LinAlgError`.
````

+++

```{admonition} Mini-Übung
:class: tip
Lösen Sie das Café-System aus Kapitel 3.1 vollständig: Legen Sie `A` und
`b` an, führen Sie den Determinanten-Test durch, lösen Sie mit `solve` und
sichern Sie das Ergebnis mit einer Probe ab.

| Tag | Kaffee | Tee | Eis    | Umsatz (€) |
|-----|--------|-----|--------|------------|
| Mo  | 5      | 2   | 3      | 25,60      |
| Di  | 3      | 4   | 1      | 17,60      |
| Mi  | 4      | 2   | 3      | 23,30      |

Beantworten Sie außerdem: Welcher Schritt aus diesem Kapitel könnte bei
einem größeren System mit 50 Unbekannten besonders lange dauern?
```

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# Koeffizientenmatrix
A = np.array([
    [5, 2, 3],
    [3, 4, 1],
    [4, 2, 3],
], dtype=float)

# Rechte Seite
b = np.array([25.60, 17.60, 23.30])

# Schritt 1: Lösbarkeit pruefen
det_A = np.linalg.det(A)
print(f'Determinante: {det_A:.2f}')   # 10.0 -> eindeutige Loesung

# Schritt 2: loesen
x = np.linalg.solve(A, b)
print(f'Kaffee: {x[0]:.2f} €')
print(f'Tee:    {x[1]:.2f} €')
print(f'Eis:    {x[2]:.2f} €')

# Schritt 3: Probe
print('Probe bestanden:', np.allclose(A @ x, b))
```

Ausgabe: Kaffee 2,30 €, Tee 1,80 €, Eis 3,50 €. Die Probe ist bestanden.
Bei einem System mit 50 Unbekannten würde die Berechnung der Inversen
`np.linalg.inv` am längsten dauern, da sie rechenaufwändiger ist als
`solve`. Wenn keine weiteren rechten Seiten folgen, ist `solve` die bessere
Wahl.
````

+++

## Zusammenfassung und Ausblick

`np.linalg.solve(A, b)` löst ein LGS in einer Zeile. Die Probe
`np.allclose(A @ x, b)` sichert jedes Ergebnis ab. Die Inverse
`np.linalg.inv(A)` ist sinnvoll, wenn dieselbe Matrix für viele
verschiedene rechte Seiten wiederverwendet wird.

Im nächsten Kapitel verlassen wir den Obststand und wenden das Gelernte
auf ein reales Ingenieurproblem an: die Berechnung von Temperaturen in einer
mehrschichtigen Wand. Das Prinzip ist identisch, aber die Gleichungen kommen
jetzt aus der Physik.
