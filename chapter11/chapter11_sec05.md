---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 11.5 Übungen

````{admonition} Übung 11.5 (✩)
:class: tip
<!-- TODO: Aufgabe 11.5 (~30 min)
     Thema: Vorgegebener Code für das nichtlineare Pendel mit vertauschten Indizes
     in f(t, y). Kein Absturz — fehlerhaftes Ergebnis (konvergiert scheinbar gegen 0).

     SZENARIO:
     ```python
     from scipy.integrate import solve_ivp
     import numpy as np

     g = 9.81; L = 1.0
     phi0 = np.radians(45)   # 45° Anfangsauslenkung
     t_eval = np.linspace(0, 10, 1001)

     def f_pendel_buggy(t, y):
         return [y[0],                        # Zeile A
                 -(g/L) * np.sin(y[1])]       # Zeile B

     sol = solve_ivp(f_pendel_buggy, (0, 10), [phi0, 0.0], t_eval=t_eval)

     print(f"phi(0)  = {sol.y[0, 0]:.4f} rad")
     print(f"phi(5)  = {sol.y[0, 500]:.4f} rad")
     print(f"phi(10) = {sol.y[0, -1]:.4f} rad")
     ```

     FRAGEN:
     1. Was gibt Zeile A zurück? Was sollte sie zurückgeben?
        Welche DGL löst der Code tatsächlich?
     2. Zeile B wertet sin(y[1]) aus statt sin(y[0]). Was ist y[1] in
        einem korrekt aufgestellten System? Was ist also der Fehler?
     3. Warum konvergiert phi(t) scheinbar gegen 0, obwohl das Pendel
        eigentlich schwingen sollte? Erklären Sie ohne Code.
     4. Nennen Sie beide Korrekturen (Zeile A und Zeile B) in einem Satz.
     5. Führen Sie den Code aus, korrigieren Sie beide Zeilen und
        überprüfen Sie, dass das Pendel mit der erwarteten Periode
        T_0 = 2*pi*sqrt(L/g) ≈ 2.006 s schwingt.

     ANALYSE:
     - Buggy: y[0]-Gleichung: d(phi)/dt = phi (statt phi_dot)
              y[1]-Gleichung: d(phi_dot)/dt = -(g/L)*sin(phi_dot) (statt sin(phi))
     - Für kleine phi_dot: sin(phi_dot) ≈ phi_dot → lineare Dämpfung,
       phi wächst exponentiell, dann oscilliert phi_dot → phi wird groß → nonlinear
     - Tatsächliches Verhalten: numerisch instabil oder chaotisch, nicht physikalisch
     - Korrektur: Zeile A: return [y[1], ...], Zeile B: np.sin(y[0]) -->

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
   Welche DGL löst der Code tatsächlich?
2. Zeile B wertet `np.sin(y[1])` aus statt `np.sin(y[0])`. Was ist
   `y[1]` im korrekt aufgestellten Pendel-System? Worin liegt der Fehler?
3. Warum nähert sich `phi(t)` scheinbar 0 an, obwohl das Pendel
   eigentlich schwingen sollte? Erklären Sie ohne Code.
4. Nennen Sie beide Korrekturen (Zeile A und Zeile B) in einem Satz.
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
<!-- TODO: Lösung zu Aufgabe 11.5.
     Zeile A: y[0] ist phi (Position), nicht phi_dot. Korrekt: return [y[1], ...]
     Zeile B: y[1] ist phi_dot, nicht phi. Korrekt: np.sin(y[0])
     Korrektur:
       def f_pendel_korrekt(t, y):
           return [y[1], -(g/L)*np.sin(y[0])]
     Erwartete Ausgabe korrekt:
       phi(0)  = 0.7854 rad  (= 45° in Bogenmaß)
       phi(5)  ≈ +/- kleiner Wert (Schwingung)
       phi(10) ≈ +/- kleiner Wert (Schwingung)
     Periode aus Plot ablesen: ≈ 2.04 s (etwas über T_0 wegen Nichtlinearität). -->
