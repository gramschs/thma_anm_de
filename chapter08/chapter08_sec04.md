---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 8.4 Übungen zu Gradient Descent in 2D

```{admonition} Warnung
:class: warning
Dieses Kapitel befindet sich derzeit im Umbau und wird rechtzeitig vor der Vorlesung im WiSe 2026/27 zur Verfügung stehen.
```

````{admonition} Übung 8.3 (✩)
:class: tip
Gegeben ist der folgende Code, der `scipy.optimize.minimize` auf eine
zweidimensionale Kostenfunktion anwendet.

```python
from scipy.optimize import minimize

def f(params):
    a, b = params
    return (a - 5)**2 + 2 * (b + 3)**2

ergebnis = minimize(f, x0=[0.0, 0.0])

print(f"a          = {ergebnis.x[0]:.4f}")
print(f"b          = {ergebnis.x[1]:.4f}")
print(f"f(a, b)    = {ergebnis.fun:.6f}")
print(f"Konvergiert: {ergebnis.success}")
print(f"Iterationen: {ergebnis.nit}")
```

1. Wo liegt das Minimum von $f(a, b)$? Berechnen Sie es ohne Code und
   geben Sie $a^*$ und $b^*$ an.
2. Was erwarten Sie für die Ausgabe von `ergebnis.x` und `ergebnis.fun`?
   Schreiben Sie die erwarteten Werte auf, bevor Sie den Code ausführen.
3. Der erste Term $(a - 5)^2$ hat den Koeffizienten 1, der zweite
   $2 \cdot (b + 3)^2$ den Koeffizienten 2. In welcher Richtung ist
   $f$ steiler, in $a$- oder in $b$-Richtung? Was würde das für die
   Wahl von Lernraten bei einem eigenen GD-Algorithmus bedeuten?
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen aus
   den Fragen 1 und 2.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
from scipy.optimize import minimize

def f(params):
    a, b = params
    return (a - 5)**2 + 2 * (b + 3)**2

ergebnis = minimize(f, x0=[0.0, 0.0])

print(f"a          = {ergebnis.x[0]:.4f}")
print(f"b          = {ergebnis.x[1]:.4f}")
print(f"f(a, b)    = {ergebnis.fun:.6f}")
print(f"Konvergiert: {ergebnis.success}")
print(f"Iterationen: {ergebnis.nit}")
```

Zu Frage 1: Jedes Quadrat ist mindestens null. Das Minimum liegt vor, wenn
beide Terme gleichzeitig null sind, also bei $a^* = 5$ und $b^* = -3$.
Dort gilt $f(5, -3) = 0$.

Zu Frage 2: `ergebnis.x` enthält `[5.0, -3.0]` (bis auf numerische
Ungenauigkeiten), `ergebnis.fun` ist nahezu null.

Zu Frage 3: In $b$-Richtung ist $f$ doppelt so steil wie in $a$-Richtung,
weil der Koeffizient 2 vor dem zweiten Term steht. Der Gradient in
$b$-Richtung ist für denselben Abstand vom Minimum doppelt so groß. Bei
einem eigenen GD müsste $\alpha_b$ halb so groß wie $\alpha_a$ gewählt
werden, damit die Schritte in beiden Richtungen gleich groß ausfallen.
````

````{admonition} Übung 8.4 (✩✩)
:class: tip
Die **Wöhlerkurve** beschreibt, wie viele Lastspiele $N$ ein Bauteil bei
einer Spannungsamplitude $S$ aushält. Das Modell lautet

$$S = a \cdot N^{-b},$$

wobei $a$ und $b$ materialabhängige Parameter sind. Maschinenbauteile aus
Stahl haben typischerweise $a \approx 1500\,\text{MPa}$ und
$b \approx 0.12$.

Im folgenden Datensatz stehen simulierte Prüfergebnisse zur Verfügung:

```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')
from scipy.optimize import minimize

np.random.seed(7)

