---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 11.4 Übungen zur gedämpften Schwingung

````{admonition} Übung 11.3 (✩)
:class: tip
Der folgende Code simuliert den gedämpften Federschwinger
($m = 250\,\text{kg}$, $k = 16000\,\text{N/m}$) für vier Dämpfungszahlen
und gibt die Zeit aus, nach der die Auslenkung **erstmals** unter
$10^{-3}\,\text{m}$ fällt. Wichtig: Diese Definition entspricht dem ersten
Unterschreiten der Schwelle, nicht dem dauerhaften Einpendeln unterhalb
der Schwelle. Im Schwingfall kann das System danach wieder über die Schwelle
zurückpendeln.

```python
from scipy.integrate import solve_ivp
import numpy as np

m = 250.0; k = 16000.0; x0 = 0.05; v0 = 0.0
omega_0 = np.sqrt(k / m)
c_krit  = 2 * np.sqrt(k * m)
t_eval  = np.linspace(0, 10, 2001)

def f_gedaempft(t, y, c):
    return [y[1], (-k * y[0] - c * y[1]) / m]

for D in [0.1, 0.3, 1.0, 2.0]:
    c   = D * c_krit
    sol = solve_ivp(lambda t, y, c=c: f_gedaempft(t, y, c),
                    (0, 10), [x0, v0], t_eval=t_eval)
    unter_schwelle = np.where(np.abs(sol.y[0]) < 1e-3)[0]
    if len(unter_schwelle) > 0:
        t_ein = t_eval[unter_schwelle[0]]
    else:
        t_ein = float('inf')
    print(f"D = {D:.1f}:  T_einschwing = {t_ein:.3f} s")
```

1. Ohne Code: Für welche $D$-Werte schwingt das System? Ordnen Sie die vier
   $D$-Werte den Fällen Schwingfall, aperiodischer Grenzfall und Kriechfall
   zu.
2. Für $D = 0.1$ (Schwingfall): Die Einhüllende klingt im Schwingfall wie
   $x_0\,e^{-D\omega_0 t}$ ab. Hat das System in 10 s die Schwelle
   $10^{-3}\,\text{m}$ unterschritten? Schätzen Sie ab, ohne den Code
   auszuführen.
3. Für welchen der vier $D$-Werte gibt der Code die kleinste Einschwingzeit
   (erstes Unterschreiten) aus? Und für welchen $D$-Wert ist die theoretisch
   schnellste Rückkehr ohne Überschwingen zu erwarten? Sind das dieselben?
   Begründen Sie in zwei Sätzen.
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: $D = 0.1$ und $D = 0.3$ gehören zum Schwingfall ($D < 1$,
schwingendes Verhalten). $D = 1.0$ ist der aperiodische Grenzfall.
$D = 2.0$ ist der Kriechfall.

Zu Frage 2: Einhüllende nach 10 s: $x_0\,e^{-D\omega_0\cdot 10} =
0.05\,e^{-0.1\cdot 8\cdot 10} = 0.05\,e^{-8} \approx
0.05\cdot 3.35\cdot 10^{-4} \approx 1.7\cdot 10^{-5}\,\text{m}$.
Das liegt deutlich unter der Schwelle. Der Code fragt aber nach dem
ersten Zeitpunkt, an dem $|x| < 10^{-3}$ gilt. Im Schwingfall tritt das
bereits nahe dem ersten Nulldurchgang auf, danach schwenkt $|x|$ wieder
über die Schwelle zurück.

Zu Frage 3: Nach der Code-Definition (erstes Unterschreiten) gibt $D = 0.1$
die kleinste Zeit aus, weil der Schwingfall einen frühen Nulldurchgang
erzeugt, bei dem $|x|$ kurz unter die Schwelle taucht. Theoretisch
(dauerhaft unter der Schwelle, ohne Überschwingen) liefert $D = 1.0$
(aperiodischer Grenzfall) die schnellste Konvergenz; das sind zwei
verschiedene Konzepte von Einschwingzeit.

Zu Frage 4: Erwartete Ausgabe:

```
D = 0.1:  T_einschwing = 0.210 s   (Schwingfall: erstes Unterschreiten nahe Nulldurchgang,
                                     danach kehrt |x| über die Schwelle zurück)
D = 0.3:  T_einschwing = 0.245 s   (Schwingfall: analoges Verhalten)
D = 1.0:  T_einschwing = 0.730 s   (aperiodischer Grenzfall: monotoner Abfall,
                                     dauerhaft unter der Schwelle)
D = 2.0:  T_einschwing = 1.860 s   (Kriechfall: langsam kriechend,
                                     dauerhaft unter der Schwelle)
```

Für $D = 1.0$ und $D = 2.0$ bleibt $|x|$ nach dem Unterschreiten dauerhaft
unter der Schwelle. Für $D = 0.1$ und $D = 0.3$ ist das nicht der Fall.
````

