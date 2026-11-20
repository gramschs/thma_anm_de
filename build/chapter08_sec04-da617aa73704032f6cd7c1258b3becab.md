---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 8.4 Kalibrierung eines Kraftsensors

In Kapitel 8.3 haben wir mit dem Gradientenverfahren zwei Koordinaten eines
Standorts gesucht. Jetzt suchen Sie zwei Parameter eines Modells, das wie in
Kapitel 8.2 zu Messdaten passen soll. Bearbeiten Sie die Teile möglichst zu
zweit und der Reihe nach.

````{admonition} Projekt: Wie genau misst die Wägezelle? (✩✩)
:class: tip
An einer Abfüllanlage wiegt eine **Wägezelle** jedes Gebinde. Sie gibt eine
Spannung aus, die mit der Kraft auf die Zelle wächst. Laut Datenblatt gilt die
**Kennlinie**

$$U = m \cdot F + b$$

mit der Empfindlichkeit $m = 2.0\,\text{V/kN}$ und dem Nullpunkt
$b = 0\,\text{V}$. Weil jede Zelle etwas anders ist, wird sie nach dem Einbau
**kalibriert**. Dazu belastet ein Prüfer sie mit Prüfgewichten von $0$ bis
$2\,\text{kN}$ und notiert die Spannung.

| Größe | Bedeutung | Einheit |
| --- | --- | --- |
| $F$ | Kraft auf die Wägezelle | kN |
| $U$ | Ausgangsspannung | V |
| $m$ | Empfindlichkeit, Steigung der Kennlinie | V/kN |
| $b$ | Nullpunkt, Spannung ohne Last | V |

Gesucht sind die tatsächlichen Werte von $m$ und $b$ und die Frage, wie stark
sich eine Messung mit der Kennlinie aus dem Datenblatt verschätzt.
````

Führen Sie zuerst die folgende Zelle aus. Sie enthält die Funktionen
`gradient()` und `gradientenverfahren_2d()` aus Kapitel 8.3, die Messdaten und
die Kennlinie aus dem Datenblatt.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Funktionen aus Kapitel 8.3
def gradient(funktion, stelle, dx):
    """Gradient von funktion an der Stelle stelle = [x, y].

    Jede partielle Ableitung mit dem zentralen Differenzenquotienten.
    """
    versatz_x = np.array([dx, 0.0])
    versatz_y = np.array([0.0, dx])
    ableitung_x = (funktion(stelle + versatz_x)
                   - funktion(stelle - versatz_x)) / (2 * dx)
    ableitung_y = (funktion(stelle + versatz_y)
                   - funktion(stelle - versatz_y)) / (2 * dx)
    return np.array([ableitung_x, ableitung_y])

def gradientenverfahren_2d(funktion, start, lernrate, toleranz,
                           max_iterationen):
    """Sucht ein Minimum von funktion, Start bei start = [x, y].

    Gibt alle besuchten Stellen als Array zurück, Zeile = [x, y].
    """
    stelle = start
    verlauf = [stelle]
    for k in range(max_iterationen):
        steigung = gradient(funktion, stelle, 1e-6)
        # Abbruchkriterium: Länge des Gradienten fast null
        if np.linalg.norm(steigung) < toleranz:
            break
        stelle = stelle - lernrate * steigung
        verlauf.append(stelle)
    return np.array(verlauf)

# Kalibrierung: Kraft der Prüfgewichte in kN und gemessene Spannung in V
kraft = np.array([0.0, 0.2, 0.4, 0.6, 0.8, 1.0, 1.2, 1.4, 1.6, 1.8, 2.0])
spannung = np.array([0.121, 0.437, 0.904, 1.293, 1.703, 2.116, 2.488,
                     2.931, 3.327, 3.818, 4.165])