# --- Wöhlerversuch: Spannungsamplitude vs. Lastspielzahl ---
a_wahr = 1500.0  # MPa
b_wahr = 0.12
N_mess = np.array([1e4, 3e4, 1e5, 5e5, 2e6, 1e7])  # Lastspielzahl
S_wahr = a_wahr * N_mess**(-b_wahr)                  # Spannungsamplitude in MPa
# Multiplikatives Rauschen: ±5 % Streuung
S_mess = S_wahr * (1 + np.random.normal(0, 0.05, len(N_mess)))
```

1. Plotten Sie `N_mess` gegen `S_mess` in einem Log-Log-Diagramm
   (`ax.loglog`). Beschriften Sie die Achsen mit Einheiten. Warum
   erscheint die Wöhlerkurve im Log-Log-Diagramm als Gerade?
2. Implementieren Sie Kostenfunktion und numerische Gradienten für die
   Parameter $a$ und $b$. Wählen Sie `alpha_a = 0.5` und
   `alpha_b = 1e-8` als Lernraten sowie `[500.0, 0.05]` als Startwerte.
   Führen Sie 10000 Iterationen durch und geben Sie die gefundenen
   Parameter aus.
3. Plotten Sie die angepasste Kurve über die Messdaten (Log-Log).
4. Wiederholen Sie die Anpassung mit `scipy.optimize.minimize` und
   vergleichen Sie die Ergebnisse.
5. Warum ist $\alpha_b$ so viel kleiner als $\alpha_a$? Berechnen Sie
   die numerischen Gradienten am Startpunkt und begründen Sie das
   Verhältnis der Lernraten damit.
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
from scipy.optimize import minimize

np.random.seed(7)

# --- Datenerzeugung ---
a_wahr = 1500.0
b_wahr = 0.12
N_mess = np.array([1e4, 3e4, 1e5, 5e5, 2e6, 1e7])
S_wahr = a_wahr * N_mess**(-b_wahr)
S_mess = S_wahr * (1 + np.random.normal(0, 0.05, len(N_mess)))

# --- Frage 1: Log-Log-Plot der Rohdaten ---
fig, ax = plt.subplots()
ax.loglog(N_mess, S_mess, 'o', label='Prüfdaten')
ax.set_xlabel('Lastspielzahl N')
ax.set_ylabel('Spannungsamplitude S in MPa')
ax.set_title('Wöhlerkurve (Rohdaten)')
ax.grid(True, which='both')
ax.legend()
plt.show()

# --- Modell- und Kostenfunktion ---

def modell_woehler(N, a, b):
    """Wöhlermodell: S = a * N^(-b)."""
    return a * N**(-b)

def kosten(a, b):
    """MSE zwischen Modell und Messdaten."""
    S_vorhergesagt = modell_woehler(N_mess, a, b)
    return np.mean((S_mess - S_vorhergesagt)**2)

# --- Fragen 2 und 3: Gradient Descent ---
dx = 1e-6

def dK_da(a, b):
    return (kosten(a + dx, b) - kosten(a - dx, b)) / (2 * dx)

def dK_db(a, b):
    return (kosten(a, b + dx) - kosten(a, b - dx)) / (2 * dx)

a_aktuell   = 500.0
b_aktuell   = 0.05
alpha_a     = 0.5
alpha_b     = 1e-8
n_iterationen = 10000

for i in range(n_iterationen):
    grad_a    = dK_da(a_aktuell, b_aktuell)
    grad_b    = dK_db(a_aktuell, b_aktuell)
    a_aktuell = a_aktuell - alpha_a * grad_a
    b_aktuell = b_aktuell - alpha_b * grad_b

print("--- Gradient Descent ---")
print(f"a = {a_aktuell:.2f} MPa  (wahr: {a_wahr:.2f} MPa)")
print(f"b = {b_aktuell:.4f}      (wahr: {b_wahr:.4f})")
print(f"MSE = {kosten(a_aktuell, b_aktuell):.4f} MPa²")
print()

# Angepasste Kurve über Messdaten
N_fein = np.logspace(4, 7, 300)
fig, ax = plt.subplots()
ax.loglog(N_mess, S_mess, 'o', label='Prüfdaten')
ax.loglog(N_fein, modell_woehler(N_fein, a_aktuell, b_aktuell),
          label='Gradient Descent')
ax.set_xlabel('Lastspielzahl N')
ax.set_ylabel('Spannungsamplitude S in MPa')
ax.set_title('Wöhlerkurve: Kurvenanpassung')
ax.grid(True, which='both')
ax.legend()
plt.show()

# --- Frage 4: scipy ---
def kosten_vektor(params):
    return kosten(params[0], params[1])

ergebnis = minimize(kosten_vektor, x0=[500.0, 0.05])
a_scipy, b_scipy = ergebnis.x

print("--- scipy.optimize.minimize ---")
print(f"a = {a_scipy:.2f} MPa  (wahr: {a_wahr:.2f} MPa)")
print(f"b = {b_scipy:.4f}      (wahr: {b_wahr:.4f})")
print(f"MSE = {kosten(a_scipy, b_scipy):.4f} MPa²")
print(f"Iterationen: {ergebnis.nit}")
print()

# --- Frage 5: Gradienten am Startpunkt ---
grad_a_start = dK_da(500.0, 0.05)
grad_b_start = dK_db(500.0, 0.05)
print("--- Gradienten am Startpunkt (a=500, b=0.05) ---")
print(f"dK/da = {grad_a_start:.2f}")
print(f"dK/db = {grad_b_start:.2f}")
print(f"Verhältnis |dK/db| / |dK/da| = {abs(grad_b_start)/abs(grad_a_start):.1f}")
```

Zu Frage 1: Im Log-Log-Diagramm gilt
$\log S = \log a - b \cdot \log N$, also eine Gerade mit Steigung $-b$.
Jede Potenzfunktion $S = a \cdot N^{-b}$ wird im Log-Log-Diagramm zu einer
Geraden.

Zu Frage 5: Der Gradient in $b$-Richtung ist um mehrere Größenordnungen
größer als in $a$-Richtung, weil $\partial S / \partial b = -a \cdot N^{-b}
\cdot \ln N$ den Faktor $\ln N$ enthält, der für $N = 10^4$ bis $10^7$
zwischen 9 und 16 liegt, und weil $a \approx 1500$ die Empfindlichkeit
weiter verstärkt. Das Verhältnis der Lernraten $\alpha_a / \alpha_b =
0.5 / 10^{-8} = 5 \cdot 10^7$ gleicht diesen Unterschied der
Gradientenbeträge aus.
````
