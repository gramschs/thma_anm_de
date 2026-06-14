---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 10.2 Übungen zum Euler-Verfahren

````{admonition} Übung 10.1 (✩)
:class: tip
Gegeben ist der folgende Code, der das Newtonsche Abkühlungsgesetz mit dem
Euler-Verfahren löst.

```python
import numpy as np

def euler_buggy(T0, T_inf, k, t_end, dt):
    n = int(t_end / dt)
    t = np.zeros(n + 1)
    T = np.zeros(n + 1)
    t[0] = 0.0
    T[0] = T0
    for i in range(n - 1):          # Zeile A
        t[i + 1] = t[i] + dt
        T[i + 1] = T[i] + dt * (-k * (T[i] - T_inf))
    return t, T

T0    = 80.0
T_inf = 20.0
k     = 0.1
t_end = 15.0
dt    = 5.0

t_out, T_out = euler_buggy(T0, T_inf, k, t_end, dt)

print("Ausgabe euler_buggy:")
for ti, Ti in zip(t_out, T_out):
    print(f"  t = {ti:.1f} min,  T = {Ti:.4f} °C")
```

1. Welchen Wert hat `n` für die gegebenen Parameter? Über welche Indizes
   läuft `range(n - 1)`? Welcher Zeitschritt fehlt dadurch?
   Beantworten Sie die Fragen ohne Code.
2. Was gibt die letzte Zeile der Ausgabe aus? Erklären Sie den Wert von
   `T_out[-1]` ohne den Code auszuführen.
3. Berechnen Sie den korrekten Euler-Wert von $T$ nach 15 min von Hand.
   Ausgangspunkt: $T(10\,\text{min}) = 35.0\,°C$ (aus den vorherigen
   Schritten). Zeigen Sie den Rechenweg.
4. Nennen Sie den Fehlertyp in Zeile A in einem Satz und geben Sie die
   Korrektur an.
5. Führen Sie den Code aus, korrigieren Sie Zeile A und überprüfen Sie,
   dass `T_out[-1]` nach der Korrektur den in Frage 3 berechneten Wert
   ergibt.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

def euler_buggy(T0, T_inf, k, t_end, dt):
    n = int(t_end / dt)
    t = np.zeros(n + 1)
    T = np.zeros(n + 1)
    t[0] = 0.0
    T[0] = T0
    for i in range(n - 1):          # Fehler: n-1 statt n
        t[i + 1] = t[i] + dt
        T[i + 1] = T[i] + dt * (-k * (T[i] - T_inf))
    return t, T

def euler_korrekt(T0, T_inf, k, t_end, dt):
    n = int(t_end / dt)
    t = np.zeros(n + 1)
    T = np.zeros(n + 1)
    t[0] = 0.0
    T[0] = T0
    for i in range(n):               # Korrektur: range(n) für n Schritte
        t[i + 1] = t[i] + dt
        T[i + 1] = T[i] + dt * (-k * (T[i] - T_inf))
    return t, T

T0, T_inf, k, t_end, dt = 80.0, 20.0, 0.1, 15.0, 5.0

print("Ausgabe euler_buggy:")
t_b, T_b = euler_buggy(T0, T_inf, k, t_end, dt)
for ti, Ti in zip(t_b, T_b):
    print(f"  t = {ti:.1f} min,  T = {Ti:.4f} °C")

print()
print("Ausgabe euler_korrekt:")
t_k, T_k = euler_korrekt(T0, T_inf, k, t_end, dt)
for ti, Ti in zip(t_k, T_k):
    print(f"  t = {ti:.1f} min,  T = {Ti:.4f} °C")
```

Erwartete Ausgabe:
```
Ausgabe euler_buggy:
  t = 0.0 min,  T = 80.0000 °C
  t = 5.0 min,  T = 50.0000 °C
  t = 10.0 min,  T = 35.0000 °C
  t = 0.0 min,  T = 0.0000 °C

Ausgabe euler_korrekt:
  t = 0.0 min,  T = 80.0000 °C
  t = 5.0 min,  T = 50.0000 °C
  t = 10.0 min,  T = 35.0000 °C
  t = 15.0 min,  T = 27.5000 °C