# Kennlinie laut Datenblatt: [m in V/kN, b in V]
datenblatt = np.array([2.0, 0.0])
```

````{admonition} Teil 1: Messdaten und Datenblatt
:class: tip
Schreiben Sie eine Funktion `kennlinie(kraft, parameter)`, die die Spannung
nach der Kennlinie berechnet. Wie in Kapitel 8.3 der Standort ist `parameter`
ein Array mit zwei Einträgen, hier `[m, b]`.

Zeichnen Sie die Messwerte als Punkte und die Kennlinie aus dem Datenblatt als
Linie. Passt das Datenblatt zu der eingebauten Zelle? Welcher der beiden
Parameter weicht vor allem ab?
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 1
:class: tip
:class: dropdown
`parameter[0]` ist die Steigung $m$, `parameter[1]` der Nullpunkt $b$. Die
Zeichnung entsteht wie in Teil 1 von Kapitel 8.2.
```python
def kennlinie(kraft, parameter):
    """Spannung in V für die Kraft in kN, parameter = [m, b]."""
    return ___

fig, ax = plt.subplots(figsize=(7, 4))
ax.scatter(kraft, spannung, color='black', label='Messwerte')
ax.plot(kraft, ___, label='Datenblatt')
ax.set_xlabel('Kraft in kN')
ax.set_ylabel('Spannung in V')
ax.set_title('Kalibrierung der Wägezelle')
ax.legend()
ax.grid(True)
plt.show()
```
````

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
```python
def kennlinie(kraft, parameter):
    """Spannung in V für die Kraft in kN, parameter = [m, b]."""
    return parameter[0] * kraft + parameter[1]

fig, ax = plt.subplots(figsize=(7, 4))
ax.scatter(kraft, spannung, color='black', label='Messwerte')
ax.plot(kraft, kennlinie(kraft, datenblatt), label='Datenblatt')
ax.set_xlabel('Kraft in kN')
ax.set_ylabel('Spannung in V')
ax.set_title('Kalibrierung der Wägezelle')
ax.legend()
ax.grid(True)
plt.show()
```
Die Messwerte liegen fast auf einer Geraden, aber durchweg etwas über der
Kennlinie aus dem Datenblatt. Schon ohne Last zeigt die Zelle $0.121\,\text{V}$
statt $0\,\text{V}$. Vor allem der Nullpunkt $b$ weicht also ab. Ob auch die
Steigung etwas anders ist, lässt sich mit bloßem Auge kaum entscheiden.
````

````{admonition} Teil 2: Kostenfunktion und Höhenlinienbild
:class: tip
Schreiben Sie wie in Teil 2 von Kapitel 8.2 eine Funktion `kosten(parameter)`,
die die mittlere quadratische Abweichung zwischen gemessener Spannung und
Kennlinie berechnet. Wie groß ist sie für das Datenblatt?

Zeichnen Sie wie im ersten Abschnitt von Kapitel 8.3 ein Höhenlinienbild der
Kostenfunktion für $m$ von $1.8$ bis $2.3\,\text{V/kN}$ und $b$ von $-0.2$ bis
$0.3\,\text{V}$, je 101 Werte. Mit `levels=[0.002, 0.005, 0.01, 0.02, 0.05]`
erscheinen gut lesbare Linien. Markieren Sie das Datenblatt.

Wo etwa liegt das Minimum? Welche Form hat das Tal?
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 2
:class: tip
:class: dropdown
Auf der waagerechten Achse liegt $m$, auf der senkrechten $b$. Die Zeilen der
Matrix gehören deshalb zu $b$, die Spalten zu $m$.
```python
def kosten(parameter):
    """Mittlere quadratische Abweichung in V² zwischen Messung und Kennlinie."""
    abweichung = ___
    return np.mean(___)

kosten_datenblatt = kosten(datenblatt)
print(f'Kosten für das Datenblatt: {kosten_datenblatt:.4f} V²')

# Kosten auf einem Gitter, Zeile = b, Spalte = m
m_werte = np.linspace(1.8, 2.3, 101)
b_werte = np.linspace(-0.2, 0.3, 101)
kosten_werte = np.zeros((len(b_werte), len(m_werte)))
for i in range(len(b_werte)):
    for j in range(len(m_werte)):
        parameter = np.array([___, ___])
        kosten_werte[i, j] = ___

fig, ax = plt.subplots(figsize=(7, 6))
linien = ax.contour(m_werte, b_werte, kosten_werte,
                    levels=[0.002, 0.005, 0.01, 0.02, 0.05])
ax.clabel(linien, fontsize=8)
ax.scatter(datenblatt[0], datenblatt[1], marker='x', s=80, color='red',
           label='Datenblatt')
