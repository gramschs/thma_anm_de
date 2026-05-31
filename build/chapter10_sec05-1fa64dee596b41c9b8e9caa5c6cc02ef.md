---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 10.5 Übungen

````{admonition} Übung 10.5 (✩)
:class: tip
Ein Raum kühlt nachts gegen die kalte Außenluft ab. Um 06:00 Uhr (im
Modell $t = 10\,\text{min}$) springt der Thermostat an und erhöht die
Zieltemperatur. Die Raumtemperatur folgt der DGL

$$\dot{T}(t) = -k\bigl(T(t) - T_\infty(t)\bigr), \qquad
T_\infty(t) = \begin{cases} 16\,°C & t < 10\,\text{min} \\ 22\,°C & t \geq 10\,\text{min} \end{cases}$$

mit $k = 0.15\,\text{min}^{-1}$ und Anfangstemperatur $T_0 = 26\,°C$.

```python
from scipy.integrate import solve_ivp
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

k     = 0.15    # Abkühlkonstante in 1/min
T0    = 26.0    # Anfangstemperatur in °C
t_end = 30.0    # Simulationsende in min

def T_inf_thermostat(t):
    # Zieltemperatur des Thermostats: nachts kälter, ab t=10 min wärmer
    if t < 10.0:
        return 16.0    # Nacht: Heizung aus
    else:
        return 22.0    # Tag: Heizung ein

def f_thermostat(t, y):
    return [-k * (y[0] - T_inf_thermostat(t))]

t_eval = np.linspace(0, t_end, 601)
sol    = solve_ivp(f_thermostat, (0, t_end), [T0],
                   t_eval=t_eval, max_step=0.1)

print(f"sol.success:  {sol.success}")
print(f"T(0 min)  = {sol.y[0,   0]:.4f} °C")
print(f"T(10 min) = {sol.y[0, 200]:.4f} °C")
print(f"T(15 min) = {sol.y[0, 300]:.4f} °C")
print(f"T(30 min) = {sol.y[0,  -1]:.4f} °C")

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(sol.t, sol.y[0], color='#005A94', linewidth=2, label='$T(t)$')
ax.axhline(16.0, color='#CCDEE9', linewidth=1.2, linestyle=':',
           label='$T_\\infty = 16\\,°C$ (Nacht)')
ax.axhline(22.0, color='#E87846', linewidth=1.2, linestyle=':',
           label='$T_\\infty = 22\\,°C$ (Tag)')
ax.axvline(10.0, color='#484949', linewidth=1.0, linestyle='--',
           label='Thermostat-Schaltpunkt')
ax.set_xlabel('Zeit in min')
ax.set_ylabel('Temperatur in °C')
ax.set_title('Raumtemperatur mit Thermostat-Schaltung')
ax.legend(fontsize=9)
ax.grid(True)
plt.tight_layout()
plt.show()
```

1. Zeichnen Sie $T_\infty(t)$ für $t \in [0, 30]$ min von Hand als Skizze.
   Um was für eine Funktion handelt es sich?
2. Ist $T(t)$ bei $t = 10\,\text{min}$ stetig? Ist $\dot{T}(t)$ stetig?
   Begründen Sie ohne Code.
3. Berechnen Sie $\dot{T}(0)$ von Hand. Berechnen Sie anschließend das
   Vorzeichen von $\dot{T}(15\,\text{min})$, indem Sie $T(15) \approx 20\,°C$
   als Näherung verwenden. Was bedeuten die Vorzeichen physikalisch?
4. Wo hat $T(t)$ ein Minimum, und warum tritt es genau dort auf?
   Erklären Sie in einem Satz ohne Code.
5. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
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
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

k = 0.15; T0 = 26.0; t_end = 30.0

def T_inf_thermostat(t):
    return 16.0 if t < 10.0 else 22.0

def f_thermostat(t, y):
    return [-k * (y[0] - T_inf_thermostat(t))]

t_eval = np.linspace(0, t_end, 601)
sol    = solve_ivp(f_thermostat, (0, t_end), [T0], t_eval=t_eval, max_step=0.1)

