---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.2 LGS lösen mit NumPy

Im letzten Notebook haben wir gelernt, ein lineares Gleichungssystem als
Matrixgleichung $\mathbf{A} \cdot \vec{x} = \vec{b}$ aufzuschreiben und
zu prüfen, ob es eine eindeutige Lösung gibt. Jetzt lösen wir es.

Wir bleiben beim Obstmarkt-Beispiel: drei Früchte, drei Einkaufstage,
drei bekannte Gesamtpreise — gesucht sind die Einzelpreise. NumPy löst
dieses System in einer einzigen Zeile. Danach lernen wir, warum es zwei
Wege zur Lösung gibt, wann man welchen wählt, und wie man das Ergebnis
immer mit einer **Probe** absichert.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können ein LGS mit `np.linalg.solve(A, b)` lösen.
* [ ] Sie können das Ergebnis mit einer Probe ($\mathbf{A} \cdot \vec{x} = \vec{b}$)
  überprüfen.
* [ ] Sie können die Inverse einer Matrix mit `np.linalg.inv()` berechnen
  und verstehen, wann ihr Einsatz sinnvoll ist.
* [ ] Sie kennen einen robusten Lösungs-Workflow mit Lösbarkeitstest und Probe.
```

```{code-cell} python
import numpy as np
```

## Lösen mit `np.linalg.solve`

Wir starten direkt mit unserem Obstmarkt-System aus dem letzten Notebook.
Die Koeffizientenmatrix und die rechte Seite sind bereits bekannt:

```{code-cell} python
# Koeffizientenmatrix: Zeile = Einkaufstag, Spalte = Fruchtsorte
A = np.array([
    [3, 2, 1],
    [2, 3, 0],
    [1, 1, 3],
], dtype=float)

# Rechte Seite: bezahlte Gesamtbeträge in Euro
b = np.array([2.10, 1.70, 2.80])

# Lösung: x = [Preis Apfel, Preis Banane, Preis Orange]
x = np.linalg.solve(A, b)

print(f'Preis Apfel:  {x[0]:.2f} €')
print(f'Preis Banane: {x[1]:.2f} €')
print(f'Preis Orange: {x[2]:.2f} €')
```

`np.linalg.solve(A, b)` löst das System $\mathbf{A} \cdot \vec{x} = \vec{b}$
direkt. Intern verwendet NumPy dabei eine sogenannte **LU-Zerlegung**: die
Matrix wird in ein Produkt aus einer unteren und einer oberen Dreiecksmatrix
zerlegt, was das Lösen deutlich effizienter macht als das Gaußsche
Eliminationsverfahren von Hand.

## Die Probe: Ergebnis immer absichern

Ein numerisches Ergebnis sollte immer überprüft werden. Für ein LGS ist die
Probe einfach: Wenn $\vec{x}$ die Lösung ist, muss $\mathbf{A} \cdot \vec{x}$
exakt $\vec{b}$ ergeben:

```{code-cell} python
b_probe = A @ x

print(f'Probe  A @ x: {b_probe}')
print(f'Soll       b: {b}')
print(f'Übereinstimmung: {np.allclose(b_probe, b)}')
```

`np.allclose(b_probe, b)` gibt `True` zurück, wenn alle Elemente beider
Arrays bis auf winzige Rundungsfehler übereinstimmen. Das ist die robustere
Alternative zu einem elementweisen `==`-Vergleich.

````{admonition} Mini-Übung
:class: tip
Lösen Sie das Café-System aus dem letzten Notebook und überprüfen Sie
das Ergebnis mit einer Probe. Geben Sie die Preise für Kaffee, Tee und
Kuchen formatiert aus.

| Tag | Kaffee | Tee | Kuchen | Umsatz (€) |
|-----|--------|-----|--------|------------|
| Mo  | 5      | 2   | 3      | 24,50      |
| Di  | 3      | 4   | 1      | 17,30      |
| Mi  | 6      | 1   | 4      | 28,10      |
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

A = np.array([[5,2,3],[3,4,1],[6,1,4]], dtype=float)
b = np.array([24.50, 17.30, 28.10])

x = np.linalg.solve(A, b)

print(f'Kaffee: {x[0]:.2f} €')
print(f'Tee:    {x[1]:.2f} €')
print(f'Kuchen: {x[2]:.2f} €')
print(f'Probe: {np.allclose(A @ x, b)}')
```

Ausgabe:
```
Kaffee: 2.30 €
Tee:    1.80 €
Kuchen: 3.50 €
Probe: True
```
````

+++

## Die Inverse einer Matrix

Es gibt einen zweiten Weg zur Lösung. Aus der Matrixgleichung
$\mathbf{A} \cdot \vec{x} = \vec{b}$ folgt durch Multiplikation mit
$\mathbf{A}^{-1}$ von links:

$$\vec{x} = \mathbf{A}^{-1} \cdot \vec{b}$$

$\mathbf{A}^{-1}$ heißt die **inverse Matrix** zu $\mathbf{A}$. Sie ist
so definiert, dass ihr Produkt mit $\mathbf{A}$ die Einheitsmatrix ergibt:

