---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 10.4 Übungen zum Runge-Kutta-Verfahren

```{admonition} Warnung
:class: warning
Dieses Kapitel befindet sich derzeit im Umbau und wird rechtzeitig vor der Vorlesung im WiSe 2026/27 zur Verfügung stehen.
```

````{admonition} Übung 10.3 (✩)
:class: tip
Gegeben ist der folgende Code, der vier Experimente mit `solve_ivp`
durchführt. Die DGL beschreibt dasselbe Abkühlungsgesetz wie in
Abschnitt 10.1, aber mit anderen Parametern.

```python
from scipy.integrate import solve_ivp
import numpy as np

T0    = 60.0   # Anfangstemperatur in °C
T_inf = 25.0   # Umgebungstemperatur in °C
k     = 0.2    # Abkühlkonstante in 1/min

def f_kuehl(t, y):
    return [-k * (y[0] - T_inf)]

t_punkte = np.array([0.0, 5.0, 10.0, 15.0])

# Experiment A
sol_a = solve_ivp(f_kuehl, (0, 15), [T0], t_eval=t_punkte)
print(f"A: sol_a.y.shape  = {sol_a.y.shape}")
print(f"A: sol_a.y        = {np.round(sol_a.y, 4)}")
print(f"A: sol_a.y[0, 1]  = {sol_a.y[0, 1]:.4f} °C")

# Experiment B
sol_b = solve_ivp(f_kuehl, (0, 15), [T0])
print(f"B: sol_b.y.shape  = {sol_b.y.shape}")

# Experiment C
sol_c = solve_ivp(f_kuehl, (0, 15), [T0], t_eval=t_punkte)
print(f"C: sol_c.y[0, -1]  = {sol_c.y[0, -1]:.4f} °C")
print(f"C: sol_c.y[-1, -1] = {sol_c.y[-1, -1]:.4f} °C")
print(f"C: sol_c.y[:, -1]  = {sol_c.y[:, -1]}")

# Experiment D
def f_buggy(t, y):
    return [-k * (y[0] - T_inf), 0.0]   # Zeile D

try:
    sol_d = solve_ivp(f_buggy, (0, 15), [T0])
    print(f"D: sol_d.success = {sol_d.success}")
except Exception as e:
    print(f"D: {type(e).__name__}: {e}")
```

1. Experiment A: Welche Form hat `sol_a.y.shape`? Erklären Sie beide
   Dimensionen in einem Satz.
2. Experiment A: Berechnen Sie `sol_a.y[0, 1]` (Temperatur nach 5 min)
   analytisch mit $T(t) = T_\infty + (T_0 - T_\infty)\,e^{-kt}$ und
   vergleichen Sie mit dem Ausgabewert. Ist der Unterschied zu erwarten?
3. Experiment B hat kein `t_eval`. Wie viele Spalten hat `sol_b.y`
   ungefähr, und was bestimmt diese Zahl?
4. Experiment C: Sind `sol_c.y[0, -1]` und `sol_c.y[-1, -1]` für diese
   DGL identisch? Wäre das auch so, wenn die DGL drei Zustandsgrößen hätte?
5. Experiment D: Läuft der Code fehlerfrei durch? Erklären Sie, was
   Zeile D falsch macht, und nennen Sie die Korrektur in einem Satz.
   Führen Sie den Code anschließend aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
from scipy.integrate import solve_ivp
import numpy as np

T0 = 60.0; T_inf = 25.0; k = 0.2
t_punkte = np.array([0.0, 5.0, 10.0, 15.0])

def f_kuehl(t, y):
    return [-k * (y[0] - T_inf)]

sol_a = solve_ivp(f_kuehl, (0, 15), [T0], t_eval=t_punkte)
sol_b = solve_ivp(f_kuehl, (0, 15), [T0])
sol_c = solve_ivp(f_kuehl, (0, 15), [T0], t_eval=t_punkte)

print(f"A: sol_a.y.shape  = {sol_a.y.shape}")
print(f"A: sol_a.y        = {np.round(sol_a.y, 4)}")
print(f"A: sol_a.y[0, 1]  = {sol_a.y[0, 1]:.4f} °C")
print(f"B: sol_b.y.shape  = {sol_b.y.shape}")
print(f"C: sol_c.y[0, -1]  = {sol_c.y[0, -1]:.4f} °C")
print(f"C: sol_c.y[-1, -1] = {sol_c.y[-1, -1]:.4f} °C")
print(f"C: sol_c.y[:, -1]  = {sol_c.y[:, -1]}")

def f_buggy(t, y):
    return [-k * (y[0] - T_inf), 0.0]

try:
    sol_d = solve_ivp(f_buggy, (0, 15), [T0])
except Exception as e:
    print(f"D: {type(e).__name__}: {e}")