print(f"sol.success:  {sol.success}")
print(f"T(0 min)  = {sol.y[0,   0]:.4f} °C")
print(f"T(10 min) = {sol.y[0, 200]:.4f} °C")
print(f"T(15 min) = {sol.y[0, 300]:.4f} °C")
print(f"T(30 min) = {sol.y[0,  -1]:.4f} °C")
```

Erwartete Ausgabe:
```
sol.success:  True
T(0 min)  = 26.0000 °C
T(10 min) = 18.2313 °C
T(15 min) = 20.2159 °C
T(30 min) = 21.8120 °C
```

Zu Frage 1: $T_\infty(t)$ ist eine **Sprungfunktion** (Heaviside-artig):
konstant bei 16 °C für $t < 10$, dann ein sofortiger Sprung auf 22 °C.

Zu Frage 2: $T(t)$ ist stetig, weil die Integration keine Sprünge einführt
(der Solver integriert eine beschränkte rechte Seite über ein kompaktes
Intervall). $\dot{T}(t)$ hat bei $t = 10\,\text{min}$ eine
**Unstetigkeitsstelle**: Die Ableitung springt, weil $T_\infty$ springt,
$T$ selbst aber nicht.

Zu Frage 3:
$$\dot{T}(0) = -0.15 \cdot (26 - 16) = -1.5\,°C/\text{min} \quad (\text{negativ: Abkühlung}).$$
Mit der Näherung $T(15) \approx 20\,°C$:
$$\dot{T}(15) \approx -0.15 \cdot (20 - 22) = +0.3\,°C/\text{min} \quad (\text{positiv: Heizung}).$$
Vor $t = 10\,\text{min}$ kühlt der Raum ab (Ziel 16 °C liegt unter der
Anfangstemperatur); nach dem Schaltpunkt heizt der Raum auf (Ziel 22 °C
liegt über der aktuellen Temperatur).

Zu Frage 4: Das Minimum liegt bei $t = 10\,\text{min}$ genau am
Thermostat-Schaltpunkt: Für $t < 10$ zieht $T_\infty = 16\,°C$ die
Temperatur nach unten, für $t \geq 10$ zieht $T_\infty = 22\,°C > T(10)$
sie wieder nach oben. Das lokale Minimum entsteht genau dort, wo die
Richtung der Triebkraft umkehrt.
````

````{admonition} Übung 10.6 (✩✩)
:class: tip
Ein Kondensator $C = 100\,\mu F$ wird über einen Widerstand $R = 1000\,\Omega$
an eine Spannungsquelle $U_0 = 12\,V$ angeschlossen. Die Spannung am
Kondensator $U_C$ genügt der DGL

$$\dot{U}_C = \frac{U_0(t) - U_C}{\tau}, \qquad \tau = RC.$$

Bei $t = 0$ ist der Kondensator ungeladen ($U_C(0) = 0$). Nach einer
Zeitkonstante ($t = \tau$) wird die Spannungsquelle abgeschaltet
($U_0 \to 0$), sodass der Kondensator über denselben Widerstand entlädt.

```python
import numpy as np