ax.set_xlabel('m in V/kN')
ax.set_ylabel('b in V')
ax.set_title('Mittlere quadratische Abweichung in V²')
ax.legend()
plt.show()
```
````

````{admonition} Lösung Teil 2
:class: tip
:class: dropdown
```python
def kosten(parameter):
    """Mittlere quadratische Abweichung in V² zwischen Messung und Kennlinie."""
    abweichung = spannung - kennlinie(kraft, parameter)
    return np.mean(abweichung**2)

kosten_datenblatt = kosten(datenblatt)
print(f'Kosten für das Datenblatt: {kosten_datenblatt:.4f} V²')

# Kosten auf einem Gitter, Zeile = b, Spalte = m
m_werte = np.linspace(1.8, 2.3, 101)
b_werte = np.linspace(-0.2, 0.3, 101)
kosten_werte = np.zeros((len(b_werte), len(m_werte)))
for i in range(len(b_werte)):
    for j in range(len(m_werte)):
        parameter = np.array([m_werte[j], b_werte[i]])
        kosten_werte[i, j] = kosten(parameter)

fig, ax = plt.subplots(figsize=(7, 6))
linien = ax.contour(m_werte, b_werte, kosten_werte,
                    levels=[0.002, 0.005, 0.01, 0.02, 0.05])
ax.clabel(linien, fontsize=8)
ax.scatter(datenblatt[0], datenblatt[1], marker='x', s=80, color='red',
           label='Datenblatt')
ax.set_xlabel('m in V/kN')
ax.set_ylabel('b in V')
ax.set_title('Mittlere quadratische Abweichung in V²')
ax.legend()
plt.show()
```
Ausgabe:

```text
Kosten für das Datenblatt: 0.0159 V²
```
Das Minimum liegt bei etwa $m = 2.05\,\text{V/kN}$ und $b = 0.07\,\text{V}$.
Das Tal ist nicht rund wie in Kapitel 8.3, sondern lang, schmal und schräg. Es
verläuft von links oben nach rechts unten. Größere Werte von $m$ passen also
fast genauso gut, wenn $b$ gleichzeitig kleiner wird.
````

````{admonition} Teil 3: Gradientenverfahren und Vergleich mit NumPy
:class: tip
Bestimmen Sie $m$ und $b$ mit `gradientenverfahren_2d()`. Starten Sie beim
Datenblatt, mit der Lernrate $0.3$, der Toleranz $10^{-4}$ und höchstens 1000
Iterationen. Zeichnen Sie den Weg des Verfahrens in das Höhenlinienbild aus
Teil 2.

Für eine Gerade gibt es eine Formel, die die mittlere quadratische Abweichung
direkt minimiert. NumPy stellt sie als `np.polyfit(kraft, spannung, 1)` bereit.
Die `1` steht für ein Polynom ersten Grades, also eine Gerade. Die Funktion
gibt die Steigung und den Achsenabschnitt als Array zurück. Vergleichen Sie
mit Ihrem Ergebnis.

Probieren Sie auch die Lernraten $0.45$ und $0.5$ aus. Was passiert?
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 3
:class: tip
:class: dropdown
Der Weg entsteht wie im dritten Abschnitt von Kapitel 8.3. Spalte 0 von
`verlauf` enthält die Werte von $m$, Spalte 1 die Werte von $b$.
```python
# Gradientenverfahren vom Datenblatt aus
verlauf = gradientenverfahren_2d(___, datenblatt, 0.3, 1e-4, 1000)
parameter_optimal = ___
anzahl = len(verlauf) - 1
print(f'{anzahl} Iterationen')
print('Gradientenverfahren [m, b]:', np.round(parameter_optimal, 4))

# Vergleich mit np.polyfit()
parameter_polyfit = np.polyfit(___, ___, 1)
print('np.polyfit()        [m, b]:', np.round(parameter_polyfit, 4))

# Weg im Höhenlinienbild
fig, ax = plt.subplots(figsize=(7, 6))
linien = ax.contour(m_werte, b_werte, kosten_werte,
                    levels=[0.002, 0.005, 0.01, 0.02, 0.05])
ax.clabel(linien, fontsize=8)
ax.plot(___, ___, marker='o', markersize=4, color='red',
        label='Weg des Verfahrens')
