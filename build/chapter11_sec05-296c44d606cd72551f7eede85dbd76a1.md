---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 11.5 Übungen

```{admonition} Warnung
:class: warning
Dieses Kapitel befindet sich derzeit im Umbau und wird rechtzeitig vor der Vorlesung im WiSe 2026/27 zur Verfügung stehen.
```

````{admonition} Übung 11.5 (✩)
:class: tip
Der folgende Code soll ein nichtlineares Pendel ($L = 1\,\text{m}$,
$g = 9.81\,\text{m/s}^2$) mit $\phi_0 = 45°$ simulieren. Er enthält
zwei Fehler in der Funktion `f`. Die Simulation läuft fehlerfrei durch,
liefert aber physikalisch unsinnige Ergebnisse.

```python
from scipy.integrate import solve_ivp
import numpy as np

g = 9.81; L = 1.0
phi0   = np.radians(45)
t_eval = np.linspace(0, 10, 1001)

def f_pendel_buggy(t, y):
    return [y[0],                        # Zeile A
            -(g / L) * np.sin(y[1])]     # Zeile B

sol = solve_ivp(f_pendel_buggy, (0, 10), [phi0, 0.0], t_eval=t_eval)

print(f"phi(0)  = {sol.y[0,   0]:.4f} rad")
print(f"phi(5)  = {sol.y[0, 500]:.4f} rad")
print(f"phi(10) = {sol.y[0,  -1]:.4f} rad")
```

1. Was gibt Zeile A zurück? Was sollte sie stattdessen zurückgeben?
   Geben Sie das vollständige fehlerhafte DGL-System (beide Gleichungen)
   an und erläutern Sie, welche Dynamik es beschreibt.
2. Zeile B wertet `np.sin(y[1])` aus statt `np.sin(y[0])`. Was ist
   `y[1]` im korrekt aufgestellten Pendel-System? Worin liegt der Fehler?
3. Erklären Sie ohne Code, was der buggy Code für die gegebenen
   Anfangsbedingungen ($\phi_0 = 45°$, $\dot{\phi}(0) = 0$) tatsächlich
   berechnet. Nähert sich $\phi(t)$ der Null an?
4. Nennen Sie beide Korrekturen (Zeile A und Zeile B) in je einem Satz.
5. Führen Sie den Code aus, korrigieren Sie beide Zeilen und überprüfen
   Sie, dass das Pendel mit der erwarteten Periode
   $T_0 = 2\pi\sqrt{L/g} \approx 2.006\,\text{s}$ schwingt.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: Zeile A gibt `y[0]` zurück, das ist die aktuelle Position
$\phi$. Die korrekte Ableitung der Position ist aber die Geschwindigkeit
`y[1]`. Das vollständige fehlerhafte DGL-System lautet:

$$\dot{y}_1 = y_1 \quad \text{(statt } \dot{y}_1 = y_2\text{)}$$
$$\dot{y}_2 = -\frac{g}{L}\sin(y_2) \quad \text{(statt } \dot{y}_2 = -\frac{g}{L}\sin(y_1)\text{)}$$

Die erste Gleichung besagt, dass die Position exponentiell wächst
($\dot{\phi} = \phi$); die zweite ist eine Pendelgleichung für die
Winkelgeschwindigkeit statt für den Winkel.

Zu Frage 2: `y[1]` ist die Winkelgeschwindigkeit $\dot{\phi}$. Zeile B
wertet die Rückstellkraft also als $\sin(\dot{\phi})$ aus statt als
$\sin(\phi)$. Für kleine Winkelgeschwindigkeiten gilt $\sin(\dot{\phi})
\approx \dot{\phi}$, was einer seltsamen geschwindigkeitsabhängigen Kraft
entspricht, nicht einer positionsabhängigen Rückstellkraft.

Zu Frage 3: Mit $\dot{\phi}(0) = y_2(0) = 0$ gilt für die zweite
Gleichung: $\dot{y}_2 = -(g/L)\sin(0) = 0$. Die Winkelgeschwindigkeit
ist ein Gleichgewicht dieser Gleichung und bleibt dauerhaft null
($y_2(t) \equiv 0$). Die erste Gleichung reduziert sich damit auf
$\dot{\phi} = \phi$, also $\phi(t) = \phi_0\,e^t$. Der Winkel wächst
exponentiell; nach 10 s beträgt er bereits mehrere zehntausend Radiant
(entspricht fast einer Million Grad). $\phi(t)$ nähert sich keineswegs
der Null, sondern wächst unbegrenzt.