U0  = 12.0         # Quellspannung in V
R   = 1000.0       # Widerstand in Ohm
C   = 100e-6       # Kapazität in F
tau = R * C        # Zeitkonstante in s
```

1. Implementieren Sie `f_rc_laden(t, y)` für den reinen Ladevorgang
   ($U_0 = 12\,V$, autonom) und lösen Sie die DGL für $5\tau$ Sekunden.
   Stellen Sie $U_C(t)$ dar. Zeichnen Sie $\tau$ auf der $t$-Achse und
   $U_0(1 - 1/e) \approx 7.58\,V$ auf der $y$-Achse als Hilfslinie ein.
2. Bestimmen Sie numerisch mit `np.searchsorted`, wann $U_C$ erstmals
   99 % von $U_0$ erreicht. Vergleichen Sie mit der analytischen Formel
   $t_{99} = -\tau\,\ln(0.01)$.
3. Implementieren Sie `f_rc_nichtautonom(t, y)` als nichtautonome DGL:
   $U_0(t) = 12\,V$ für $t < \tau$, sonst $U_0(t) = 0$. Simulieren Sie
   für $4\tau$ Sekunden und stellen Sie Lade- und Entladevorgang in
   einem gemeinsamen Plot dar.
4. Lesen Sie die Zeitkonstante aus dem Entladeverlauf ab: Bei $t = \tau$
   nach dem Abschalten (also bei $t = 2\tau$ insgesamt) sollte
   $U_C(2\tau) = U_C(\tau)/e$ gelten. Prüfen Sie numerisch.
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

U0 = 12.0; R = 1000.0; C = 100e-6; tau = R * C
print(f"Zeitkonstante tau = RC = {tau:.4f} s")

# --- Teilaufgabe 1: Ladevorgang (autonom) ---
def f_rc_laden(t, y):
    # Ladegleichung: dUC/dt = (U0 - UC) / tau
    return [(U0 - y[0]) / tau]

t_eval_l = np.linspace(0, 5 * tau, 1001)
sol_laden = solve_ivp(f_rc_laden, (0, 5 * tau), [0.0], t_eval=t_eval_l)

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(sol_laden.t, sol_laden.y[0], color='#005A94', linewidth=2,
        label='$U_C(t)$ Ladevorgang')
ax.axvline(tau, color='#484949', linewidth=1.0, linestyle='--',
           label=f'$\\tau = {tau:.3f}$ s')
ax.axhline(U0 * (1 - 1/np.e), color='#E87846', linewidth=1.2, linestyle=':',
           label=f'$U_0(1-1/e) = {U0*(1-1/np.e):.2f}$ V')
ax.axhline(U0, color='#CCDEE9', linewidth=1.0, linestyle=':',
           label=f'$U_0 = {U0:.0f}$ V')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Spannung $U_C$ in V')
ax.set_title('RC-Schaltung: Ladevorgang')
ax.legend(fontsize=9)
ax.grid(True)
plt.tight_layout()
plt.show()
```

```python
# --- Teilaufgabe 2: Ladezeit bis 99 % ---
t_99_ana = -tau * np.log(0.01)
print(f"Analytisch t_99 = -tau*ln(0.01) = {t_99_ana:.4f} s = {t_99_ana/tau:.3f} tau")

idx_99   = np.searchsorted(sol_laden.y[0], 0.99 * U0)
t_99_num = sol_laden.t[idx_99]
print(f"Numerisch  t_99 = {t_99_num:.4f} s = {t_99_num/tau:.3f} tau")
print(f"UC an diesem Punkt: {sol_laden.y[0, idx_99]:.4f} V")
```

Erwartete Ausgabe:
```
Analytisch t_99 = -tau*ln(0.01) = 0.4605 s = 4.605 tau
Numerisch  t_99 = 0.4610 s = 4.610 tau
UC an diesem Punkt: 11.8918 V
```

```python
# --- Teilaufgabe 3: Nichtautonome Simulation (Laden + Entladen) ---
def f_rc_nichtautonom(t, y):
    # Quellspannung schaltet bei t = tau ab
    U_quelle = U0 if t < tau else 0.0
    return [(U_quelle - y[0]) / tau]

t_eval_f = np.linspace(0, 4 * tau, 2001)
sol_full  = solve_ivp(f_rc_nichtautonom, (0, 4 * tau), [0.0],
                      t_eval=t_eval_f, max_step=tau / 100)

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(sol_full.t, sol_full.y[0], color='#005A94', linewidth=2,
        label='$U_C(t)$')
ax.axvline(tau, color='#484949', linewidth=1.0, linestyle='--',
           label=f'Abschalten bei $t = \\tau = {tau:.3f}$ s')
ax.axhline(0, color='#CCDEE9', linewidth=0.8, linestyle=':')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Spannung $U_C$ in V')
ax.set_title('RC-Schaltung: Laden und Entladen')
ax.legend(fontsize=9)
ax.grid(True)
plt.tight_layout()
plt.show()
```

```python
# --- Teilaufgabe 4: Zeitkonstante aus der Entladekurve ablesen ---
# Index für t = tau (Schaltzeitpunkt)
idx_tau  = np.searchsorted(sol_full.t, tau)
UC_tau   = sol_full.y[0, idx_tau]      # UC direkt nach dem Abschalten

# Index für t = 2*tau (eine Zeitkonstante nach dem Abschalten)
idx_2tau = np.searchsorted(sol_full.t, 2 * tau)
UC_2tau  = sol_full.y[0, idx_2tau]

print(f"UC(tau)       = {UC_tau:.4f} V  (= U0*(1-1/e) = {U0*(1-1/np.e):.4f} V)")
print(f"UC(2*tau)     = {UC_2tau:.4f} V")
print(f"UC(tau)/e     = {UC_tau / np.e:.4f} V")
print(f"Verhältnis UC(2tau)/UC(tau) = {UC_2tau/UC_tau:.4f}  (1/e = {1/np.e:.4f})")
```