````

````{admonition} Übung 11.6 (✩✩)
:class: tip
<!-- TODO: Aufgabe 11.6 (~50 min)
     Thema: RLC-Reihenschwingkreis als mechanische Analogie.

     SZENARIO:
     Reihenschwingkreis mit Induktivität L_ind, Widerstand R, Kapazität C,
     angetrieben durch U0*cos(Omega*t):
         L_ind * q_ddot + R * q_dot + q/C = U0 * cos(Omega*t)
     Analogie zum mechanischen Schwinger:
         m ↔ L_ind, c ↔ R, k ↔ 1/C, F0 ↔ U0, x ↔ q (Ladung)

     PARAMETER:
     L_ind = 0.1 H, R = 10 Ohm, C = 100e-6 F, U0 = 5 V
     omega_0_rle = 1/sqrt(L_ind*C) = 1/sqrt(0.1*100e-6) = 1/sqrt(1e-5) ≈ 316.2 rad/s
     D_rlc = R/(2*sqrt(L_ind/C)) = 10/(2*sqrt(1000)) ≈ 0.158
     Omega_res ≈ omega_0_rlc * sqrt(1-2*D^2)

     AUFGABEN:
     1. Stellen Sie das System erster Ordnung auf (y=[q, q_dot]) und
        implementieren Sie f_rlc(t, y, Omega).
     2. Legen Sie die Analogietabelle an (Kommentar oder Markdown-Zelle):
        m↔L_ind, c↔R, k↔1/C, F0*sin(Omega*t)↔U0*cos(Omega*t), x↔q.
     3. Lösen Sie für Omega=omega_0_rlc (Resonanz) und D=0.158.
        Bestimmen Sie die stationäre Ladungsamplitude Q_stat.
     4. Berechnen Sie den Strom i(t) = q_dot(t) = sol.y[1] und plotten
        Sie i(t) im eingeschwungenen Zustand.
     5. Bestimmen Sie numerisch die Resonanzkurve Q(Omega) und vergleichen
        Sie Omega_res mit der analytischen Formel.

     VORBERECHNETE WERTE:
     omega_0_rlc = 1/sqrt(0.1*100e-6) = 316.23 rad/s
     D_rlc = 10/(2*sqrt(0.1/100e-6)) = 10/(2*31.62) = 0.1581
     Omega_res = 316.23*sqrt(1-2*0.025) = 316.23*sqrt(0.95) ≈ 308.1 rad/s -->

Ein RLC-Reihenschwingkreis mit Induktivität $L = 0.1\,\text{H}$,
Widerstand $R = 10\,\Omega$ und Kapazität $C = 100\,\mu F$ wird durch
$U_0\cos(\Omega t)$ mit $U_0 = 5\,\text{V}$ angetrieben. Die Ladung $q$
genügt der DGL

$$L\,\ddot{q} + R\,\dot{q} + \frac{q}{C} = U_0\cos(\Omega t).$$

1. Stellen Sie das System erster Ordnung auf
   ($y_1 = q$, $y_2 = \dot{q}$) und implementieren Sie
   `f_rlc(t, y, Omega)`.
2. Legen Sie eine Analogietabelle an: Welche mechanische Größe entspricht
   jeweils $L$, $R$, $1/C$, $U_0$, $q$?
3. Berechnen Sie $\omega_0 = 1/\sqrt{LC}$ und $D = R\,/\,(2\sqrt{L/C})$.
   Lösen Sie für $\Omega = \omega_0$ (Resonanz) und bestimmen Sie die
   stationäre Amplitude $Q_\text{stat}$.
4. Berechnen und plotten Sie den Strom $i(t) = \dot{q}(t)$
   (aus `sol.y[1]`) im eingeschwungenen Zustand.
