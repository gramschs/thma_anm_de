---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 11.2 Übungen zum Federschwinger

````{admonition} Übung 11.1 (✩)
:class: tip
Der folgende Code simuliert einen Federschwinger ($m = 250\,\text{kg}$,
$k = 16000\,\text{N/m}$), enthält aber einen Fehler in der Funktion `f`.
Der Integrator löst das aufgestellte System mathematisch korrekt. Das
Ergebnis ist dennoch falsch, weil `f` die DGL falsch modelliert: Es handelt
sich um einen Modellfehler, nicht um einen Numerikfehler.

```python
from scipy.integrate import solve_ivp
import numpy as np

m       = 250.0
k_feder = 16000.0
x0      = 0.05    # Anfangsauslenkung in m
v0      = 0.0     # Anfangsgeschwindigkeit in m/s
t_end   = 5.0     # Simulationsende in s

def f_schwinger_buggy(t, y):
    return [y[1], -y[0]]    # Zeile A

t_eval = np.linspace(0, t_end, 501)
sol    = solve_ivp(f_schwinger_buggy, (0, t_end), [x0, v0], t_eval=t_eval)

T_buggy   = 2 * np.pi
T_korrekt = 2 * np.pi / np.sqrt(k_feder / m)

print(f"Periode buggy:   {T_buggy:.4f} s")
print(f"Periode korrekt: {T_korrekt:.4f} s")
print(f"Abweichung:      {abs(T_buggy - T_korrekt) / T_korrekt * 100:.1f} %")
```

1. Was berechnet Zeile A tatsächlich? Welche DGL löst der Code, und welche
   Eigenkreisfrequenz hat sie? Antworten Sie ohne Code.
2. Um welchen Faktor weicht die berechnete Periode von der korrekten ab?
   Berechnen Sie das Verhältnis $T_\text{buggy}/T_\text{korrekt}$ aus den
   Eigenkreisfrequenzen.
3. Nennen Sie die Korrektur in Zeile A in einem Satz.
4. Führen Sie den Code aus, korrigieren Sie Zeile A und überprüfen Sie,
   dass die korrigierte Periode mit `T_korrekt` übereinstimmt.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: Zeile A gibt `[y[1], -y[0]]` zurück. Das entspricht der DGL
$\ddot{x} = -x$, also einem Schwinger mit $\omega_\text{buggy} = \sqrt{1}
= 1\,\text{rad/s}$ statt $\omega_0 = \sqrt{k/m} = 8\,\text{rad/s}$.
Der Integrator hat dabei keine Fehler gemacht; er hat exakt die falsch
modellierte DGL gelöst.

Zu Frage 2: $T_\text{buggy}/T_\text{korrekt} = \omega_0/\omega_\text{buggy}
= 8/1 = 8$. Die berechnete Periode ist achtmal zu groß.

Zu Frage 3: Korrektur: `return [y[1], -(k_feder / m) * y[0]]`.

Zu Frage 4: Erwartete Ausgabe vor der Korrektur:

```
Periode buggy:   6.2832 s
Periode korrekt: 0.7854 s
Abweichung:      700.0 %
```

Nach der Korrektur stimmen beide Periodenwerte überein:

```python
def f_schwinger_korrekt(t, y):
    return [y[1], -(k_feder / m) * y[0]]

sol = solve_ivp(f_schwinger_korrekt, (0, t_end), [x0, v0], t_eval=t_eval)
# Im Plot sieht man jetzt ca. 6 vollständige Schwingungen in 5 s (T_0 ≈ 0.785 s)
```
````

````{admonition} Übung 11.2 (✩✩)
:class: tip
Das nichtlineare Pendel der Länge $L$ genügt der DGL

$$\ddot{\phi} = -\frac{g}{L}\sin\phi.$$

Für kleine Winkel gilt $\sin\phi \approx \phi$, und die DGL vereinfacht
sich zur linearen Näherung mit der Periode $T_0 = 2\pi\sqrt{L/g}$.
Gegeben: $g = 9.81\,\text{m/s}^2$, $L = 1.0\,\text{m}$,
Anfangsgeschwindigkeit $\dot{\phi}(0) = 0$ für beide Fälle.

1. Stellen Sie die DGL als System erster Ordnung auf. Definieren Sie
   $y_1 = \phi$ und $y_2 = \dot{\phi}$ und schreiben Sie `f(t, y)` auf.
2. Lösen Sie mit `solve_ivp` für beide Anfangsbedingungen
   $\phi_0 = 5°$ und $\phi_0 = 90°$ (Winkel zuerst in Bogenmaß umrechnen).
   Stellen Sie $\phi(t)$ für $t \in [0, 10\,\text{s}]$ dar.
3. Lesen Sie die Schwingungsperiode für beide Fälle aus dem Plot ab und
   vergleichen Sie mit der analytischen Kleinwinkelperiode
   $T_0 = 2\pi\sqrt{L/g}$.
4. Warum ist die Periode für $\phi_0 = 90°$ länger als $T_0$? Erklären Sie
   in einem Satz ohne Code.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: Mit $y_1 = \phi$ und $y_2 = \dot{\phi}$ gilt $\dot{y}_1 = y_2$
und $\dot{y}_2 = -(g/L)\sin(y_1)$. Die Funktion lautet:

```python
def f_pendel(t, y):
    return [y[1], -(g / L) * np.sin(y[0])]
```

Zu Fragen 2 und 3:

```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

g = 9.81
L = 1.0
T_0 = 2 * np.pi * np.sqrt(L / g)

def f_pendel(t, y):
    return [y[1], -(g / L) * np.sin(y[0])]

t_eval = np.linspace(0, 10, 1001)
farben = ['#005A94', '#E60000']
winkel = [5, 90]

fig, ax = plt.subplots(figsize=(8, 4))
for phi_grad, farbe in zip(winkel, farben):
    phi0 = np.radians(phi_grad)
    sol  = solve_ivp(f_pendel, (0, 10), [phi0, 0.0], t_eval=t_eval)
    ax.plot(t_eval, np.degrees(sol.y[0]), color=farbe,
            label=f'φ₀ = {phi_grad}°')

ax.set_xlabel('Zeit in s')
ax.set_ylabel('Winkel in Grad')
ax.set_title(f'Nichtlineares Pendel  (T₀ = {T_0:.3f} s)')
ax.legend()
plt.tight_layout()
plt.show()

print(f"Kleinwinkelperiode T_0 = {T_0:.4f} s")
```

Ausgabe: `Kleinwinkelperiode T_0 = 2.0064 s`

Aus dem Plot lassen sich die Perioden ablesen:
$T(5°) \approx 2.007\,\text{s}$ (Abweichung $< 0.1\,\%$),
$T(90°) \approx 2.365\,\text{s}$ (Abweichung $\approx 17.9\,\%$).

Zu Frage 4: Für $\phi > 0$ gilt $\sin\phi < \phi$: Die tatsächliche
Rückstellkraft ist schwächer als die lineare Näherung, daher dauert eine
Schwingung bei großen Auslenkungen länger.
````