Erwartete Ausgabe:
```
UC(tau)       = 7.5793 V  (= U0*(1-1/e) = 7.5854 V)
UC(2*tau)     = 2.7892 V
UC(tau)/e     = 2.7883 V
Verhältnis UC(2tau)/UC(tau) = 0.3680  (1/e = 0.3679)
```

Das Verhältnis $U_C(2\tau)/U_C(\tau) \approx 1/e$ bestätigt, dass die
Zeitkonstante der Entladung ebenfalls $\tau = RC$ beträgt. Das ist
physikalisch konsistent: Laden und Entladen laufen beide über denselben
Widerstand $R$ und Kondensator $C$, also mit derselben Zeitkonstante.
````

````{admonition} Übung 10.7 (✩✩✩)
:class: tip
Ein Fallschirmspringer springt aus dem Flugzeug und öffnet den Schirm
noch nicht. Neben der Schwerkraft wirkt eine quadratische Luftreibung:

$$\dot{v}(t) = g - k_q\,v(t)^2, \qquad v(0) = 0.$$

Die Endgeschwindigkeit (Gleichgewicht $\dot{v} = 0$) ist
$v_\infty = \sqrt{g/k_q}$. Die analytische Lösung lautet
$v(t) = v_\infty \tanh\!\bigl(g\,t/v_\infty\bigr)$.

```python
import numpy as np

g    = 9.81    # Schwerebeschleunigung in m/s^2
k_q  = 0.006   # spezifischer Luftwiderstandsbeiwert in 1/m
```

**Teil 1 – Geschwindigkeitsverlauf**

1. Implementieren Sie `f_fall_quad(t, y)` und lösen Sie die DGL für 60 s.
   Stellen Sie $v(t)$ dar und markieren Sie $v_\infty$ als waagerechte
   Linie. Berechnen Sie $v_\infty = \sqrt{g/k_q}$ und vergleichen Sie den
   numerischen Endwert $v(60\,\text{s})$ damit.
2. Überlagern Sie die analytische Lösung
   $v_\text{ana}(t) = v_\infty\,\tanh(g\,t/v_\infty)$ im selben Plot.
   Berechnen Sie den maximalen absoluten Fehler zwischen `solve_ivp` und
   der analytischen Lösung.

**Teil 2 – Parameterstudie**

3. Erstellen Sie eine Kurvenschar: Lösen Sie die DGL für vier Werte
   $k_q \in \{0.003,\, 0.006,\, 0.012,\, 0.024\}\,\text{m}^{-1}$ und
   stellen Sie alle vier Kurven in einem Plot dar. Geben Sie die
   zugehörigen Endgeschwindigkeiten als Tabelle aus.

**Teil 3 – Zurückgelegte Strecke**

4. Berechnen Sie die zurückgelegte Strecke $h(t) = \int_0^t v(\xi)\,\mathrm{d}\xi$
   mit `scipy.integrate.cumulative_trapezoid` auf der `solve_ivp`-Ausgabe
   (kein neues Gleichungssystem nötig). Stellen Sie $h(t)$ in einem zweiten
   Plot dar. Lesen Sie ab: Nach wie vielen Sekunden hat der Springer
   1000 m zurückgelegt?

**Teil 4 – Vergleich mit linearem Luftwiderstand**

5. Der lineare Luftwiderstand führt auf $\dot{v} = g - k_l\,v$ mit
   $v_\infty = g/k_l$. Wählen Sie $k_l = g/v_\infty$ so, dass beide
   Modelle dieselbe Endgeschwindigkeit haben, und lösen Sie beide DGLen.
   Plotten Sie $v_\text{lin}(t)$ und $v_\text{quad}(t)$ gemeinsam.
   Welches Modell erreicht $v_\infty$ schneller? Berechnen Sie
   $v(\tau)/v_\infty$ für beide Modelle bei $\tau = v_\infty/g$ und
   erklären Sie den Unterschied in zwei Sätzen.
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
from scipy.integrate import solve_ivp, cumulative_trapezoid
style.use('seaborn-v0_8')

