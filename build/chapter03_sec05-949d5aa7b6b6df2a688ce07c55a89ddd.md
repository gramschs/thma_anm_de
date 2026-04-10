---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.5 Anwendung: Modellierung einer Messbrücke

Bisher haben wir LGS aus Energiebilanzen hergeleitet, zum Beispiel beim
Wärmedurchgang durch eine Mehrschichtwand. Dasselbe Vorgehen funktioniert
für elektrische Schaltungen: Statt Energiebilanzgleichungen nutzen wir die
**Kirchhoffschen Regeln**. In diesem Kapitel leiten wir das LGS für eine
**Wheatstone-Brückenschaltung** her — eine Schaltung, die in der Messtechnik
zur präzisen Messung kleiner Widerstandsänderungen eingesetzt wird, etwa
bei Dehnungsmessstreifen oder Temperatursensoren.

*Wie groß ist der Querstrom durch die Brücke, wenn einer der Widerstände
vom Sollwert abweicht?*

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können die Kirchhoffsche **Knotenregel** und **Maschenregel**
  auf eine einfache Schaltung anwenden und ein LGS
  $\mathbf{A} \cdot \vec{x} = \vec{b}$ daraus ablesen.
* [ ] Sie können das LGS als Python-Funktion implementieren, die den
  Querstrom $I$ für ein gegebenes $R_4$ zurückgibt.
* [ ] Sie können den Sonderfall der abgeglichenen Brücke physikalisch
  begründen und numerisch überprüfen.
```

+++

## Die Schaltung

Die Brückenschaltung besteht aus vier Widerständen und einem
Brückenwiderstand $R_L$. Von oben fließt der Gesamtstrom $I_0$; er teilt
sich in einen linken Zweig ($I_1$, $I_2$) und einen rechten Zweig
($I_3$, $I_4$) auf. Der Querstrom $I$ fließt durch $R_L$.

```{figure} pics/wheatstone_bruecke.svg
:alt: Wheatstone-Brückenschaltung mit eingezeichneten Stromrichtungen
:align: center

Wheatstone-Brückenschaltung mit den definierten Zählpfeilrichtungen.
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

**Gegebene Größen:**
$R_1 = R_2 = R_3 = 100\,\Omega$, $R_L = 10\,\Omega$, $U_0 = 10\,\text{V}$.
Der Messwiderstand $R_4$ ist variabel.

Der Lösungsvektor enthält alle sechs Ströme:
$\vec{x} = (I_0,\, I_1,\, I_2,\, I_3,\, I_4,\, I)^T$.

+++

## Knotenregel: die ersten drei Gleichungen

Wir beginnen mit der **Knotenregel** (1. Kirchhoffsches Gesetz): An
jedem Knoten ist die Summe der zufließenden Ströme gleich der Summe der
abfließenden. Das folgt aus der Ladungserhaltung: Ladung kann sich an
einem Knoten nicht ansammeln.

Jeder Knoten liefert eine Gleichung. Zufließende Ströme erhalten
ein positives Vorzeichen, abfließende ein negatives:

```{code-cell} python
import numpy as np

# --- Gegebene Größen ---
R1 = 100.   # Ohm
R2 = 100.   # Ohm
R3 = 100.   # Ohm
R4 = 200.   # Ohm  (Testwert: Messwiderstand vom Sollwert abgewichen)
RL = 10.    # Ohm  (Brückenwiderstand)
U0 = 10.    # V

# Unbekannte: x = [I0, I1, I2, I3, I4, I]  (6 Ströme, 6 Gleichungen)

# --- Knotenregel (1. Kirchhoffsches Gesetz) ---
# Knotengleichung: zufließende Ströme (+) = abfließende Ströme (-)
# rechte Seite b = 0 (keine externe Quelle an einem Knoten)
#
# K1  oberer Knoten: I0 fließt zu, I1 und I3 fließen ab
#     I0 - I1 - I3 = 0
#     Koeffizienten [I0, I1, I2, I3, I4,  I]: [+1, -1,  0, -1,  0,  0]
#
# K2  Knoten Mitte links: I1 fließt zu, I2 und I fließen ab
#     I1 - I2 - I = 0
#     Koeffizienten:                           [ 0, +1, -1,  0,  0, -1]
#
# K3  Knoten Mitte rechts: I3 und I fließen zu, I4 fließt ab
#     I3 + I - I4 = 0
#     Koeffizienten:                           [ 0,  0,  0, +1, -1, +1]

A_knoten = np.array([
    [+1., -1.,  0., -1.,  0.,  0.],   # K1
    [ 0., +1., -1.,  0.,  0., -1.],   # K2
    [ 0.,  0.,  0., +1., -1., +1.],   # K3
])
print('Knotengleichungen (erste 3 Zeilen von A):')
print(A_knoten)
```

