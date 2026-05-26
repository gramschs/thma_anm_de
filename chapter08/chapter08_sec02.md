---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 8.2 Übungen zu Gradient Descent in 1D

````{admonition} Übung 8.1 (✩)
:class: tip
Gegeben ist der folgende Code, der Gradient Descent auf eine einfache
Kostenfunktion anwendet.

```python
import numpy as np

def kosten(x):
    return (x - 3)**2 + 1

def ableitung_kosten(x, dx=1e-5):
    return (kosten(x + dx) - kosten(x - dx)) / (2 * dx)

x_aktuell = 0.0
alpha = 0.1

for i in range(5):
    grad = ableitung_kosten(x_aktuell)
    x_aktuell = x_aktuell - alpha * grad
    print(f"Iteration {i+1}: x = {x_aktuell:.4f},  K = {kosten(x_aktuell):.4f}")
```

1. Wo liegt das Minimum von $K(x)$? Begründen Sie ohne Code.
2. Berechnen Sie von Hand, was die erste Ausgabezeile (Iteration 1) zeigt.
   Geben Sie `x_aktuell` und `K(x_aktuell)` nach dem ersten Schritt an.
3. Bewegt sich `x_aktuell` in jedem Schritt nach links oder nach rechts?
   Begründen Sie anhand der Update-Formel ohne Code.
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen aus den
   Fragen 1 bis 3.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

def kosten(x):
    return (x - 3)**2 + 1

def ableitung_kosten(x, dx=1e-5):
    return (kosten(x + dx) - kosten(x - dx)) / (2 * dx)

x_aktuell = 0.0
alpha = 0.1

for i in range(5):
    grad = ableitung_kosten(x_aktuell)
    x_aktuell = x_aktuell - alpha * grad
    print(f"Iteration {i+1}: x = {x_aktuell:.4f},  K = {kosten(x_aktuell):.4f}")