```

Zu Frage 1: Für `t_end = 15.0` und `dt = 5.0` gilt `n = 3`. Damit läuft
`range(n - 1) = range(2)` über die Indizes `[0, 1]`. Es werden also nur
zwei Schritte berechnet (Intervalle $[0, 5]$ und $[5, 10]$); der dritte
Schritt von $t = 10$ auf $t = 15$ min fehlt.

Zu Frage 2: Die letzte Zeile gibt `t = 0.0 min, T = 0.0000 °C` aus.
Beide Einträge `t[3]` und `T[3]` wurden nie beschrieben. NumPy-Arrays
werden bei der Erzeugung mit `np.zeros` auf null initialisiert. Da der
dritte Schritt in der Schleife nicht ausgeführt wird, stehen in diesen
Feldern weiterhin die Initialwerte 0.0.

Zu Frage 3: Ausgehend von $T(10\,\text{min}) = 35.0\,°C$:
$$T(15) = 35.0 + 5 \cdot (-0.1) \cdot (35.0 - 20.0) = 35.0 - 7.5 = 27.5\,°C.$$

Zu Frage 4: Es handelt sich um einen **Off-by-One-Fehler** (Zaunpfahlfehler).
Das Intervall $[0, t_{end}]$ wird in $n=t_{end}/\Delta t$ Schritte zerlegt. Wir
benötigen daher neua $n$ Euler-Updates, also `range(n)` in Zeile A.
````

````{admonition} Übung 10.2 (✩✩)
:class: tip
Der **radioaktive Zerfall** eines instabilen Isotops folgt demselben
mathematischen Grundprinzip wie das Abkühlungsgesetz: Die Zerfallsrate ist
proportional zur aktuellen Atomzahl $N$:

$$\dot{N}(t) = -\lambda \cdot N(t).$$

Gegeben: Zerfallskonstante $\lambda = 0.2\,\text{min}^{-1}$,
Anfangszahl $N_0 = 1000$ Atome. Die analytische Lösung lautet
$N(t) = N_0\,e^{-\lambda t}$.

Der folgende Code erzeugt die Ausgangsdaten:

```python
import numpy as np

lambda_zerfall = 0.2    # Zerfallskonstante in 1/min
N0             = 1000   # Anfangszahl der Atome
t_end          = 20.0   # Simulationsende in min
```

1. Implementieren Sie das Euler-Verfahren für diese DGL mit Schrittweite
   $\Delta t = 0.5\,\text{min}$. Geben Sie $N(10\,\text{min})$ numerisch
   und analytisch aus. Berechnen Sie den relativen Fehler in Prozent.
2. Stellen Sie den Euler-Verlauf und die analytische Lösung in einem
   gemeinsamen Plot dar. Beschriften Sie die Achsen mit Einheiten und
   fügen Sie eine Legende ein.
3. Setzen Sie $\Delta t = 6.0\,\text{min}$ und führen Sie die Simulation
   erneut durch. Was stellen Sie an den Ausgabewerten fest? Ist das
   Ergebnis physikalisch sinnvoll? Begründen Sie in einem Satz.
4. Berechnen Sie $N_1 = N_0 + \Delta t \cdot (-\lambda N_0)$ von Hand für
   $\Delta t = 6.0\,\text{min}$. Für welche Schrittweite $\Delta t$ gilt
   genau $N_1 = 0$? Was passiert beim nächsten Schritt, wenn $N_i = 0$?
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

# --- Gegebene Größen ---
lambda_zerfall = 0.2    # Zerfallskonstante in 1/min
N0             = 1000   # Anfangszahl der Atome
t_end          = 20.0   # Simulationsende in min

# --- Teilaufgabe 1: Euler-Verfahren mit dt = 0.5 min ---
dt         = 0.5
n_schritte = int(t_end / dt)
t_euler    = np.zeros(n_schritte + 1)
N_euler    = np.zeros(n_schritte + 1)
t_euler[0] = 0.0
N_euler[0] = N0

for i in range(n_schritte):
    # Euler-Update: N[i+1] = N[i] + dt * (-lambda * N[i])
    t_euler[i + 1] = t_euler[i] + dt
    N_euler[i + 1] = N_euler[i] + dt * (-lambda_zerfall * N_euler[i])

# Analytische Referenz
idx_10    = int(10.0 / dt)
N_exakt_10 = N0 * np.exp(-lambda_zerfall * 10.0)
rel_fehler = abs(N_euler[idx_10] - N_exakt_10) / N_exakt_10 * 100

print(f"Euler bei t = 10 min:       {N_euler[idx_10]:.2f} Atome")
print(f"Analytisch bei t = 10 min:  {N_exakt_10:.2f} Atome")
print(f"Relativer Fehler:           {rel_fehler:.2f} %")

# --- Teilaufgabe 2: Plot ---
t_fein  = np.linspace(0, t_end, 500)
N_fein  = N0 * np.exp(-lambda_zerfall * t_fein)

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(t_fein, N_fein, color='#005A94', linewidth=2,
        label='Analytische Lösung $N(t) = N_0\\,e^{-\\lambda t}$')
ax.plot(t_euler, N_euler, color='#E60000', linewidth=1.5, linestyle='--',
        marker='o', markersize=4, label=f'Euler ($\\Delta t = {dt}$ min)')
ax.set_xlabel('Zeit in min')
ax.set_ylabel('Atomzahl $N$')
ax.set_title('Radioaktiver Zerfall: Euler-Verfahren vs. analytische Lösung')
ax.legend()
ax.grid(True)
plt.tight_layout()
plt.show()
```

