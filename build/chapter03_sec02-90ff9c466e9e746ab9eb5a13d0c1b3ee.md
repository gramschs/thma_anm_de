---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.2 Vertiefung: Auflagerkräfte an einem Träger

In Kapitel 3.1 haben wir ein Gleichungssystem als Matrixgleichung geschrieben,
seine Lösbarkeit geprüft und es mit `np.linalg.solve` gelöst. In diesem Kapitel
wenden wir dasselbe Vorgehen auf ein Problem aus der Technischen Mechanik an:
die Auflagerkräfte eines belasteten Trägers. Bearbeiten Sie die Teilaufgaben
möglichst zu zweit und der Reihe nach, jeder Teil baut auf dem vorherigen auf.

````{admonition} Projekt: Auflagerkräfte an einem Träger (✩✩)
:class: tip
Ein waagerechter Träger der Länge $L = 4\,\text{m}$ ist links im Punkt A durch
ein **Festlager** und rechts im Punkt B durch ein **Loslager** gelagert. Das
Festlager kann eine waagerechte Kraft $A_x$ und eine senkrechte Kraft $A_y$
aufnehmen, das Loslager nur eine senkrechte Kraft $B_y$. Die x-Achse zeigt nach
rechts, die y-Achse nach oben, der Ursprung liegt in A.

Auf den Träger wirken:

* eine **Seilkraft** im Abstand $1\,\text{m}$ von A, mit den Komponenten
  $6\,\text{kN}$ nach rechts und $8\,\text{kN}$ nach oben,
* eine **Last** $F = 12\,\text{kN}$ senkrecht nach unten im Abstand
  $3\,\text{m}$ von A.

Gesucht sind die drei Auflagerkräfte $A_x$, $A_y$ und $B_y$.
````

```{admonition} Teil 1: Gleichgewichtsbedingungen aufstellen
:class: tip
Am starren Träger gelten drei Gleichgewichtsbedingungen: Die Summe aller
waagerechten Kräfte ist null, die Summe aller senkrechten Kräfte ist null, und
die Summe aller Momente um den Punkt A ist null.

Stellen Sie die drei Gleichungen mit den Zahlenwerten aus dem Projektkopf auf.
Verwenden Sie die Vorzeichenkonvention: Kräfte nach rechts und nach oben sind
positiv, Momente gegen den Uhrzeigersinn sind positiv. Die Seilkraft greift auf
Höhe der Trägerachse an, ihre waagerechte Komponente erzeugt daher kein Moment
um A.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
Summe der waagerechten Kräfte:

$$A_x + 6 = 0$$

Summe der senkrechten Kräfte:

$$A_y + B_y + 8 - 12 = 0 \quad\Longrightarrow\quad A_y + B_y = 4$$

Summe der Momente um A (Hebelarm mal Kraft, gegen den Uhrzeigersinn positiv):

$$4 \cdot B_y + 1 \cdot 8 - 3 \cdot 12 = 0 \quad\Longrightarrow\quad 4\,B_y = 28$$

Die senkrechte Seilkomponente ($8\,\text{kN}$ nach oben im Abstand $1\,\text{m}$)
erzeugt ein positives Moment, die Last ($12\,\text{kN}$ nach unten im Abstand
$3\,\text{m}$) ein negatives.
````

```{admonition} Teil 2: In Matrixform bringen und Lösbarkeit prüfen
:class: tip
Fassen Sie die drei Gleichungen aus Teil 1 zur Matrixgleichung
$\mathbf{A} \cdot \vec{x} = \vec{b}$ zusammen, mit dem Unbekanntenvektor
$\vec{x} = (A_x,\ A_y,\ B_y)^\top$. Legen Sie `A` als zweidimensionales Array
und `b` als eindimensionales Array an und prüfen Sie mit der Determinante, ob
das System eine eindeutige Lösung hat.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 2
:class: tip
:class: dropdown
```python
import numpy as np

# Unbekannte: x = [A_x, A_y, B_y]
A = np.array([
    [1, 0, 0],   # Summe Fx:  1*A_x + 0*A_y + 0*B_y = -6
    [0, 1, 1],   # Summe Fy:  0*A_x + 1*A_y + 1*B_y =  4
    [0, 0, 4],   # Summe M_A: 0*A_x + 0*A_y + 4*B_y = 28
], dtype=float)

b = np.array([-6.0, 4.0, 28.0])

det_A = np.linalg.det(A)
print(f'Determinante: {det_A:.1f}')
print('eindeutig lösbar:', not np.isclose(det_A, 0.0))
```
Die Determinante ist 4.0 und damit ungleich null. Das System hat genau eine
Lösung. Der Träger ist **statisch bestimmt**: Er hat genau so viele
Auflagerreaktionen, wie zur Fesselung nötig sind.
````

```{admonition} Teil 3: Lösen und Probe
:class: tip
Lösen Sie das System mit `np.linalg.solve` und sichern Sie das Ergebnis mit
einer Probe ab. Geben Sie die drei Auflagerkräfte in kN aus.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 3
:class: tip
:class: dropdown
```python
x = np.linalg.solve(A, b)

print(f'A_x = {x[0]:.1f} kN')
print(f'A_y = {x[1]:.1f} kN')
print(f'B_y = {x[2]:.1f} kN')