```

Zu Frage 1: $K(x) = (x - 3)^2 + 1$ ist ein nach oben geöffnetes Parabel mit
Scheitel bei $x = 3$. Das Minimum liegt bei $x = 3$, $K(3) = 1$.

Zu Frage 2: Bei $x = 0$ liefert die zentrale Differenz
$K'(0) \approx 2 \cdot (0 - 3) = -6$. Der Update-Schritt lautet
$x_\text{neu} = 0 - 0.1 \cdot (-6) = 0.6$. Der Kostenwert danach:
$K(0.6) = (0.6 - 3)^2 + 1 = 5.76 + 1 = 6.76$.
Die erste Ausgabezeile lautet also:
`Iteration 1: x = 0.6000,  K = 6.7600`

Zu Frage 3: Für $x < 3$ gilt $K'(x) < 0$. Der Schritt
$-\alpha \cdot K'(x)$ ist dann positiv, `x_aktuell` wächst. Der Algorithmus
bewegt sich nach rechts, in Richtung Minimum bei $x = 3$.
````

````{admonition} Übung 8.2 (✩✩)
:class: tip
Die Funktion $f(x) = x^4 - 4x^2$ hat mehr als ein lokales Minimum.

1. Plotten Sie $f(x)$ im Bereich $x \in [-2.5,\; 2.5]$. Wo liegen die
   lokalen Minima ungefähr? Bestimmen Sie die genauen Positionen analytisch
   mit $f'(x) = 0$.
2. Implementieren Sie Gradient Descent mit der numerischen Ableitung
   (zentrale Differenz). Verwenden Sie `alpha = 0.02` und
   `n_iterationen = 500`. Starten Sie von `x_start = 2.0` und geben Sie
   das gefundene Minimum aus.
3. Wiederholen Sie den Lauf mit `x_start = -2.0`. Was beobachten Sie?
4. Stellen Sie die Konvergenzkurven beider Läufe in einem gemeinsamen Plot
   dar. Beschriften Sie Achsen und Kurven.
5. Starten Sie einen dritten Lauf von `x_start = 0.01`. Zu welchem Minimum
   konvergiert der Algorithmus? Begründen Sie, warum er nicht bei $x = 0$
   stehen bleibt, obwohl $f'(0) = 0$ gilt.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

# --- Frage 1: Funktion plotten und Minima bestimmen ---

def f(x):
    """f(x) = x^4 - 4x^2"""
    return x**4 - 4 * x**2

# Analytische Ableitung: f'(x) = 4x^3 - 8x = 4x(x^2 - 2)
# f'(x) = 0  =>  x = 0  oder  x = +-sqrt(2)
# f''(x) = 12x^2 - 8: f''(0) = -8 < 0 (lokales Maximum)
#                      f''(+-sqrt(2)) = 16 > 0 (lokale Minima)
x_min_analytisch = np.sqrt(2)
print(f"Lokale Minima bei x = +-{x_min_analytisch:.4f},  f = {f(x_min_analytisch):.4f}")

x_plot = np.linspace(-2.5, 2.5, 300)
fig, ax = plt.subplots()
ax.plot(x_plot, f(x_plot), label='f(x) = x⁴ − 4x²')
ax.axhline(0, color='gray', linewidth=0.8)
ax.scatter([-x_min_analytisch, x_min_analytisch],
           [f(-x_min_analytisch), f(x_min_analytisch)],
           color='red', zorder=5, label=f'Minima bei x = ±√2 ≈ ±{x_min_analytisch:.2f}')
ax.set_xlabel('x')
ax.set_ylabel('f(x)')
ax.set_title('f(x) = x⁴ − 4x² mit zwei lokalen Minima')
ax.legend()
ax.grid(True)
plt.show()

# --- Gradient-Descent-Implementierung ---

def ableitung_f(x, dx=1e-6):
    """Numerische Ableitung von f mit der zentralen Differenz."""
    return (f(x + dx) - f(x - dx)) / (2 * dx)

def gradient_descent(x_start, alpha=0.02, n_iterationen=500):
    """GD-Loop: gibt den Endwert und den Kostenverlauf zurück."""
    x_aktuell     = x_start
    kosten_verlauf = []
    for i in range(n_iterationen):
        kosten_verlauf.append(f(x_aktuell))
        grad      = ableitung_f(x_aktuell)
        x_aktuell = x_aktuell - alpha * grad
    return x_aktuell, kosten_verlauf

# --- Fragen 2 und 3: Läufe von x = 2 und x = -2 ---
x_ende_pos, verlauf_pos = gradient_descent(x_start= 2.0)
x_ende_neg, verlauf_neg = gradient_descent(x_start=-2.0)

print(f"Start  2.0: Minimum bei x = {x_ende_pos:.5f},  f = {f(x_ende_pos):.5f}")
print(f"Start -2.0: Minimum bei x = {x_ende_neg:.5f},  f = {f(x_ende_neg):.5f}")

# --- Frage 4: Konvergenzkurven ---
fig, ax = plt.subplots()
ax.plot(verlauf_pos, label='Start x = 2.0')
ax.plot(verlauf_neg, label='Start x = −2.0')
ax.set_xlabel('Iteration')
ax.set_ylabel('f(x)')
ax.set_title('Konvergenz von zwei Startwerten')
ax.legend()
ax.grid(True)
plt.show()

# --- Frage 5: Start nahe bei x = 0 ---
x_ende_null, _ = gradient_descent(x_start=0.01)
print(f"Start  0.01: Minimum bei x = {x_ende_null:.5f},  f = {f(x_ende_null):.5f}")
```

Zu Frage 2 und 3: Von $x = 2$ findet GD das rechte Minimum bei
$x \approx +\sqrt{2} \approx 1.4142$, von $x = -2$ das linke bei
$x \approx -\sqrt{2}$. Beide Minima haben denselben Funktionswert $f = -4$.
Welches Minimum gefunden wird, hängt allein vom Startwert ab.

Zu Frage 5: Bei $x = 0.01$ ist die Ableitung $f'(0.01) \approx -0.08$
(negativ), der Schritt also positiv. Der Algorithmus bewegt sich nach rechts
und konvergiert zum Minimum bei $+\sqrt{2}$. Bei genau $x = 0$ wäre
$f'(0) = 0$, der Algorithmus würde sich nicht bewegen und am lokalen
Maximum stehen bleiben. Das ist jedoch ein instabiler Gleichgewichtspunkt:
jede kleine Abweichung führt den Algorithmus in eine der beiden Richtungen.

Gradient Descent garantiert nur die Konvergenz zu einem lokalen Minimum,
nicht zum globalen. Die Kostenfunktion der Abkühlkurve aus Abschnitt 8.1
hat zum Glück nur ein einziges Minimum, sodass dieses Problem dort nicht
auftritt.
````