Erwartete Ausgabe (Teilaufgabe 1):
```
Euler bei t = 10 min:       121.58 Atome
Analytisch bei t = 10 min:  135.34 Atome
Relativer Fehler:           10.17 %
```

```python
# --- Teilaufgabe 3: Euler mit großer Schrittweite dt = 6 min ---
dt_gross   = 6.0
n_gross    = int(t_end / dt_gross)
t_gross    = np.zeros(n_gross + 1)
N_gross    = np.zeros(n_gross + 1)
t_gross[0] = 0.0
N_gross[0] = N0

for i in range(n_gross):
    t_gross[i + 1] = t_gross[i] + dt_gross
    N_gross[i + 1] = N_gross[i] + dt_gross * (-lambda_zerfall * N_gross[i])

print("Euler mit dt = 6 min:")
for ti, Ni in zip(t_gross, N_gross):
    print(f"  t = {ti:.1f} min,  N = {Ni:.4f}")
```

Erwartete Ausgabe (Teilaufgabe 3):
```
Euler mit dt = 6 min:
  t = 0.0 min,  N = 1000.0000
  t = 6.0 min,  N = -200.0000
  t = 12.0 min,  N = 40.0000
  t = 18.0 min,  N = -8.0000
```

Zu Teilaufgabe 3: Die Atomzahl wechselt zwischen positiven und negativen
Werten. Eine negative Atomzahl ist physikalisch sinnlos; das Euler-Verfahren
produziert hier ein unphysikalisches Ergebnis, weil die Schrittweite zu groß
gewählt wurde.

Zu Teilaufgabe 4: Handrechnung für $\Delta t = 6\,\text{min}$:
$$N_1 = 1000 + 6 \cdot (-0.2 \cdot 1000) = 1000 - 1200 = -200.$$

$N_1 = 0$ gilt genau wenn $\Delta t \cdot \lambda = 1$, also
$\Delta t = 1/\lambda = 5\,\text{min}$. Das ist die **kritische Schrittweite**:
Beim Grenzfall $\Delta t = 5\,\text{min}$ fällt $N_1 = 0$ und alle weiteren
Werte bleiben null, weil das Euler-Update $N_{i+1} = N_i + \Delta t \cdot
(-\lambda N_i) = N_i (1 - \lambda \Delta t)$ dann $N_i \cdot 0 = 0$ ergibt.
Für $\Delta t > 5\,\text{min}$ wird der Verstärkungsfaktor $(1 - \lambda
\Delta t)$ negativ, was den Vorzeichenwechsel erklärt. Das Euler-Verfahren
ist dann numerisch instabil in dem Sinne, dass es unphysikalische Werte
erzeugt, selbst wenn die Lösung formal konvergiert.
````
