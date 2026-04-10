---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 3.4 Anwendung: Modellierung Messbrücke

In der Messtechnik wird die **Wheatstone-Brückenschaltung** eingesetzt, um
kleine Widerstandsänderungen präzise zu messen — zum Beispiel bei
Dehnungsmessstreifen, Temperatursensoren oder Kraftsensoren. Das Prinzip:
Wenn alle vier Brückenwiderstände in einem bestimmten Verhältnis stehen,
ist der Strom durch den Brückenwiderstand $R_L$ exakt null. Jede Abweichung
erzeugt einen messbaren Querstrom.

In diesem Notebook leiten wir das Gleichungssystem für die Brückenschaltung
vollständig aus den Kirchhoffschen Regeln her und lösen es für einen
bestimmten Wert von $R_4$.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können die Kirchhoffsche **Knotenregel** und **Maschenregel**
  auf eine Schaltung anwenden.
* [ ] Sie können aus 6 Kirchhoff-Gleichungen das LGS
  $\mathbf{A} \cdot \vec{x} = \vec{b}$ aufstellen.
* [ ] Sie können das System als Python-Funktion implementieren, die $I$
  für ein gegebenes $R_4$ zurückgibt.
* [ ] Sie können das Ergebnis für einen Einzelwert testen und
  physikalisch einordnen.
```

+++

## Die Schaltung

Die Brückenschaltung hat folgende Topologie:

```code
            I_0
     +------+------+
     |      |      |
    [R1]   [R3]    |
  I_1|    I_3|     |
     +--[RL]--+    |
  I_2|   I   I_4|  |
    [R2]   [R4]    |
     |      |      |
     +------+------+
    U0