````{admonition} Übung 11.4 (✩✩)
:class: tip
Gegeben: Erzwungener Schwinger $m\ddot{x} + c\dot{x} + kx = F_0\sin(\Omega
t)$ mit $m = 250\,\text{kg}$, $k = 16000\,\text{N/m}$, $D = 0.2$,
$F_0 = 500\,\text{N}$, $x(0) = 0$, $\dot{x}(0) = 0$.

1. Schreiben Sie `f_erzwungen(t, y, Omega)` und implementieren Sie eine
   Schleife über $\Omega \in [0.2\,\omega_0,\; 2.0\,\omega_0]$
   (mindestens 40 gleichmäßig verteilte Werte). Berechnen Sie für jeden
   $\Omega$-Wert die stationäre Amplitude als `np.max(np.abs(sol.y[0,
   -n:]))`, wobei `n` das letzte Drittel der Zeitpunkte abdeckt.
2. Plotten Sie die Resonanzkurve $A(\Omega)$ und markieren Sie $\omega_0$
   als vertikale gestrichelte Linie.
3. Bestimmen Sie die Resonanzfrequenz der erzwungenen Schwingung numerisch
   als `Omega_werte[np.argmax(A_werte)]` und vergleichen Sie mit der
   analytischen Formel $\Omega_\text{res} = \omega_0\sqrt{1 - 2D^2}$.
   Hinweis: Diese Resonanzfrequenz unterscheidet sich von der
   Eigenkreisfrequenz $\omega_0$ des Systems.
4. Warum liegt $\Omega_\text{res}$ leicht unterhalb von $\omega_0$?
   Erklären Sie in einem Satz.
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

# --- Parameter ---
m       = 250.0
k_feder = 16000.0
omega_0 = np.sqrt(k_feder / m)
c_krit  = 2 * np.sqrt(k_feder * m)
D       = 0.2
c       = D * c_krit      # = 800 N*s/m
F0      = 500.0

def f_erzwungen(t, y, Omega):
    x_dot  = y[1]
    x_ddot = (-k_feder * y[0] - c * y[1] + F0 * np.sin(Omega * t)) / m
    return [x_dot, x_ddot]

# --- Schleife über Omega-Werte ---
Omega_werte = np.linspace(0.2 * omega_0, 2.0 * omega_0, 60)
A_werte     = np.zeros(len(Omega_werte))
t_end       = 30.0
t_eval      = np.linspace(0, t_end, 3001)
n_last      = len(t_eval) // 3

for i, Omega in enumerate(Omega_werte):
    sol = solve_ivp(lambda t, y, Om=Omega: f_erzwungen(t, y, Om),
                    (0, t_end), [0.0, 0.0], t_eval=t_eval)
    A_werte[i] = np.max(np.abs(sol.y[0, -n_last:]))

# --- Plot ---
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(Omega_werte, A_werte * 100, color='#005A94', linewidth=2,
        label='A(Ω) numerisch')
ax.axvline(omega_0, color='#E60000', linestyle='--',
           label=f'ω₀ = {omega_0:.1f} rad/s')

Omega_res_num = Omega_werte[np.argmax(A_werte)]
ax.axvline(Omega_res_num, color='#E87846', linestyle=':',
           label=f'Ω_res numerisch = {Omega_res_num:.2f} rad/s')

ax.set_xlabel('Erregerfrequenz Ω in rad/s')
ax.set_ylabel('Stationäre Amplitude in cm')
ax.set_title(f'Resonanzkurve (D = {D})')
ax.legend()
plt.tight_layout()
plt.show()

# --- Vergleich analytisch / numerisch ---
# Omega_res ist die Resonanzfrequenz der erzwungenen Schwingung;
# sie liegt unterhalb der Eigenkreisfrequenz omega_0.
Omega_res_ana = omega_0 * np.sqrt(1 - 2 * D**2)
A_max_ana     = F0 / (k_feder * 2 * D * np.sqrt(1 - D**2))
print(f"Omega_res numerisch:  {Omega_res_num:.4f} rad/s")
print(f"Omega_res analytisch: {Omega_res_ana:.4f} rad/s")
print(f"(Eigenkreisfrequenz omega_0 = {omega_0:.4f} rad/s zum Vergleich)")
print(f"A_max numerisch:      {np.max(A_werte)*100:.2f} cm")
print(f"A_max analytisch:     {A_max_ana*100:.2f} cm")
```

Erwartete Ausgabe:

```
Omega_res numerisch:  7.6700 rad/s
Omega_res analytisch: 7.6681 rad/s
(Eigenkreisfrequenz omega_0 = 8.0000 rad/s zum Vergleich)
A_max numerisch:      7.98 cm
A_max analytisch:     7.98 cm
```

Zu Frage 4: Die Dämpfung entzieht dem System pro Schwingungszyklus Energie;
dadurch verschiebt sich das Amplitudenmaximum der erzwungenen Schwingung zu
einer Frequenz etwas unterhalb von $\omega_0$. Für $D \to 0$ fällt
$\Omega_\text{res} = \omega_0\sqrt{1-2D^2}$ mit $\omega_0$ zusammen; je
größer $D$, desto weiter verschiebt sich $\Omega_\text{res}$ nach unten.
````