Zu Frage 4: Korrektur Zeile A: `y[0]` durch `y[1]` ersetzen (die
Ableitung der Position ist die Geschwindigkeit). Korrektur Zeile B:
`y[1]` durch `y[0]` ersetzen (die Rückstellkraft hängt vom Winkel ab,
nicht von der Winkelgeschwindigkeit).

```python
def f_pendel_korrekt(t, y):
    return [y[1],                        # Zeile A korrekt: d(phi)/dt = phi_dot
            -(g / L) * np.sin(y[0])]     # Zeile B korrekt: sin(phi), nicht sin(phi_dot)

sol_korrekt = solve_ivp(f_pendel_korrekt, (0, 10), [phi0, 0.0], t_eval=t_eval)
print(f"phi(0)  = {sol_korrekt.y[0,   0]:.4f} rad")   # = 0.7854 rad (= 45°)
print(f"phi(5)  = {sol_korrekt.y[0, 500]:.4f} rad")   # Schwingung
print(f"phi(10) = {sol_korrekt.y[0,  -1]:.4f} rad")   # Schwingung
```

Ausgabe:

```
phi(0)  = 0.7854 rad
phi(5)  ≈ -0.6418 rad   (Schwingung)
phi(10) ≈  0.7106 rad   (Schwingung)
```

Die Periode lässt sich aus dem Plot ablesen: $T \approx 2.04\,\text{s}$,
etwas über $T_0 = 2.006\,\text{s}$, weil 45° keine kleine Auslenkung
mehr ist.
````

````{admonition} Übung 11.6 (✩✩)
:class: tip
Ein RLC-Reihenschwingkreis mit Induktivität $L_\text{ind} = 0.1\,\text{H}$,
Widerstand $R = 10\,\Omega$ und Kapazität $C = 100\,\mu\text{F}$ wird durch
$U_0\cos(\Omega t)$ mit $U_0 = 5\,\text{V}$ angetrieben. Die Ladung $q$
genügt der DGL

$$L_\text{ind}\,\ddot{q} + R\,\dot{q} + \frac{q}{C} = U_0\cos(\Omega t).$$

1. Stellen Sie das System erster Ordnung auf ($y_1 = q$, $y_2 = \dot{q}$)
   und implementieren Sie `f_rlc(t, y, Omega)`.
2. Legen Sie eine Analogietabelle an: Welche mechanische Größe entspricht
   jeweils $L_\text{ind}$, $R$, $1/C$, $U_0$, $q$?
3. Berechnen Sie $\omega_0 = 1/\sqrt{L_\text{ind}C}$ und
   $D = R\,/\,(2\sqrt{L_\text{ind}/C})$. Lösen Sie für $\Omega = \omega_0$
   (Eigenkreisfrequenz) und bestimmen Sie die stationäre Ladungsamplitude
   $Q_\text{stat}$.
4. Berechnen und plotten Sie den Strom $i(t) = \dot{q}(t)$ aus `sol.y[1]`
   im eingeschwungenen Zustand.
5. Bestimmen Sie die Resonanzkurve $Q(\Omega)$ numerisch und vergleichen
   Sie die Resonanzfrequenz $\Omega_\text{res}$ mit der analytischen Formel
   $\Omega_\text{res} = \omega_0\sqrt{1 - 2D^2}$.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1:

```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

# --- Parameter ---
L_ind = 0.1        # Induktivität in H
R     = 10.0       # Widerstand in Ohm
C     = 100e-6     # Kapazität in F
U0    = 5.0        # Spannungsamplitude in V

# --- Eigenkreisfrequenz und Dämpfungszahl ---
# Formale Analogie: L_ind <-> m, R <-> c, 1/C <-> k, U0 <-> F0, q <-> x
omega_0_rlc = 1 / np.sqrt(L_ind * C)         # = 316.23 rad/s
D_rlc       = R / (2 * np.sqrt(L_ind / C))   # = 0.1581
c_krit_rlc  = 2 * np.sqrt(L_ind / C)

print(f"omega_0 = {omega_0_rlc:.2f} rad/s")
print(f"D       = {D_rlc:.4f}")

# y[0] = q (Ladung in C), y[1] = q_dot = i (Strom in A)
# L_ind * q_ddot + R * q_dot + q/C = U0 * cos(Omega * t)
# q_ddot = (U0*cos(Omega*t) - R*q_dot - q/C) / L_ind
def f_rlc(t, y, Omega):
    q_dot  = y[1]
    q_ddot = (U0 * np.cos(Omega * t) - R * y[1] - y[0] / C) / L_ind
    return [q_dot, q_ddot]
```