```

Genauer: Die Spannungsquelle $U_0$ liegt zwischen dem unteren und oberen
Knotenpunkt. Von oben fließt $I_0$; dieser teilt sich auf:

- Linker Zweig: $I_1$ durch $R_1$, dann $I_2$ durch $R_2$
- Rechter Zweig: $I_3$ durch $R_3$, dann $I_4$ durch $R_4$
- Querzweig: $I$ durch $R_L$ (Brückenwiderstand)

**Daten:**
$R_1 = R_2 = R_3 = 100\,\Omega$,\quad $R_L = 10\,\Omega$,\quad $U_0 = 10\,\text{V}$

$R_4$ ist variabel — das ist der Messwiderstand.

+++

## Schritt 1: Stromrichtungen festlegen

Bevor wir die Kirchhoff-Gleichungen aufschreiben, legen wir die
**Zählpfeilrichtungen** für alle Ströme fest. Das ist eine willkürliche
Wahl — wenn ein Strom negativ herauskommt, fließt er in Wirklichkeit
entgegen unserer angenommenen Richtung.

Wir wählen:

- $I_0$: von der Quelle in den oberen Knoten (von links nach rechts)
- $I_1$: durch $R_1$ nach unten (linker Zweig, oberer Teil)
- $I_2$: durch $R_2$ nach unten (linker Zweig, unterer Teil)
- $I_3$: durch $R_3$ nach unten (rechter Zweig, oberer Teil)
- $I_4$: durch $R_4$ nach unten (rechter Zweig, unterer Teil)
- $I$:   durch $R_L$ von links nach rechts (Brücke)

Der Lösungsvektor ist also:
$$\vec{x} = (I_0,\, I_1,\, I_2,\, I_3,\, I_4,\, I)^T$$

+++

## Schritt 2: Knotenregel anwenden

Die **Knotenregel** (1. Kirchhoffsches Gesetz) sagt: An jedem Knoten ist
die Summe der zufließenden Ströme gleich der Summe der abfließenden Ströme.

Wir identifizieren die relevanten Knoten:

**Knoten A** (oben links, zwischen $R_1$ und $R_3$, wo $I_0$ ankommt):

$$I_0 = I_1 + I_3 \quad\Rightarrow\quad I_0 - I_1 - I_3 = 0 \tag{K1}$$

**Knoten B** (Mitte links, zwischen $R_1$, $R_L$ und $R_2$):

$$I_1 = I + I_2 \quad\Rightarrow\quad I_1 - I_2 - I = 0 \tag{K2}$$

**Knoten C** (Mitte rechts, zwischen $R_3$, $R_L$ und $R_4$):

$$I_3 + I = I_4 \quad\Rightarrow\quad I_3 - I_4 + I = 0 \tag{K3}$$

```{admonition} Knotenregel kompakt
:class: note
Wir schreiben die Knotenregel immer so um, dass alle Terme auf der linken
Seite stehen und rechts eine Null steht. Zufließende Ströme erhalten
ein positives Vorzeichen, abfließende ein negatives — oder umgekehrt,
Hauptsache konsistent.
```

+++

## Schritt 3: Maschenregel anwenden

Die **Maschenregel** (2. Kirchhoffsches Gesetz) sagt: Die Summe aller
Spannungsabfälle in einer geschlossenen Masche ist gleich der Summe
der Quellenspannungen.

Der Spannungsabfall über einem Widerstand berechnet sich nach dem Ohmschen
Gesetz: $U_R = R \cdot I$. Wir durchlaufen jede Masche im Uhrzeigersinn.
Ein Widerstand, den wir in Richtung des definierten Stroms durchlaufen,
liefert einen positiven Spannungsabfall.

**Masche 1** (linke Masche: Quelle - $R_1$ - $R_2$):

$$U_0 = R_1 \cdot I_1 + R_2 \cdot I_2
\quad\Rightarrow\quad R_1 I_1 + R_2 I_2 = U_0 \tag{M1}$$

**Masche 2** (rechte Masche: $R_3$ - $R_4$ - Quelle):

$$U_0 = R_3 \cdot I_3 + R_4 \cdot I_4
\quad\Rightarrow\quad R_3 I_3 + R_4 I_4 = U_0 \tag{M2}$$

**Masche 3** (Quermasche: $R_1$ - $R_L$ - $R_3$):

$$R_1 \cdot I_1 = R_L \cdot I + R_3 \cdot I_3
\quad\Rightarrow\quad R_1 I_1 - R_3 I_3 - R_L I = 0 \tag{M3}$$

```{admonition} Drei Maschen reichen
:class: note
Wir haben 6 Unbekannte und brauchen daher 6 unabhängige Gleichungen.
3 Knotengleichungen + 3 Maschengleichungen = 6. Eine vierte Masche
(z. B. die äußere Gesamtmasche) wäre linear abhängig von den drei
obigen und würde keine neue Information liefern.
```

+++

## Schritt 4: Matrixform aufstellen

Wir sammeln alle sechs Gleichungen und sortieren sie nach den Unbekannten
$I_0, I_1, I_2, I_3, I_4, I$:

| Gleichung | $I_0$ | $I_1$ | $I_2$ | $I_3$ | $I_4$ | $I$ | $= b$ |
|-----------|-------|-------|-------|-------|-------|-----|-------|
| K1        | $+1$  | $-1$  | $0$   | $-1$  | $0$   | $0$ | $0$   |
| K2        | $0$   | $+1$  | $-1$  | $0$   | $0$   | $-1$| $0$   |
| K3        | $0$   | $0$   | $0$   | $+1$  | $-1$  | $+1$| $0$   |
| M1        | $0$   | $R_1$ | $R_2$ | $0$   | $0$   | $0$ | $U_0$ |
| M2        | $0$   | $0$   | $0$   | $R_3$ | $R_4$ | $0$ | $U_0$ |
| M3        | $0$   | $R_1$ | $0$   | $-R_3$| $0$   | $-R_L$| $0$ |

Diese Tabelle ist direkt die Koeffizientenmatrix $\mathbf{A}$ und der
Vektor $\vec{b}$.

+++

## Schritt 5: Implementierung in Python

Wir implementieren das System als Funktion, die $R_4$ als Parameter
entgegennimmt und den Querstrom $I$ zurückgibt:

```{code-cell} python
import numpy as np

# Feste Parameter
R1 = 100.   # Ohm
R2 = 100.   # Ohm
R3 = 100.   # Ohm
RL = 10.    # Ohm (Brückenwiderstand)
U0 = 10.    # V