5. Bestimmen Sie die Resonanzkurve $Q(\Omega)$ numerisch und vergleichen
   Sie $\Omega_\text{res}$ mit der analytischen Formel
   $\Omega_\text{res} = \omega_0\sqrt{1 - 2D^2}$.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
<!-- TODO: Lösung zu Aufgabe 11.6.
     omega_0_rlc = 1/sqrt(0.1*100e-6) = 316.23 rad/s
     D_rlc = 10/(2*sqrt(0.1/100e-6)) = 10/63.25 ≈ 0.1581
     Analogietabelle:
       m ↔ L = 0.1 H
       c ↔ R = 10 Ω
       k ↔ 1/C = 10000 F^{-1}
       F0 ↔ U0 = 5 V
       x ↔ q (Ladung in C)
     Q_stat = U0 / ((1/C) * 2*D*sqrt(1-D^2))  (analytisch)
            = 5 / (10000 * 2*0.158*sqrt(1-0.025))
            ≈ 5 / (10000 * 0.309) ≈ 1.62e-3 C -->
````

````{admonition} Übung 11.7 (✩✩✩)
:class: tip
<!-- TODO: Aufgabe 11.7 (~90 min)
     Thema: Fahrzeugfederung auf sinusförmiger Fahrbahn — Parameterstudie + Energiedissipation.

     SZENARIO:
     Fahrbahn: z_bahn(t) = A_bahn * sin(Omega*t), A_bahn = 0.02 m (2 cm Unebenheit).
     Anregungskraft: F(t) = k * z_bahn(t) = k * A_bahn * sin(Omega*t).
     DGL: m*x_ddot + c*x_dot + k*x = k*A_bahn*sin(Omega*t).

     PARAMETER:
     m=250 kg, k=16000 N/m, A_bahn=0.02 m
     omega_0 = 8 rad/s, c_krit = 4000 N*s/m
     Drei D-Werte: D in [0.1, 0.3, 1.0]

     TEIL 1: Zeitsignale bei Resonanz (Omega = omega_0)
     → drei Kurven x(t) in einem Plot, stationäre Amplituden ablesen
     → Welches D führt zu welcher stationären Amplitude?

     VORBERECHNETE AMPLITUDE bei Resonanz (Omega=omega_0):
     A_stat = (A_bahn * k) / (k * 2*D*sqrt(1-D^2))
            = A_bahn / (2*D*sqrt(1-D^2))
     D=0.1: A_stat = 0.02/(2*0.1*0.995) = 0.02/0.199 ≈ 0.1005 m
     D=0.3: A_stat = 0.02/(2*0.3*0.954) = 0.02/0.572 ≈ 0.0350 m
     D=1.0: A_stat (kein Resonanzpeak für D>=1/sqrt(2)≈0.707, Amplitude kleiner)
            A_stat(Omega=omega_0) = A_bahn/(2*D) = 0.02/2 = 0.01 m

     TEIL 2: Resonanzkurven für alle drei D-Werte
     → Drei A(Omega)-Kurven in einem Plot
     → Resonanzfrequenz für D=0.1 und D=0.3 ablesen und vergleichen

     TEIL 3: Energiedissipation (Brücke zu Kap. 9)
     → Verlustleistung P(t) = c * x_dot(t)^2 = c * sol.y[1]^2
     → E_diss(t) = cumulative_trapezoid(P, sol.t) [Kap. 9!]
     → Gesamte dissipierte Energie nach 30 s für D=0.1 und D=1.0 vergleichen
     → Verständnisfrage: Warum dissipiert D=0.1 mehr Energie im Resonanzfall?

     VORBERECHNETE WERTE ENERGIE:
     Stationäre Dissipationsleistung P_stat = c * (A_stat * Omega)^2 / 2
     (Amplitude der Geschwindigkeit = A_stat * Omega)
     D=0.1: c=400, P_stat = 400*(0.1005*8)^2/2 = 400*0.648 = 259.2 W
     D=1.0: c=4000, P_stat = 4000*(0.01*8)^2/2 = 4000*0.0032 = 12.8 W
     → D=0.1 dissipiert trotz kleinerem c MEHR Energie wegen viel größerer Amplitude.

     TEIL 4: Auslegungsfrage
     → Welchen D-Wert würden Sie wählen?
     → Zwei Ziele: kleine stationäre Amplitude (Komfort/Sicherheit) vs. schnelles
       Einschwingen bei Einzelhindernis (Handling).
     → Optimum: D ≈ 0.3–0.5 als Kompromiss (keine eindeutige Antwort, Ingenieururteil). -->