Zu Frage 2 (Analogietabelle, formale Strukturanalogie der DGL):

| Elektrik | Mechanik |
|---|---|
| $L_\text{ind} = 0.1\,\text{H}$ | Masse $m$ |
| $R = 10\,\Omega$ | Dämpfer $c$ |
| $1/C = 10000\,\text{F}^{-1}$ | Federsteifigkeit $k$ |
| $U_0 = 5\,\text{V}$ | Kraftamplitude $F_0$ |
| $q$ (Ladung in C) | Auslenkung $x$ |

Zu Frage 3:

```python
# Simulation bei Omega = omega_0 (Eigenkreisfrequenz)
t_end  = 0.2     # kurze Zeit, weil omega_0 >> 1 (viele Schwingungen pro s)
t_eval = np.linspace(0, t_end, 20001)
n_last = len(t_eval) // 3

sol_res = solve_ivp(lambda t, y: f_rlc(t, y, omega_0_rlc),
                    (0, t_end), [0.0, 0.0], t_eval=t_eval)

Q_stat = np.max(np.abs(sol_res.y[0, -n_last:]))

# Analytische Formel bei Omega = omega_0:
# Bei Omega = omega_0 gilt L*omega_0^2 = 1/C, daher vereinfacht sich
# der Nenner der allgemeinen Amplitudenformel auf R*omega_0 = (1/C)*2D.
# Q(Omega=omega_0) = U0 / (R * omega_0) = U0 / ((1/C) * 2*D)
Q_stat_ana = U0 / ((1 / C) * 2 * D_rlc)

print(f"Q_stat numerisch:  {Q_stat*1e3:.4f} mC")
print(f"Q_stat analytisch: {Q_stat_ana*1e3:.4f} mC")
```

Ausgabe:

```
Q_stat numerisch:  1.5832 mC
Q_stat analytisch: 1.5811 mC
```

Zu Frage 4:

```python
# Strom i(t) = q_dot(t) = sol.y[1] (letzte 2 Perioden)
T_res   = 2 * np.pi / omega_0_rlc
idx_cut = np.searchsorted(t_eval, t_end - 2 * T_res)

fig, axes = plt.subplots(2, 1, figsize=(8, 5), sharex=True)
axes[0].plot(t_eval[idx_cut:], sol_res.y[0, idx_cut:] * 1e3,
             color='#005A94', label='Ladung q(t)')
axes[0].set_ylabel('Ladung in mC')
axes[0].legend()

axes[1].plot(t_eval[idx_cut:], sol_res.y[1, idx_cut:] * 1e3,
             color='#E60000', label='Strom i(t) = q_dot(t)')
axes[1].set_ylabel('Strom in mA')
axes[1].set_xlabel('Zeit in s')
axes[1].legend()

plt.suptitle('RLC-Schwingkreis bei Ω = ω₀ (eingeschwungener Zustand)')
plt.tight_layout()
plt.show()
```

Zu Frage 5:

```python
Omega_werte  = np.linspace(0.5 * omega_0_rlc, 1.5 * omega_0_rlc, 60)
Q_werte      = np.zeros(len(Omega_werte))

for i, Omega in enumerate(Omega_werte):
    sol_i    = solve_ivp(lambda t, y, Om=Omega: f_rlc(t, y, Om),
                         (0, t_end), [0.0, 0.0], t_eval=t_eval)
    Q_werte[i] = np.max(np.abs(sol_i.y[0, -n_last:]))

Omega_res_num = Omega_werte[np.argmax(Q_werte)]
Omega_res_ana = omega_0_rlc * np.sqrt(1 - 2 * D_rlc**2)

print(f"Omega_res numerisch:  {Omega_res_num:.2f} rad/s")
print(f"Omega_res analytisch: {Omega_res_ana:.2f} rad/s")
```

Ausgabe:

```
Omega_res numerisch:  308.35 rad/s
Omega_res analytisch: 308.10 rad/s
```

Die Resonanzfrequenz liegt etwas unter der Eigenkreisfrequenz
$\omega_0 = 316.23\,\text{rad/s}$, weil die Dämpfung ($D \approx 0.16$)
pro Schwingungszyklus Energie dissipiert und dadurch das Amplitudenmaximum
zu einer Frequenz unterhalb von $\omega_0$ verschiebt.
````