def solve_bridge(R4):
    """Berechnet den Querstrom I durch den Brückenwiderstand R_L.

    R4: variabler Widerstand in Ohm
    Rückgabe: Querstrom I in Ampere
    """
    # Koeffizientenmatrix: Zeilen = Gleichungen, Spalten = [I0,I1,I2,I3,I4,I]
    A = np.array([
        # K1: I0 - I1 - I3 = 0
        [+1., -1.,  0., -1.,   0.,   0.],
        # K2: I1 - I2 - I = 0
        [ 0., +1., -1.,  0.,   0.,  -1.],
        # K3: I3 - I4 + I = 0
        [ 0.,  0.,  0., +1.,  -1.,  +1.],
        # M1: R1*I1 + R2*I2 = U0
        [ 0.,  R1,  R2,  0.,   0.,   0.],
        # M2: R3*I3 + R4*I4 = U0
        [ 0.,  0.,  0.,  R3,   R4,   0.],
        # M3: R1*I1 - R3*I3 - RL*I = 0
        [ 0.,  R1,  0., -R3,   0.,  -RL],
    ])

    # Rechte Seite
    b = np.array([0., 0., 0., U0, U0, 0.])

    # Lösbarkeitstest
    if np.isclose(np.linalg.det(A), 0.):
        print(f'Warnung: singuläre Matrix bei R4 = {R4} Ohm')
        return None

    # Lösung
    x = np.linalg.solve(A, b)

    # x = [I0, I1, I2, I3, I4, I]
    I = x[5]   # Querstrom durch R_L
    return I
```

## Schritt 6: Test für einen Einzelwert

Wir testen die Funktion für $R_4 = 200\,\Omega$:

```{code-cell} python
R4_test = 200.   # Ohm
I_test  = solve_bridge(R4_test)

print(f'R4 = {R4_test:.0f} Ohm')
print(f'Querstrom I = {I_test*1000:.4f} mA')
print(f'Verlustleistung P_L = {RL * I_test**2 * 1000:.4f} mW')
```

````{admonition} Mini-Übung
:class: tip
Überprüfen Sie die Funktion `solve_bridge` für $R_4 = 100\,\Omega$ (alle
vier Brückenwiderstände gleich groß).

1. Was erwarten Sie physikalisch für den Querstrom $I$ in diesem Fall?
   Überlegen Sie, bevor Sie den Code ausführen.
2. Rufen Sie `solve_bridge(100.)` auf. Stimmt das Ergebnis mit Ihrer
   Erwartung überein?
3. Berechnen Sie auch $I_0$ (den Gesamtstrom) für $R_4 = 100\,\Omega$.
   Tipp: Erweitern Sie `solve_bridge` so, dass sie den gesamten
   Lösungsvektor $\vec{x}$ zurückgibt, oder lösen Sie das System
   direkt in einer neuen Zelle.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# Test R4 = 100 Ohm (abgeglichene Brücke)
I_abgeglichen = solve_bridge(100.)
print(f'I bei R4 = 100 Ohm: {I_abgeglichen:.2e} A')
# Erwartung: I ≈ 0, da die Brücke abgeglichen ist

# Gesamtstrom I0 direkt berechnen
R4 = 100.
A = np.array([
    [+1., -1.,  0., -1.,   0.,   0.],
    [ 0., +1., -1.,  0.,   0.,  -1.],
    [ 0.,  0.,  0., +1.,  -1.,  +1.],
    [ 0., R1,  R2,  0.,   0.,   0.],
    [ 0.,  0.,  0., R3,   R4,   0.],
    [ 0., R1,  0., -R3,   0., -RL],
])
b = np.array([0., 0., 0., U0, U0, 0.])
x = np.linalg.solve(A, b)
print(f'I0 = {x[0]*1000:.2f} mA   (Gesamtstrom)')
print(f'I  = {x[5]:.2e} A   (Querstrom, ≈ 0)')
```

Bei $R_1 = R_2 = R_3 = R_4 = 100\,\Omega$ ist die Brücke abgeglichen: der
Querstrom $I$ ist exakt null (oder so nahe bei null, dass er numerisch
als null erscheint). Das ist das Messprinzip der Wheatstone-Brücke: Abweichung
von $R_4 = 100\,\Omega$ erzeugt einen messbaren Querstrom.
````

+++

## Zusammenfassung und Ausblick

Wir haben die Kirchhoffschen Regeln Schritt für Schritt auf die
Brückenschaltung angewendet und ein 6×6-LGS aufgestellt. Der Schlüssel
ist das systematische Vorgehen:

1. Stromrichtungen festlegen
2. Knotenregel an jedem relevanten Knoten anwenden
3. Maschenregel für genügend viele unabhängige Maschen anwenden
4. Matrixform $\mathbf{A} \cdot \vec{x} = \vec{b}$ durch Ablesen aufstellen
5. Als Python-Funktion implementieren und testen

Im nächsten Notebook nutzen wir `solve_bridge(R4)` in einer Schleife über
viele Werte von $R_4$ und erstellen ein vollständiges Diagramm von
$I(R_4)$ und $P_L(R_4)$ — inklusive der Nullstelle, die das
Wheatstonesche Abgleichprinzip beschreibt.