ax.set_xlabel('m in V/kN')
ax.set_ylabel('b in V')
ax.set_title('Gradientenverfahren für die Kennlinie')
ax.legend()
plt.show()
```
````

````{admonition} Lösung Teil 3
:class: tip
:class: dropdown
```python
# Gradientenverfahren vom Datenblatt aus
verlauf = gradientenverfahren_2d(kosten, datenblatt, 0.3, 1e-4, 1000)
parameter_optimal = verlauf[-1]
anzahl = len(verlauf) - 1
print(f'{anzahl} Iterationen')
print('Gradientenverfahren [m, b]:', np.round(parameter_optimal, 4))

# Vergleich mit np.polyfit()
parameter_polyfit = np.polyfit(kraft, spannung, 1)
print('np.polyfit()        [m, b]:', np.round(parameter_polyfit, 4))

# Weg im Höhenlinienbild
fig, ax = plt.subplots(figsize=(7, 6))
linien = ax.contour(m_werte, b_werte, kosten_werte,
                    levels=[0.002, 0.005, 0.01, 0.02, 0.05])
ax.clabel(linien, fontsize=8)
ax.plot(verlauf[:, 0], verlauf[:, 1], marker='o', markersize=4, color='red',
        label='Weg des Verfahrens')
ax.set_xlabel('m in V/kN')
ax.set_ylabel('b in V')
ax.set_title('Gradientenverfahren für die Kennlinie')
ax.legend()
plt.show()

# Größere Lernraten
for lernrate in [0.45, 0.5]:
    verlauf_test = gradientenverfahren_2d(kosten, datenblatt, lernrate,
                                          1e-4, 1000)
    anzahl_test = len(verlauf_test) - 1
    print(f'Lernrate {lernrate}: {anzahl_test} Iterationen, '
          f'[m, b] = {np.round(verlauf_test[-1], 4)}')
```
Ausgabe:

```text
39 Iterationen
Gradientenverfahren [m, b]: [2.049  0.0694]
np.polyfit()        [m, b]: [2.0488 0.0696]
Lernrate 0.45: 1000 Iterationen, [m, b] = [2.0417 0.0638]
Lernrate 0.5: 1000 Iterationen, [m, b] = [-2.32175471e+09 -2.77992362e+09]
```
Nach 39 Iterationen findet das Verfahren $m = 2.049\,\text{V/kN}$ und
$b = 0.069\,\text{V}$. Das stimmt mit `np.polyfit()` bis auf wenige
Zehntausendstel überein, den Unterschied bestimmt die Toleranz. Der erste
Schritt führt quer über das schmale Tal hinweg. Danach läuft der Weg in kleinen
Zickzackschritten den Talboden entlang zum Minimum.

Mit der Lernrate $0.45$ kommt das Verfahren zwar in die Nähe des Minimums,
springt aber quer zum schmalen Tal hin und her und erfüllt das
Abbruchkriterium nie. Mit $0.5$ schaukeln sich die Sprünge auf, die Parameter
wachsen auf Milliarden. Das schmale Tal ist quer zu seiner Richtung sehr
steil, deshalb verträgt es nur kleine Lernraten.
````

````{admonition} Teil 4: Was bringt die Kalibrierung?
:class: tip
Im Betrieb zeigt die Wägezelle $2.73\,\text{V}$. Lösen Sie die Kennlinie nach
der Kraft auf:

$$F = \frac{U - b}{m}.$$

Berechnen Sie die Kraft einmal mit dem Datenblatt und einmal mit den
kalibrierten Parametern aus Teil 3. Um wie viel Newton und um wie viel Prozent
verschätzt sich das Datenblatt?
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 4
:class: tip
:class: dropdown
Eine Kraft von $1\,\text{kN}$ sind $1000\,\text{N}$. Die prozentuale Abweichung
beziehen wir auf die kalibrierte Kraft.
```python
# Kraft aus der gemessenen Spannung
u_betrieb = 2.73
kraft_datenblatt = (u_betrieb - ___) / ___
kraft_kalibriert = (u_betrieb - ___) / ___