## Maschenregel: die restlichen drei Gleichungen

Die **Maschenregel** (2. Kirchhoffsches Gesetz) sagt: Die Summe aller
Spannungsabfälle entlang einer geschlossenen Masche ist gleich der
Quellenspannung. Das folgt aus der Energieerhaltung. Der
Spannungsabfall an einem Widerstand berechnet sich nach dem Ohmschen
Gesetz als $U = R \cdot I$.

Wir durchlaufen jede Masche im Uhrzeigersinn. Durchläuft man einen
Widerstand in Richtung des definierten Stroms, ergibt das einen
positiven Beitrag; entgegen der Stromrichtung einen negativen:

```{code-cell} python
# --- Maschenregel (2. Kirchhoffsches Gesetz) ---
# Maschengleichung: Summe der Spannungsabfälle = Quellenspannung
# Spannungsabfall: U = R * I (Ohmsches Gesetz)
# Widerstand in Stromrichtung durchlaufen:        +R * I
# Widerstand entgegen der Stromrichtung:           -R * I
#
# M1  linke Masche (Uhrzeigersinn: Quelle -> R1 -> R2):
#     R1*I1 + R2*I2 = U0
#     Koeffizienten [I0,  I1,  I2,  I3,  I4,  I]: [ 0,  R1,  R2,   0,   0,   0]  | b = U0
#
# M2  rechte Masche (Uhrzeigersinn: Quelle -> R3 -> R4):
#     R3*I3 + R4*I4 = U0
#     Koeffizienten:                               [ 0,   0,   0,  R3,  R4,   0]  | b = U0
#
# M3  Quermasche (Uhrzeigersinn: R1 -> RL -> R3, keine Quelle):
#     R1 in Richtung I1:  +R1*I1
#     RL entgegen I:      -RL*I   (wir laufen entgegen der definierten I-Richtung)
#     R3 entgegen I3:     -R3*I3
#     R1*I1 - R3*I3 - RL*I = 0
#     Koeffizienten:                               [ 0,  R1,   0, -R3,   0, -RL]  | b = 0

A = np.array([
    [+1., -1.,  0., -1.,   0.,   0.],   # K1
    [ 0., +1., -1.,  0.,   0.,  -1.],   # K2
    [ 0.,  0.,  0., +1.,  -1.,  +1.],   # K3
    [ 0.,  R1,  R2,  0.,   0.,   0.],   # M1
    [ 0.,  0.,  0.,  R3,   R4,   0.],   # M2
    [ 0.,  R1,  0., -R3,   0.,  -RL],   # M3
])
b = np.array([0., 0., 0., U0, U0, 0.])

# --- Lösbarkeit prüfen und lösen ---
print(f'Determinante: {np.linalg.det(A):.4f}')

x = np.linalg.solve(A, b)
I0, I1, I2, I3, I4, I = x

print(f'Gesamtstrom  I0 = {I0*1000:.2f} mA')
print(f'Querstrom    I  = {I*1000:.4f} mA')
print(f'Probe bestanden: {np.allclose(A @ x, b)}')
```

Der Querstrom $I$ ist bei $R_4 = 200\,\Omega$ ungleich null. Das
Vorzeichen sagt, in welche Richtung $I$ tatsächlich fließt: positiv
bedeutet Fluss in der definierten Zählpfeilrichtung, negativ entgegen.

Wie viele Gleichungen brauchen wir insgesamt? Für 6 Unbekannte genau
6 unabhängige Gleichungen. Drei Knoten und drei Maschen reichen genau
aus. Eine vierte Maschengleichung, etwa für die äußere Gesamtmasche,
wäre eine Linearkombination der drei obigen und würde den Rang der
Matrix nicht erhöhen.