```

Erwartete Ausgabe:
```
A: sol_a.y.shape  = (1, 4)
A: sol_a.y        = [[60.     37.8643 29.7398 26.7473]]
A: sol_a.y[0, 1]  = 37.8643 °C
B: sol_b.y.shape  = (1, 6)
C: sol_c.y[0, -1]  = 26.7473 °C
C: sol_c.y[-1, -1] = 26.7473 °C
C: sol_c.y[:, -1]  = [26.74733677]
D: ValueError: could not broadcast input array from shape (2,) into shape (1,)
```

Zu Frage 1: `sol_a.y.shape = (1, 4)`. Die erste Dimension (1) ist die
Anzahl der Zustandsgrößen; die zweite (4) ist die Anzahl der
Ausgabezeitpunkte aus `t_eval`.

Zu Frage 2: Analytisch: $T(5) = 25 + 35 \cdot e^{-1} = 37.8759\,°C$.
Der Solver liefert 37.8643 °C. Die Abweichung von rund 0.01 °C liegt
innerhalb der Standard-Toleranz von `solve_ivp` (rtol=1e-3). Die Lösung
ist korrekt, aber nicht exakt, weil numerische Integration nie fehlerfrei
ist.

Zu Frage 3: Ohne `t_eval` gibt `solve_ivp` die intern gewählten
Auswertungspunkte zurück. Für dieses glatte Problem sind das nur wenige
Schritte (hier 6); die adaptive Schrittweite erlaubt dem RK45-Solver,
große Schritte zu machen, sobald sich die Lösung verlangsamt.

Zu Frage 4: Für diese skalare DGL (1 Zustandsgröße) sind `sol_c.y[0, -1]`
und `sol_c.y[-1, -1]` identisch, weil der erste und der letzte Index in der
ersten Dimension (Länge 1) dieselbe Zeile adressieren. Bei drei
Zustandsgrößen hätte `sol_c.y` die Form `(3, n)`: `sol_c.y[0, -1]`
gäbe den ersten Zustand und `sol_c.y[-1, -1]` den dritten. Sie wären im
Allgemeinen verschieden.

Zu Frage 5: Zeile D gibt eine Liste mit zwei Elementen zurück, obwohl `y0`
nur ein Element hat. `solve_ivp` erwartet, dass die Dimension der Rückgabe
mit der Dimension von `y0` übereinstimmt, und wirft einen `ValueError`.
Korrektur: zweiten Eintrag entfernen, also `return [-k * (y[0] - T_inf)]`.
````

````{admonition} Übung 10.4 (✩✩)
:class: tip
Das **logistische Wachstum** beschreibt, wie eine Größe $N$ zunächst
exponentiell wächst, dann aber durch eine Kapazitätsgrenze $K$ gebremst
wird:

$$\dot{N}(t) = r \cdot N(t) \cdot \left(1 - \frac{N(t)}{K}\right).$$

Ein Hersteller führt eine neue Fertigungstechnologie in einer Fabrik mit
$K = 200$ Maschinen ein. Anfangs sind $N_0 = 10$ Maschinen umgerüstet;
die Ausbreitungsrate beträgt $r = 0.3\,\text{Woche}^{-1}$.

```python
import numpy as np

r_wachs = 0.3    # Wachstumsrate in 1/Woche
K       = 200.0  # Kapazitätsgrenze (Gesamtzahl Maschinen)
N0_log  = 10.0   # Anfangszahl umgerüsteter Maschinen
t_end   = 25.0   # Simulationsende in Wochen
```

1. Schreiben Sie die Funktion `f_logistisch(t, y)` für die rechte Seite
   der DGL und lösen Sie das Anfangswertproblem mit `solve_ivp` für 25
   Wochen. Stellen Sie $N(t)$ in einem Plot dar und markieren Sie die
   Kapazitätsgrenze $K$ als waagerechte gestrichelte Linie.
2. Bestimmen Sie aus dem Plot oder mit `np.searchsorted` den Zeitpunkt,
   an dem $N$ erstmals die Hälfte der Kapazitätsgrenze ($K/2 = 100$)
   überschreitet. Vergleichen Sie mit der analytischen Formel für den
   Wendepunkt:
   $$t^* = \frac{1}{r}\ln\!\left(\frac{K}{N_0} - 1\right).$$
3. Starten Sie die Simulation erneut mit $N_0 = 250 > K$. Was beobachten
   Sie im Plot? Berechnen Sie $f(0) = r \cdot N_0 \cdot (1 - N_0/K)$ von
   Hand, geben Sie das Vorzeichen an und erklären Sie das Verhalten in
   einem Satz.
4. Für welche Werte von $N$ gilt $\dot{N} = 0$? Nennen Sie beide Lösungen
   und begründen Sie, welcher Gleichgewichtspunkt physikalisch stabil und
   welcher instabil ist.
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
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

r_wachs = 0.3; K = 200.0; N0_log = 10.0; t_end = 25.0
t_eval  = np.linspace(0, t_end, 501)

# --- Teilaufgabe 1: solve_ivp ---
def f_logistisch(t, y):
    N = y[0]
    # Logistische Wachstumsrate: positiv für N < K, null bei N = K, negativ für N > K
    return [r_wachs * N * (1 - N / K)]

sol_log = solve_ivp(f_logistisch, (0, t_end), [N0_log], t_eval=t_eval)

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(sol_log.t, sol_log.y[0], color='#005A94', linewidth=2,
        label='$N(t)$: umgerüstete Maschinen')
ax.axhline(K, color='#E87846', linewidth=1.5, linestyle='--',
           label=f'Kapazitätsgrenze $K = {K:.0f}$')
ax.axhline(K / 2, color='#CCDEE9', linewidth=1.2, linestyle=':',
           label=f'Wendepunkt $K/2 = {K/2:.0f}$')
ax.set_xlabel('Zeit in Wochen')
ax.set_ylabel('Anzahl umgerüsteter Maschinen $N$')
ax.set_title('Ausbreitung einer neuen Fertigungstechnologie')
ax.legend()
ax.grid(True)
plt.tight_layout()
plt.show()

print(f"N nach 25 Wochen: {sol_log.y[0, -1]:.2f}  ({sol_log.y[0,-1]/K*100:.1f} % von K)")
```