print('Probe bestanden:', np.allclose(A @ x, b))
```
Die Lösung lautet $A_x = -6.0\,\text{kN}$, $A_y = -3.0\,\text{kN}$,
$B_y = 7.0\,\text{kN}$. Die Probe ist bestanden.
````

```{admonition} Teil 4: Ergebnis interpretieren
:class: tip
Beantworten Sie in eigenen Worten:

1. Was bedeutet das negative Vorzeichen von $A_x$ für die Richtung der
   waagerechten Auflagerkraft?
2. Auch $A_y$ ist negativ. In welche Richtung zeigt die senkrechte
   Auflagerkraft am Festlager, und wie passt das zur angreifenden Seilkraft?
```

````{admonition} Lösung Teil 4
:class: tip
:class: dropdown
1. Wir hatten $A_x$ als Kraft nach rechts angesetzt. Das Ergebnis
   $A_x = -6\,\text{kN}$ bedeutet, dass die tatsächliche Auflagerkraft mit
   $6\,\text{kN}$ nach links zeigt. Sie hält der waagerechten Seilkomponente
   das Gleichgewicht.
2. $A_y = -3\,\text{kN}$ bedeutet, dass die senkrechte Auflagerkraft am
   Festlager nach unten zeigt. Das Festlager hält den Träger an dieser Stelle
   also fest. Der Grund ist die Seilkraft: Sie zieht mit $8\,\text{kN}$ nach
   oben, das ist in der Nähe von A mehr, als zum Gleichgewicht nötig wäre, und
   der Träger würde ohne das Festlager an dieser Stelle abheben.
````

```{admonition} Abschlussfrage
:class: tip
Angenommen, bei A säße statt des Festlagers ein Loslager, das nur eine
senkrechte Kraft **nach oben** aufnehmen kann (es kann drücken, aber nicht
ziehen). Was würde mit dem Träger passieren? Nutzen Sie Ihr Ergebnis aus
Teil 3.
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
Aus Teil 3 wissen wir, dass die senkrechte Auflagerkraft bei A nach unten
zeigt ($A_y = -3\,\text{kN}$). Ein Loslager, das nur drücken kann, ist dazu
nicht in der Lage. Der Träger würde bei A abheben und sich um das Lager B
drehen, bis er an einem anderen Bauteil anschlägt oder herunterfällt. Er wäre
nicht mehr im Gleichgewicht. Das rechnerische Modell mit drei Unbekannten
setzt also stillschweigend voraus, dass das Festlager Kräfte in beide
Richtungen aufnehmen kann.
````

````{admonition} Zusatzaufgabe: Ein Träger ohne waagerechte Fesselung (✩✩✩)
:class: tip
Jetzt sind **beide** Lager Loslager, die nur senkrechte Kräfte aufnehmen. Es
gibt daher nur noch zwei unbekannte Auflagerkräfte, $A_y$ und $B_y$, aber
weiterhin drei Gleichgewichtsbedingungen. Die Belastung bleibt unverändert.

1. Schreiben Sie die drei Gleichungen mit den zwei Unbekannten als
   $\mathbf{A} \cdot \vec{x} = \vec{b}$. Die Matrix `A` hat dann drei Zeilen
   und zwei Spalten.
2. Versuchen Sie, das System mit `np.linalg.solve` zu lösen. Was meldet
   Python?
3. Betrachten Sie die erste Gleichung ($\sum F_x = 0$) für sich allein. Ist
   sie erfüllbar?
4. Erklären Sie physikalisch, warum dieser Träger nicht im Gleichgewicht sein
   kann.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Zusatzaufgabe
:class: tip
:class: dropdown
```python
import numpy as np

# Unbekannte: x = [A_y, B_y]
A = np.array([
    [0, 0],   # Summe Fx:  0*A_y + 0*B_y = -6
    [1, 1],   # Summe Fy:  1*A_y + 1*B_y =  4
    [0, 4],   # Summe M_A: 0*A_y + 4*B_y = 28
], dtype=float)

b = np.array([-6.0, 4.0, 28.0])

print('Form von A:', A.shape)

try:
    x = np.linalg.solve(A, b)
except np.linalg.LinAlgError as fehler:
    print('solve schlägt fehl:', fehler)
```
`np.linalg.solve` verlangt eine quadratische Matrix und bricht bei der
Form `(3, 2)` mit einem `LinAlgError` ab.

Die erste Gleichung lautet $0 \cdot A_y + 0 \cdot B_y = -6$, also $0 = -6$.
Das ist ein Widerspruch, keine Wahl von $A_y$ und $B_y$ kann ihn erfüllen. Das
System hat **keine Lösung**.

Physikalisch heißt das: Auf den Träger wirkt mit der waagerechten
Seilkomponente eine Kraft von $6\,\text{kN}$ nach rechts, aber keines der
beiden Loslager kann eine waagerechte Gegenkraft aufbringen. Der Träger würde
nach rechts wegrutschen, er ist ein **verschiebliches System** und nicht im
Gleichgewicht. Für die statische Berechnung brauchen wir mindestens ein Lager,
das waagerechte Kräfte aufnimmt.
````