```{admonition} Mini-Übung
:class: tip
Sehen Sie sich die Koeffizientenmatrix $\mathbf{A}$ an und beantworten
Sie die folgenden Fragen zunächst im Kopf, ohne Code auszuführen.

1. In Zeile 4 (M1) steht $A_{4,2} = R_1 = 100$. Aus welcher
   physikalischen Gleichung stammt dieser Eintrag? Was bedeutet er
   inhaltlich?
2. In Zeile 6 (M3) steht $A_{6,4} = -R_3 = -100$. Warum ist dieser
   Eintrag negativ?
3. Die Einträge $b[0] = b[1] = b[2] = 0$, aber $b[3] = b[4] = U_0$.
   Was bedeutet es, wenn ein Eintrag von $\vec{b}$ null ist?
4. Ändern Sie im Code oben $R_4 = 50\,\Omega$ und führen Sie die
   Zelle aus. Ändert sich das Vorzeichen von $I$? Was sagt das
   physikalisch aus?
```

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
**Zu Frage 1:** $A_{4,2} = R_1$ stammt aus Gleichung M1:
$R_1 \cdot I_1 + R_2 \cdot I_2 = U_0$. Der Eintrag ist der Koeffizient
von $I_1$ (zweite Unbekannte, Index 1) in der Maschengleichung der linken
Masche. Er beschreibt den Spannungsabfall, den der Strom $I_1$ am
Widerstand $R_1$ erzeugt.

**Zu Frage 2:** In Gleichung M3 wird die Quermasche im Uhrzeigersinn
durchlaufen. Der Widerstand $R_3$ wird dabei entgegen der definierten
Zählpfeilrichtung von $I_3$ durchquert; das Vorzeichen wechselt daher.

**Zu Frage 3:** $b_i = 0$ bedeutet, dass in dieser Gleichung keine
Spannungsquelle auftaucht. Die drei Knotengleichungen enthalten keine
externe Quelle. Auch die Quermasche (M3) enthält keine Quelle, daher
ist auch $b[5] = 0$.

**Zu Frage 4:**
```python
import numpy as np

R1 = 100.; R2 = 100.; R3 = 100.; R4 = 50.; RL = 10.; U0 = 10.

A = np.array([
    [+1., -1.,  0., -1.,   0.,   0.],
    [ 0., +1., -1.,  0.,   0.,  -1.],
    [ 0.,  0.,  0., +1.,  -1.,  +1.],
    [ 0.,  R1,  R2,  0.,   0.,   0.],
    [ 0.,  0.,  0.,  R3,   R4,   0.],
    [ 0.,  R1,  0., -R3,   0.,  -RL],
])
b = np.array([0., 0., 0., U0, U0, 0.])
x = np.linalg.solve(A, b)
print(f'I = {x[5]*1000:.4f} mA')
```

Bei $R_4 = 50\,\Omega$ ändert sich das Vorzeichen von $I$ gegenüber
$R_4 = 200\,\Omega$. Das ist physikalisch sinnvoll: der Querstrom
kehrt beim Übergang durch den Abgleichwert $R_4^* = 100\,\Omega$
seine Richtung um. Oberhalb des Abgleichwerts fließt er in die eine,
unterhalb in die andere Richtung.
````

+++

## Als Funktion implementieren

Das Muster aus der Zelle oben verallgemeinern wir zu einer Funktion.
$R_4$ wird als Parameter übergeben; alles andere bleibt gleich:

```{code-cell} python
def solve_bridge(R4):
    """Berechnet den Querstrom I durch den Brückenwiderstand R_L.

    Parameter
    ----------
    R4 : float
        Variabler Messwiderstand in Ohm

    Rückgabe
    --------
    I : float
        Querstrom durch R_L in Ampere, oder None bei singulärer Matrix
    """
    # Koeffizientenmatrix: gleiche Struktur wie oben, R4 als Parameter
    A = np.array([
        [+1., -1.,  0., -1.,   0.,   0.],   # K1: I0 - I1 - I3 = 0
        [ 0., +1., -1.,  0.,   0.,  -1.],   # K2: I1 - I2 - I  = 0
        [ 0.,  0.,  0., +1.,  -1.,  +1.],   # K3: I3 - I4 + I  = 0
        [ 0.,  R1,  R2,  0.,   0.,   0.],   # M1: R1*I1 + R2*I2 = U0
        [ 0.,  0.,  0.,  R3,   R4,   0.],   # M2: R3*I3 + R4*I4 = U0
        [ 0.,  R1,  0., -R3,   0.,  -RL],   # M3: R1*I1 - R3*I3 - RL*I = 0
    ])
    b = np.array([0., 0., 0., U0, U0, 0.])

    if np.isclose(np.linalg.det(A), 0.):
        print(f'Warnung: singuläre Matrix bei R4 = {R4} Ohm')
        return None

    # Index 5 im Lösungsvektor ist der Querstrom I
    return np.linalg.solve(A, b)[5]


# Test für R4 = 200 Ohm
R4_test = 200.
I_test  = solve_bridge(R4_test)

print(f'R4 = {R4_test:.0f} Ohm')
print(f'Querstrom        I   = {I_test*1000:.4f} mA')
print(f'Verlustleistung  P_L = {RL * I_test**2 * 1000:.4f} mW')
```