```python
# --- Teilaufgabe 2: Wendepunkt ---
# Analytisch
t_stern_ana = np.log(K / N0_log - 1) / r_wachs
print(f"Analytischer Wendepunkt t* = {t_stern_ana:.3f} Wochen")

# Numerisch: erster Zeitpunkt, an dem N >= K/2
idx_halb    = np.searchsorted(sol_log.y[0], K / 2)
t_halb_num  = sol_log.t[idx_halb]
print(f"Numerisch (searchsorted):  t ≈ {t_halb_num:.3f} Wochen")
print(f"N an diesem Zeitpunkt:     {sol_log.y[0, idx_halb]:.2f}")
```

Erwartete Ausgabe (Teilaufgabe 2):
```
Analytischer Wendepunkt t* = 9.815 Wochen
Numerisch (searchsorted):  t ≈ 9.850 Wochen
N an diesem Zeitpunkt:     100.16
```

```python
# --- Teilaufgabe 3: N0 > K ---
sol_ueber = solve_ivp(f_logistisch, (0, t_end), [250.0], t_eval=t_eval)

f0_hand = r_wachs * 250.0 * (1 - 250.0 / K)
print(f"f(0) bei N0 = 250: {f0_hand:.2f}  (Vorzeichen: {'negativ' if f0_hand < 0 else 'positiv'})")
print(f"N(5 Wochen)  = {sol_ueber.y[0, 100]:.2f}")
print(f"N(25 Wochen) = {sol_ueber.y[0, -1]:.2f}")

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(sol_log.t,   sol_log.y[0],   color='#005A94', linewidth=2,
        label=f'$N_0 = {N0_log:.0f}$ (Wachstum von unten)')
ax.plot(sol_ueber.t, sol_ueber.y[0], color='#E60000', linewidth=2, linestyle='--',
        label='$N_0 = 250$ (Abfall von oben)')
ax.axhline(K, color='#E87846', linewidth=1.5, linestyle='--',
           label=f'$K = {K:.0f}$')
ax.set_xlabel('Zeit in Wochen')
ax.set_ylabel('$N$')
ax.set_title('Logistisches Wachstum: zwei Startbedingungen')
ax.legend()
ax.grid(True)
plt.tight_layout()
plt.show()
```

Erwartete Ausgabe (Teilaufgabe 3):
```
f(0) bei N0 = 250: -18.75  (Vorzeichen: negativ)
N(5 Wochen)  = 209.44
N(25 Wochen) = 200.03
```

$f(0) = 0.3 \cdot 250 \cdot (1 - 250/200) = 0.3 \cdot 250 \cdot (-0.25) =
-18.75$. Die Änderungsrate ist negativ: Die Technologie ist „überdosiert",
es gibt mehr umgerüstete Maschinen als die Fabrik fassen kann; das System
pendelt sich auf $K = 200$ ein.

Zu Teilaufgabe 4: $\dot{N} = r \cdot N \cdot (1 - N/K) = 0$ gilt für
$N = 0$ und $N = K$. Das Gleichgewicht $N = 0$ ist **instabil**: Eine
kleine Störung $N > 0$ erzeugt $\dot{N} > 0$ und treibt $N$ weg von null.
Das Gleichgewicht $N = K$ ist **stabil**: Für $N < K$ gilt $\dot{N} > 0$
(Wachstum in Richtung $K$), für $N > K$ gilt $\dot{N} < 0$ (Rückfall in
Richtung $K$). Unabhängig vom Startpunkt $N_0 > 0$ konvergiert die Lösung
gegen $K$.
````