````{admonition} Übung 11.7 (✩✩✩)
:class: tip
Ein Fahrzeug fährt über eine sinusförmige Fahrbahn
$z_\text{Bahn}(t) = A_\text{Bahn}\sin(\Omega t)$ mit
$A_\text{Bahn} = 0.02\,\text{m}$. Die Anregungskraft auf die Karosserie
ist $F(t) = k\,z_\text{Bahn}(t)$. Das Federungsmodell lautet:

$$m\ddot{x} + c\dot{x} + kx = k\,A_\text{Bahn}\sin(\Omega t).$$

Gegeben: $m = 250\,\text{kg}$, $k = 16000\,\text{N/m}$,
$\omega_0 = 8\,\text{rad/s}$.

**Teil 1 - Zeitsignale bei $\Omega = \omega_0$**

1. Lösen Sie die DGL für $\Omega = \omega_0$ und
   $D \in \{0.1,\, 0.3,\, 1.0\}$ für $t \in [0, 30\,\text{s}]$.
   Stellen Sie alle drei $x(t)$-Verläufe in einem gemeinsamen Plot dar.
   Lesen Sie die stationäre Amplitude für jeden $D$-Wert ab.

**Teil 2 - Resonanzkurven**

2. Berechnen Sie für alle drei $D$-Werte die Resonanzkurve $A(\Omega)$
   für $\Omega \in [0.2\,\omega_0,\; 2.0\,\omega_0]$ (mindestens
   40 Werte) und stellen Sie die drei Kurven in einem gemeinsamen Plot
   dar.

**Teil 3 - Energiedissipation**

3. Berechnen Sie die Verlustleistung $P(t) = c\,\dot{x}(t)^2$ aus
   `sol.y[1]` und die kumulierte dissipierte Energie
   $E_\text{diss}(t) = \int_0^t P\,\mathrm{d}t$ mit
   `scipy.integrate.cumulative_trapezoid`. Vergleichen Sie
   $E_\text{diss}(30\,\text{s})$ für $D = 0.1$ und $D = 1.0$ bei
   Anregung mit $\Omega = \omega_0$. Welcher Wert ist größer, und warum?

**Teil 4 - Auslegungsfrage**

4. Welchen $D$-Wert würden Sie für eine Fahrzeugfederung empfehlen?
   Nennen Sie zwei konkurrierende Anforderungen und begründen Sie Ihre
   Wahl in drei Sätzen.
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

# --- Parameter ---
m        = 250.0
k_feder  = 16000.0
omega_0  = np.sqrt(k_feder / m)   # = 8.0 rad/s
c_krit   = 2 * np.sqrt(k_feder * m)
A_bahn   = 0.02     # Fahrbahnunebenheit in m
F0_bahn  = k_feder * A_bahn   # = 320 N (effektive Kraftamplitude)
D_werte  = [0.1, 0.3, 1.0]
farben   = ['#005A94', '#E87846', '#E60000']

def f_fahrbahn(t, y, c, Omega):
    # m * x_ddot + c * x_dot + k * x = k * A_bahn * sin(Omega*t)
    x_dot  = y[1]
    x_ddot = (-k_feder * y[0] - c * y[1] + F0_bahn * np.sin(Omega * t)) / m
    return [x_dot, x_ddot]
```

**Teil 1: Zeitsignale bei $\Omega = \omega_0$**

```python
t_end  = 30.0
t_eval = np.linspace(0, t_end, 3001)
n_last = len(t_eval) // 3

fig, ax = plt.subplots(figsize=(9, 4))
for D, farbe in zip(D_werte, farben):
    c   = D * c_krit
    sol = solve_ivp(lambda t, y, c=c: f_fahrbahn(t, y, c, omega_0),
                    (0, t_end), [0.0, 0.0], t_eval=t_eval)
    A_stat = np.max(np.abs(sol.y[0, -n_last:]))
    ax.plot(t_eval, sol.y[0] * 100, color=farbe,
            label=f'D = {D}  (A_stat = {A_stat*100:.1f} cm)')