# Abweichung in N und in Prozent
abweichung_newton = ___
abweichung_prozent = ___
print(f'Kraft laut Datenblatt: {kraft_datenblatt:.4f} kN')
print(f'Kraft kalibriert:      {kraft_kalibriert:.4f} kN')
print(f'Abweichung: {abweichung_newton:.0f} N, {abweichung_prozent:.1f} %')
```
````

````{admonition} Lösung Teil 4
:class: tip
:class: dropdown
```python
# Kraft aus der gemessenen Spannung
u_betrieb = 2.73
kraft_datenblatt = (u_betrieb - datenblatt[1]) / datenblatt[0]
kraft_kalibriert = (u_betrieb - parameter_optimal[1]) / parameter_optimal[0]

# Abweichung in N und in Prozent
abweichung_newton = (kraft_datenblatt - kraft_kalibriert) * 1000
abweichung_prozent = abweichung_newton / (kraft_kalibriert * 1000) * 100
print(f'Kraft laut Datenblatt: {kraft_datenblatt:.4f} kN')
print(f'Kraft kalibriert:      {kraft_kalibriert:.4f} kN')
print(f'Abweichung: {abweichung_newton:.0f} N, {abweichung_prozent:.1f} %')
```
Ausgabe:

```text
Kraft laut Datenblatt: 1.3650 kN
Kraft kalibriert:      1.2985 kN
Abweichung: 67 N, 5.1 %
```
Mit dem Datenblatt läge die angezeigte Kraft um $67\,\text{N}$ zu hoch, das
sind rund $6.8\,\text{kg}$ Gewicht oder $5\,\%$. Für eine Abfüllanlage wäre
das viel zu ungenau. Den größten Teil des Fehlers verursacht der Nullpunkt
von $0.07\,\text{V}$, der ohne Kalibrierung gar nicht berücksichtigt würde.
````

```{admonition} Abschlussfrage
:class: tip
Warum ist das Tal im Höhenlinienbild schmal und schräg? Denken Sie an die
Messpunkte: Was passiert mit der Geraden, wenn wir $m$ etwas vergrößern und
$b$ gleichzeitig etwas verkleinern?
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
Vergrößern wir $m$, wird die Gerade steiler. Verkleinern wir gleichzeitig $b$,
rutscht sie nach unten. Beides zusammen dreht die Gerade ungefähr um einen
Punkt in der Mitte der Messwerte, bei etwa $1\,\text{kN}$. In der Mitte passt
sie dann noch fast genauso gut, nur an den Enden weicht sie etwas mehr ab.
Deshalb steigen die Kosten in dieser Richtung nur langsam an, und das Tal
erstreckt sich von links oben nach rechts unten. Verschieben wir dagegen nur
$b$, rutscht die ganze Gerade an allen Messpunkten gleichzeitig weg. Quer zum
Tal steigen die Kosten deshalb steil an.

Für das Gradientenverfahren bedeutet das: Quer zum Tal ist die Steigung groß
und begrenzt die Lernrate. Entlang des Tals ist sie klein, und mit dieser
kleinen Lernrate kommt das Verfahren dort nur langsam voran. Je schmaler das
Tal, desto mehr Iterationen braucht das Verfahren.
````

````{admonition} Zusatzaufgabe: Kraft in Newton statt Kilonewton (✩✩✩)
:class: tip
Ein Kollege rechnet lieber in Newton. Er multipliziert die Kräfte mit 1000 und
startet beim Datenblatt, das jetzt $m = 0.002\,\text{V/N}$ und
$b = 0\,\text{V}$ lautet.

1. Schreiben Sie Kennlinie und Kostenfunktion für die Kraft in Newton.
   Berechnen Sie den Gradienten am Datenblatt und vergleichen Sie seine beiden
   Komponenten mit dem Gradienten am Datenblatt in Kilonewton.
2. Starten Sie das Gradientenverfahren mit der Lernrate $0.3$. Suchen Sie dann
   die größte Lernrate der Form $1 \cdot 10^{-k}$ oder $4 \cdot 10^{-k}$, mit
   der das Verfahren nicht explodiert, und lassen Sie es 2000 Iterationen
   laufen. Vergleichen Sie das Ergebnis mit `np.polyfit()`.
3. Erklären Sie, warum die Wahl der Einheit über Erfolg und Misserfolg des
   Verfahrens entscheidet, obwohl die Gerade dieselbe ist.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Zusatzaufgabe