g   = 9.81; k_q = 0.006
v_end = np.sqrt(g / k_q)
print(f"v_end = sqrt({g}/{k_q}) = {v_end:.4f} m/s = {v_end*3.6:.1f} km/h")

def f_fall_quad(t, y):
    # Freier Fall mit quadratischem Luftwiderstand
    # dv/dt = g - k_q * v^2
    return [g - k_q * y[0]**2]

t_eval = np.linspace(0, 60, 601)
sol    = solve_ivp(f_fall_quad, (0, 60), [0.0], t_eval=t_eval)

print(f"v(60 s)        = {sol.y[0, -1]:.4f} m/s")
print(f"v(60 s)/v_end  = {sol.y[0, -1]/v_end:.6f}")

# --- Teilaufgaben 1 und 2 ---
v_ana = v_end * np.tanh(g * t_eval / v_end)
max_err = np.max(np.abs(sol.y[0] - v_ana))
print(f"Max. Fehler solve_ivp vs. analytisch: {max_err:.2e} m/s")

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(t_eval, v_ana,     color='#005A94', linewidth=2.5,
        label='Analytisch $v_\\infty \\tanh(gt/v_\\infty)$')
ax.plot(t_eval, sol.y[0],  color='#E60000', linewidth=1.5, linestyle=':',
        label='solve\\_ivp (RK45)')
ax.axhline(v_end, color='#E87846', linewidth=1.2, linestyle='--',
           label=f'$v_\\infty = {v_end:.1f}$ m/s')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Geschwindigkeit in m/s')
ax.set_title('Freier Fall mit quadratischem Luftwiderstand')
ax.legend(fontsize=9)
ax.grid(True)
plt.tight_layout()
plt.show()
```

Erwartete Ausgabe:
```
v_end = sqrt(9.81/0.006) = 40.4351 m/s = 145.6 km/h
v(60 s)        = 40.4318 m/s
v(60 s)/v_end  = 0.999917
Max. Fehler solve_ivp vs. analytisch: 3.22e-02 m/s
```

```python
# --- Teilaufgabe 3: Parameterstudie ---
k_q_werte = [0.003, 0.006, 0.012, 0.024]
farben     = ['#005A94', '#E87846', '#E60000', '#484949']

fig, ax = plt.subplots(figsize=(8, 4))
print(f"{'k_q [1/m]':>12}  {'v_end [m/s]':>12}  {'v_end [km/h]':>14}")
print("-" * 44)

for k_qi, farbe in zip(k_q_werte, farben):
    v_endi = np.sqrt(g / k_qi)
    print(f"{k_qi:>12.3f}  {v_endi:>12.2f}  {v_endi*3.6:>14.1f}")
    def f_i(t, y, k=k_qi):
        return [g - k * y[0]**2]
    sol_i = solve_ivp(f_i, (0, 60), [0.0], t_eval=t_eval)
    ax.plot(t_eval, sol_i.y[0], color=farbe, linewidth=2,
            label=f'$k_q = {k_qi}$ → $v_\\infty = {v_endi:.1f}$ m/s')

ax.set_xlabel('Zeit in s')
ax.set_ylabel('Geschwindigkeit in m/s')
ax.set_title('Parameterstudie: Einfluss des Luftwiderstandsbeiwertes')
ax.legend(fontsize=9)
ax.grid(True)
plt.tight_layout()
plt.show()
```

Erwartete Tabelle:
```
   k_q [1/m]   v_end [m/s]   v_end [km/h]
--------------------------------------------
       0.003         57.18          205.9
       0.006         40.44          145.6
       0.012         28.59          102.9
       0.024         20.22           72.8
```

```python
# --- Teilaufgabe 4: Zurückgelegte Strecke ---
# v(t) aus solve_ivp mit k_q = 0.006 (bereits berechnet)
h = cumulative_trapezoid(sol.y[0], sol.t, initial=0)

print(f"h(10 s) = {h[100]:.1f} m")
print(f"h(30 s) = {h[300]:.1f} m")
print(f"h(60 s) = {h[600]:.1f} m")

# Wann wird h = 1000 m erstmals überschritten?
idx_1km = np.searchsorted(h, 1000.0)
print(f"h = 1000 m bei t = {sol.t[idx_1km]:.1f} s")