Ein Fahrzeug fährt über eine sinusförmige Fahrbahn
$z_\text{Bahn}(t) = A_\text{Bahn}\sin(\Omega t)$ mit $A_\text{Bahn} = 0.02\,\text{m}$.
Die Anregungskraft auf die Karosserie ist $F(t) = k\,z_\text{Bahn}(t)$. Das
Federungsmodell lautet:

$$m\ddot{x} + c\dot{x} + kx = k\,A_\text{Bahn}\sin(\Omega t).$$

Gegeben: $m = 250\,\text{kg}$, $k = 16000\,\text{N/m}$,
$\omega_0 = 8\,\text{rad/s}$.

**Teil 1 – Zeitsignale bei Resonanz**

1. Lösen Sie die DGL für $\Omega = \omega_0$ und
   $D \in \{0.1,\, 0.3,\, 1.0\}$ für $t \in [0, 30\,\text{s}]$.
   Stellen Sie alle drei $x(t)$-Verläufe in einem Plot dar.
   Lesen Sie die stationäre Amplitude für jeden $D$-Wert ab.

**Teil 2 – Resonanzkurven**

2. Berechnen Sie für alle drei $D$-Werte die Resonanzkurve
   $A(\Omega)$ für $\Omega \in [0.2\,\omega_0,\, 2.0\,\omega_0]$
   und stellen Sie die drei Kurven in einem gemeinsamen Plot dar.

**Teil 3 – Energiedissipation**

3. Berechnen Sie die Verlustleistung $P(t) = c\,\dot{x}(t)^2$
   aus `sol.y[1]` und die kumulierte dissipierte Energie
   $E_\text{diss}(t) = \int_0^t P\,\mathrm{d}t$
   mit `scipy.integrate.cumulative_trapezoid`.
   Vergleichen Sie $E_\text{diss}(30\,\text{s})$ für $D = 0.1$ und $D = 1.0$
   bei Resonanzanregung. Welcher Wert ist größer, und warum?

**Teil 4 – Auslegungsfrage**

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
<!-- TODO: Lösung zu Aufgabe 11.7.
     Teil 1: stationäre Amplituden bei Omega=omega_0:
       D=0.1: A_stat ≈ 0.1005 m (10x Fahrbahnunebenheit!)
       D=0.3: A_stat ≈ 0.0350 m (1.75x)
       D=1.0: A_stat ≈ 0.0100 m (0.5x, gedämpft unter Fahrbahnunebenheit)

     Teil 3: E_diss(30s) bei Omega=omega_0:
       D=0.1: c=400 N*s/m, große Amplitude → E_diss >> D=1.0
       D=1.0: c=4000 N*s/m, kleine Amplitude → P_stat klein trotz großem c
       → D=0.1 dissipiert im Resonanzfall mehr Energie (stationäre Leistung ≈ 259 W)
         als D=1.0 (≈ 12.8 W). Kleine Dämpfung + Resonanz = viel Energie.

     Teil 4 (Ingenieururteil, keine eindeutige Antwort):
       Komfort/Sicherheit: großes D für kleine Resonanzamplitude.
       Handling: D nahe 1 für schnelles Einschwingen nach Einzelhindernis.
       Kompromiss: D ≈ 0.3–0.5 (typischer Pkw: D ≈ 0.25–0.4). -->
````