:class: tip
:class: dropdown
1. Nur die Kraft und das Datenblatt ändern sich:

   ```python
   # Teilaufgabe 1: Kraft in N, Datenblatt in V/N und V
   kraft_newton = kraft * 1000
   datenblatt_newton = np.array([0.002, 0.0])

   def kosten_newton(parameter):
       """Mittlere quadratische Abweichung mit der Kraft in N."""
       abweichung = spannung - kennlinie(kraft_newton, parameter)
       return np.mean(abweichung**2)

   print('Gradient in kN:', gradient(kosten, datenblatt, 1e-6))
   print('Gradient in N: ', gradient(kosten_newton, datenblatt_newton, 1e-6))
   ```

   Ausgabe:

   ```text
   Gradient in kN: [-0.27596364 -0.23690909]
   Gradient in N:  [-2.75963636e+02 -2.36909091e-01]
   ```

   Die Komponente für $b$ ist gleich geblieben. Die Komponente für $m$ ist
   tausendmal so groß. Eine Änderung von $m$ um $1\,\text{V/N}$ ist eine
   riesige Änderung, denn sie wird mit Kräften bis $2000\,\text{N}$
   multipliziert.
2. Wir probieren die Lernraten der Reihe nach aus:

   ```python
   # Teilaufgabe 2: verschiedene Lernraten mit der Kraft in N
   for lernrate in [0.3, 1e-6, 4e-7]:
       verlauf_newton = gradientenverfahren_2d(kosten_newton,
                                               datenblatt_newton, lernrate,
                                               1e-4, 2000)
       anzahl_newton = len(verlauf_newton) - 1
       print(f'Lernrate {lernrate:.0e}: {anzahl_newton} Iterationen, '
             f'[m, b] = {verlauf_newton[-1]}')
   print('np.polyfit():', np.polyfit(kraft_newton, spannung, 1))
   ```

   Ausgabe:

   ```text
   Lernrate 3e-01: 3 Iterationen, [m, b] = [ 5.81958985e+13 -4.96732955e+04]
   Lernrate 1e-06: 57 Iterationen, [m, b] = [ 3.70621357e+10 -8.19482317e+05]
   Lernrate 4e-07: 2000 Iterationen, [m, b] = [2.09853567e-03 3.18968452e-05]
   np.polyfit(): [0.00204882 0.06963636]
   ```

   Mit $0.3$ und mit $10^{-6}$ explodiert das Verfahren. Dass es trotzdem
   nach drei beziehungsweise 57 Iterationen anhält, ist eine Falle. Die
   Parameter sind dann so groß, dass sich `stelle + 1e-6` wegen der
   Rundungsfehler aus Kapitel 7.2 nicht mehr von `stelle` unterscheidet. Der
   zentrale Differenzenquotient wird exakt null, und das Abbruchkriterium
   meldet ein Minimum, das keines ist. Ein erfülltes Abbruchkriterium allein
   beweist also nichts, wir müssen uns das Ergebnis immer ansehen. Erst mit
   $4 \cdot 10^{-7}$ bleibt das Verfahren stabil. Nach 2000 Iterationen steht
   $b$ aber
   noch immer bei $0.00003\,\text{V}$ statt bei $0.07\,\text{V}$. Auch $m$
   liegt mit $0.00210\,\text{V/N}$ daneben, weil das Verfahren den zu kleinen
   Nullpunkt mit einer zu steilen Geraden ausgleicht.
3. Die Gerade ist dieselbe, aber das Tal der Kostenfunktion sieht in den neuen
   Einheiten anders aus. In Richtung $m$ ist es tausendmal steiler als in
   Kilonewton, in Richtung $b$ gleich geblieben. Die Lernrate muss so klein
   sein, dass die steile Richtung nicht explodiert. Mit dieser kleinen
   Lernrate kommt das Verfahren in der flachen Richtung $b$ kaum noch voran.
   Deshalb wählen wir Einheiten, in denen alle gesuchten Parameter ungefähr
   dieselbe Größenordnung haben. Im maschinellen Lernen heißt dieser Schritt
   **Skalierung** der Daten und gehört vor jedes Training.
````