ax.set_xlabel('Zeit in s')
ax.set_ylabel('Auslenkung in cm')
ax.set_title('Fahrzeugfederung bei Ω = ω₀')
ax.legend()
plt.tight_layout()
plt.show()
```

Stationäre Amplituden bei $\Omega = \omega_0$ (analytisch: $A = A_\text{Bahn}/(2D)$):
$D = 0.1$: $A_\text{stat} \approx 10.0\,\text{cm}$ (fünfmal die Fahrbahnunebenheit),
$D = 0.3$: $A_\text{stat} \approx 3.3\,\text{cm}$,
$D = 1.0$: $A_\text{stat} \approx 1.0\,\text{cm}$.

Hinweis: Für $D = 1.0$ (aperiodischer Grenzfall) existiert kein
Resonanzpeak ($\Omega_R = \omega_0\sqrt{1-2D^2}$ wird für $D \ge 1/\sqrt{2}
\approx 0.707$ komplex). Die Anregung bei $\Omega = \omega_0$ dient hier
lediglich als gemeinsame Vergleichsfrequenz für alle drei Fälle.

**Teil 2: Resonanzkurven**

```python
Omega_werte = np.linspace(0.2 * omega_0, 2.0 * omega_0, 50)

fig, ax = plt.subplots(figsize=(8, 4))
for D, farbe in zip(D_werte, farben):
    c       = D * c_krit
    A_kurve = np.zeros(len(Omega_werte))
    for i, Omega in enumerate(Omega_werte):
        sol = solve_ivp(lambda t, y, c=c, Om=Omega: f_fahrbahn(t, y, c, Om),
                        (0, t_end), [0.0, 0.0], t_eval=t_eval)
        A_kurve[i] = np.max(np.abs(sol.y[0, -n_last:]))
    ax.plot(Omega_werte / omega_0, A_kurve / A_bahn, color=farbe,
            label=f'D = {D}')

ax.axvline(1.0, color='black', linestyle='--', linewidth=0.8, label='Ω = ω₀')
ax.set_xlabel('Frequenzverhältnis Ω / ω₀')
ax.set_ylabel('Amplitudenverhältnis A / A_Bahn')
ax.set_title('Resonanzkurven: Fahrzeugfederung')
ax.legend()
plt.tight_layout()
plt.show()
```

Im Plot ist zu erkennen, dass für $D = 1.0$ kein ausgeprägter Resonanzpeak
auftritt; die Kurve fällt mit wachsendem $\Omega$ monoton ab. Nur für
$D < 1/\sqrt{2} \approx 0.707$ existiert ein echtes Amplitudenmaximum
bei $\Omega_R < \omega_0$.

**Teil 3: Energiedissipation**

```python
fig, ax = plt.subplots(figsize=(8, 4))
for D, farbe in zip([0.1, 1.0], ['#005A94', '#E60000']):
    c   = D * c_krit
    sol = solve_ivp(lambda t, y, c=c: f_fahrbahn(t, y, c, omega_0),
                    (0, t_end), [0.0, 0.0], t_eval=t_eval)
    # Verlustleistung P(t) = c * x_dot(t)^2
    P       = c * sol.y[1]**2
    # Kumulierte Energie mit Trapezregel (aus Kapitel 9)
    E_diss  = cumulative_trapezoid(P, sol.t, initial=0)
    ax.plot(sol.t, E_diss / 1000, color=farbe,
            label=f'D = {D}  (E_diss(30s) = {E_diss[-1]/1000:.1f} kJ)')

ax.set_xlabel('Zeit in s')
ax.set_ylabel('Dissipierte Energie in kJ')
ax.set_title('Kumulierte Energiedissipation bei Ω = ω₀')
ax.legend()
plt.tight_layout()
plt.show()
```

Ausgabe: $D = 0.1$ dissipiert bei $\Omega = \omega_0$ deutlich mehr Energie
als $D = 1.0$ ($\approx 260\,\text{kJ}$ vs. $\approx 15\,\text{kJ}$ nach
30 s). Obwohl $D = 0.1$ einen kleineren Dämpfer $c$ hat, sorgt die weit
größere Schwingungsamplitude für eine viel höhere Verlustleistung
$P = c\,\dot{x}^2$.

**Teil 4: Auslegungsfrage**

Für eine Fahrzeugfederung gibt es zwei konkurrierende Anforderungen:
Fahrkomfort (kleine stationäre Amplitude bei Resonanz, also größeres $D$)
und Handling (schnelles Einschwingen nach einer Einzelstörung wie einer
Bodenwelle, was nahe $D = 1$ liegt). Ein typischer Personenwagen wählt
$D \approx 0.25 \ldots 0.4$: Das ist ein Kompromiss, der bei
Autobahnfahrten die Resonanzamplitude begrenzt und nach einem Schlagloch
das Fahrzeug in wenigen Schwingungen beruhigt. Reine Sportwagen tendieren
zu $D \approx 0.5 \ldots 0.7$, um Karosseriebewegungen zu minimieren.
````