````{admonition} Mini-Übung
:class: tip
Überprüfen Sie `solve_bridge` für den Sonderfall $R_4 = 100\,\Omega$
(alle vier Brückenwiderstände gleich groß).

1. Was erwarten Sie physikalisch für den Querstrom $I$? Überlegen Sie
   zunächst im Kopf: Was passiert mit den Spannungen in der linken und
   rechten Hälfte der Brücke, wenn $R_1 = R_2 = R_3 = R_4$?
2. Rufen Sie `solve_bridge(100.)` auf. Stimmt das Ergebnis mit Ihrer
   Erwartung überein?
3. Erweitern Sie die Funktion oder schreiben Sie eine neue Zelle, die
   den gesamten Lösungsvektor $\vec{x}$ berechnet. Wie groß ist der
   Gesamtstrom $I_0$ bei $R_4 = 100\,\Omega$? Überprüfen Sie das
   Ergebnis mit einer Probe.
````

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# Frage 1: Erwartung
# Bei R1=R2=R3=R4=100 Ohm teilen beide Hälfte U0 symmetrisch auf.
# Spannung in der Mitte links:  U0 * R2/(R1+R2) = 5 V
# Spannung in der Mitte rechts: U0 * R4/(R3+R4) = 5 V
# Keine Spannungsdifferenz über R_L -> kein Querstrom: I = 0

# Frage 2
I_abgeglichen = solve_bridge(100.)
print(f'I bei R4 = 100 Ohm: {I_abgeglichen:.2e} A   (numerisch nahezu 0)')

# Frage 3: gesamter Lösungsvektor
R4 = 100.
A = np.array([
    [+1., -1.,  0., -1.,   0.,   0.],
    [ 0., +1., -1.,  0.,   0.,  -1.],
    [ 0.,  0.,  0., +1.,  -1.,  +1.],
    [ 0.,  R1,  R2,  0.,   0.,   0.],
    [ 0.,  0.,  0.,  R3,   R4,   0.],
    [ 0.,  R1,  0., -R3,   0.,  -RL],
])
b = np.array([0., 0., 0., U0, U0, 0.])
x = np.linalg.solve(A, b)

print(f'I0 = {x[0]*1000:.2f} mA   (Gesamtstrom)')
print(f'I  = {x[5]:.2e} A    (Querstrom, nahezu 0)')
print(f'Probe bestanden: {np.allclose(A @ x, b)}')
```

Bei $R_1 = R_2 = R_3 = R_4$ ist die Brücke **abgeglichen**: der
Querstrom ist numerisch nahezu null. Genau das ist das Messprinzip
der Wheatstone-Brücke: Jede Abweichung von $R_4 = 100\,\Omega$
erzeugt einen messbaren Querstrom. Im nächsten Kapitel variieren
wir $R_4$ systematisch und stellen $I(R_4)$ grafisch dar.
````

+++

## Zusammenfassung und Ausblick

Das Vorgehen ist dasselbe wie bei der Wärmeübertragung in Kapitel 3.3:
physikalische Gleichungen aufstellen, alle Unbekannten auf die linke
Seite bringen, $\mathbf{A}$ und $\vec{b}$ ablesen. Neu ist nur, dass
die Gleichungen jetzt aus der Knotenregel und der Maschenregel stammen.

Als Funktion `solve_bridge(R4)` verpackt, lässt sich das System für
beliebige $R_4$-Werte auswerten. Im nächsten Kapitel nutzen wir diese
Funktion für eine vollständige Parameterstudie: Wir berechnen $I(R_4)$
und die Verlustleistung $P_L(R_4)$ für viele Werte und stellen die
Ergebnisse in einem Diagramm dar. Dort sehen wir auch die Nullstelle
bei $R_4^* = 100\,\Omega$ und können sie mit der analytischen
Abgleichbedingung vergleichen.