# Analytisch: h(t) = v_end^2/g * ln(cosh(g*t/v_end))
h_ana = v_end**2 / g * np.log(np.cosh(g * t_eval / v_end))
print(f"h(60 s) analytisch = {h_ana[-1]:.1f} m  (numerisch: {h[-1]:.1f} m)")

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(sol.t, h, color='#005A94', linewidth=2,
        label='zurückgelegte Strecke $h(t)$')
ax.axhline(1000, color='#E87846', linewidth=1.2, linestyle='--',
           label='1000 m')
ax.scatter([sol.t[idx_1km]], [h[idx_1km]], color='#E60000', s=60, zorder=5)
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Strecke in m')
ax.set_title('Zurückgelegte Strecke (Integration der Geschwindigkeit)')
ax.legend()
ax.grid(True)
plt.tight_layout()
plt.show()
```

Erwartete Ausgabe:
```
h(10 s) =  290.1 m
h(30 s) = 1097.6 m
h(60 s) = 2310.5 m
h = 1000 m bei t = 27.3 s
h(60 s) analytisch = 2310.6 m  (numerisch: 2310.5 m)
```

```python
# --- Teilaufgabe 5: Vergleich linear vs. quadratisch ---
# k_l so gewählt, dass beide Modelle dieselbe Endgeschwindigkeit haben
k_l = g / v_end
tau = v_end / g   # = 1/k_l
print(f"tau = v_end/g = {tau:.4f} s")

def f_fall_linear(t, y):
    # Freier Fall mit linearem Luftwiderstand: dv/dt = g - k_l*v
    return [g - k_l * y[0]]

sol_lin = solve_ivp(f_fall_linear, (0, 60), [0.0], t_eval=t_eval)

idx_tau = np.searchsorted(t_eval, tau)
v_quad_tau = sol.y[0, idx_tau]
v_lin_tau  = sol_lin.y[0, idx_tau]

print(f"Bei t = tau = {tau:.2f} s:")
print(f"  linear:    v(tau)/v_end = {v_lin_tau/v_end:.4f}  "
      f"(Theorie: 1-1/e = {1-1/np.e:.4f})")
print(f"  quadratic: v(tau)/v_end = {v_quad_tau/v_end:.4f}  "
      f"(Theorie: tanh(1) = {np.tanh(1):.4f})")

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(t_eval, sol_lin.y[0], color='#E87846', linewidth=2, linestyle='--',
        label=f'Linear: $\\dot{{v}} = g - k_l v$')
ax.plot(t_eval, sol.y[0],     color='#005A94', linewidth=2,
        label=f'Quadratisch: $\\dot{{v}} = g - k_q v^2$')
ax.axhline(v_end, color='#484949', linewidth=1.0, linestyle=':',
           label=f'$v_\\infty = {v_end:.1f}$ m/s (gleich für beide)')
ax.axvline(tau,   color='#E60000', linewidth=1.0, linestyle=':',
           label=f'$\\tau = {tau:.2f}$ s')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Geschwindigkeit in m/s')
ax.set_title('Vergleich: linearer vs. quadratischer Luftwiderstand')
ax.legend(fontsize=9)
ax.grid(True)
plt.tight_layout()
plt.show()
```

Erwartete Ausgabe:
```
tau = v_end/g = 4.1218 s
Bei t = tau = 4.12 s:
  linear:    v(tau)/v_end = 0.6390  (Theorie: 1-1/e = 0.6321)
  quadratic: v(tau)/v_end = 0.7696  (Theorie: tanh(1) = 0.7616)
```

Das quadratische Modell erreicht $v_\infty$ schneller, weil die
Bremskraft $F = k_q v^2$ nahe der Endgeschwindigkeit proportional zu
$v_\infty + v \approx 2v_\infty$ mal $(v_\infty - v)$ ist — also
doppelt so stark rückstellend wie beim linearen Modell
($F = k_l v \propto 1 \cdot (v_\infty - v)$). Bei $t = \tau$ hat das
quadratische Modell bereits 76.9 % statt 63.2 % von $v_\infty$ erreicht;
bei $t = 2\tau$ sind es 96.5 % statt 86.5 %. In der Praxis ist das
quadratische Modell für schnelle Körper in turbulenter Umströmung
(Flugzeuge, Fallschirmspringer) zutreffender, das lineare für langsame
Körper in viskosen Medien (Mikropartikel, Sedimentkörper).
````