$$\mathbf{A}^{-1} \cdot \mathbf{A} = \mathbf{A} \cdot \mathbf{A}^{-1} = \mathbf{I}$$

Die Einheitsmatrix $\mathbf{I}$ ist das Matrix-Äquivalent der Zahl 1:
Sie hat Einsen auf der Hauptdiagonale und Nullen überall sonst.

```{code-cell} python
# Obstmarkt-System
A = np.array([[3,2,1],[2,3,0],[1,1,3]], dtype=float)
b = np.array([2.10, 1.70, 2.80])

# Inverse berechnen und prüfen
A_inv = np.linalg.inv(A)
print('Überprüfung A_inv @ A (sollte Einheitsmatrix sein):')
print(np.round(A_inv @ A, 10))

# Lösung über die Inverse
x_inv = A_inv @ b
print(f'\nLösung via Inverse: {x_inv}')
print(f'Probe: {np.allclose(A @ x_inv, b)}')
```

Das Ergebnis ist identisch mit dem von `np.linalg.solve`.

```{admonition} solve oder inv — wann was?
:class: note
**In der Regel: `np.linalg.solve`.**

`solve` ist schneller und numerisch stabiler als `inv`, weil es die Matrix
nicht explizit invertiert, sondern direkt die LU-Zerlegung zur Lösung
nutzt. Die Berechnung der Inversen ist rechenaufwändig und anfälliger
für Rundungsfehler.

**Ausnahme: `np.linalg.inv` ist sinnvoll**, wenn man dasselbe System
$\mathbf{A} \cdot \vec{x} = \vec{b}$ für **viele verschiedene** rechte
Seiten $\vec{b}$ lösen muss, die Koeffizientenmatrix $\mathbf{A}$ aber
immer gleich bleibt. In diesem Fall berechnet man $\mathbf{A}^{-1}$ einmal
und multipliziert dann nur noch mit jedem neuen $\vec{b}$.
```

+++

## Was tun, wenn `solve` einen Fehler wirft?

Wenn $\det(\mathbf{A}) = 0$ ist — das System also keine eindeutige Lösung
hat — wirft `np.linalg.solve` einen `LinAlgError`. Das sollte man abfangen:

```{code-cell} python
# Singuläre Matrix (Determinante = 0)
A_sing = np.array([[1,2],[2,4]], dtype=float)
b_sing = np.array([3., 6.])

try:
    x = np.linalg.solve(A_sing, b_sing)
    print(f'Lösung: {x}')
except np.linalg.LinAlgError as e:
    print(f'Fehler: {e}')
    print('→ Matrix ist singulär, kein eindeutiges Ergebnis.')
```

```{admonition} Praxis-Tipp
:class: note
In der Ingenieurpraxis überprüft man deshalb immer zuerst die Determinante
oder den Rang, bevor man `solve` aufruft — genau wie wir es in Notebook
3.1 gelernt haben. Ein robustes Programm prüft zuerst die Lösbarkeit und
gibt eine verständliche Fehlermeldung aus, statt mit einem `LinAlgError`
abzustürzen.
```

Zusammengefasst sieht ein robuster Lösungs-Workflow so aus:

```{code-cell} python
def lgs_loesen(A, b):
    """Löst das LGS A·x = b mit Lösbarkeitstest und Probe.

    A: Koeffizientenmatrix (n×n NumPy-Array)
    b: rechte Seite (NumPy-Array der Länge n)
    Rückgabe: Lösungsvektor x oder None bei singulärer Matrix
    """
    det = np.linalg.det(A)
    if np.isclose(det, 0.0):
        print('Keine eindeutige Lösung (det = 0).')
        return None

    x = np.linalg.solve(A, b)

    if not np.allclose(A @ x, b):
        print('Warnung: Probe fehlgeschlagen!')

    return x


# Test mit Obstmarkt-System
A = np.array([[3,2,1],[2,3,0],[1,1,3]], dtype=float)
b = np.array([2.10, 1.70, 2.80])

x = lgs_loesen(A, b)
if x is not None:
    print(f'Apfel: {x[0]:.2f} €, Banane: {x[1]:.2f} €, Orange: {x[2]:.2f} €')
```

+++

## Zusammenfassung und Ausblick

`np.linalg.solve(A, b)` löst ein lineares Gleichungssystem direkt und ist
die Standardmethode. Die Probe $\mathbf{A} \cdot \vec{x} \approx \vec{b}$
mit `np.allclose` sichert das Ergebnis ab. Die Inverse `np.linalg.inv(A)`
ist eine Alternative, wenn dieselbe Matrix $\mathbf{A}$ für viele
verschiedene rechte Seiten wiederverwendet wird. Ein robuster Workflow prüft
immer zuerst die Lösbarkeit.

Im nächsten Notebook verlassen wir den Obstmarkt und wenden das Gelernte
auf ein echtes Ingenieurproblem an: die Temperaturberechnung in einer
mehrschichtigen Wand.
