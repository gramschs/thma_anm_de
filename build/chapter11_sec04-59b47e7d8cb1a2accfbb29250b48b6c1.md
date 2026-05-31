---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 11.4 Übungen zur gedämpften Schwingung

````{admonition} Übung 11.3 (✩)
:class: tip
<!-- TODO: Aufgabe 11.3 (~20 min)
     Thema: Vorgegebener Code für den gedämpften Schwinger lesen und Verhalten vorhersagen.

     SZENARIO: Code simuliert den Masse-Feder-Dämpfer für vier D-Werte und gibt
     T_einschwing (Zeit bis |x| < 0.001 m erstmals) aus. Kein Bug — reines Lesen.

     VORGEGEBENER CODE:
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
         sol = solve_ivp(lambda t, y: f_gedaempft(t, y, c),
                         (0, 10), [x0, v0], t_eval=t_eval)
         unter_schwelle = np.where(np.abs(sol.y[0]) < 1e-3)[0]
         if len(unter_schwelle) > 0:
             t_ein = t_eval[unter_schwelle[0]]
         else:
             t_ein = float('inf')
         print(f"D = {D:.1f}:  T_einschwing = {t_ein:.3f} s")
     ```

     FRAGEN:
     1. Ohne Code: Für welchen D-Wert schwingt das System? Für welchen nicht?
        Welche Fälle entsprechen unter-, über- und kritisch gedämpft?
     2. Für D = 0.1: Hat das System in den ersten 10 s die Schwelle 0.001 m
        unterschritten? Schätzen Sie ab (gedämpfte Eigenfrequenz
        omega_d = omega_0 * sqrt(1-D^2), Abklingrate = D*omega_0).
     3. Welchen D-Wert erwarten Sie als Minimum in T_einschwing? Begründen Sie.
     4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.

     VORBERECHNETE AUSGABE (zum Abgleich nach dem Ausführen):
     D = 0.1:  T_einschwing = inf      (schwingt 10 s lang über Schwelle)
     D = 0.3:  T_einschwing = ...      (TODO: berechnen)
     D = 1.0:  T_einschwing = ...      (TODO: berechnen)
     D = 2.0:  T_einschwing = ...      (TODO: berechnen)
     → D = 1.0 hat kleinste endliche Einschwingzeit (aperiodischer Grenzfall) -->

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
    sol = solve_ivp(lambda t, y: f_gedaempft(t, y, c),
                    (0, 10), [x0, v0], t_eval=t_eval)
    unter_schwelle = np.where(np.abs(sol.y[0]) < 1e-3)[0]
    if len(unter_schwelle) > 0:
        t_ein = t_eval[unter_schwelle[0]]
    else:
        t_ein = float('inf')
    print(f"D = {D:.1f}:  T_einschwing = {t_ein:.3f} s")
```

1. Ohne Code: Für welchen D-Wert schwingt das System? Für welchen nicht?
   Welche Fälle entsprechen unter-, über- und kritisch gedämpft?
2. Für $D = 0.1$: Hat das System in 10 s die Schwelle $10^{-3}$ m
   unterschritten? Schätzen Sie ab: Die Amplitude klingt ungefähr
   wie $e^{-D\,\omega_0\,t}$ ab.
3. Für welchen der vier $D$-Werte erwarten Sie die kürzeste endliche
   Einschwingzeit? Begründen Sie in einem Satz.
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
<!-- TODO: Lösung zu Aufgabe 11.3.
     Zu Frage 1: D=0.1 und D=0.3: schwingend (D<1); D=1.0: aperiodischer Grenzfall;
     D=2.0: Kriechfall (D>1).
     Zu Frage 2: Abklingrate = D*omega_0 = 0.1*8 = 0.8 s^{-1}.
     Amplitude nach 10 s: 0.05*exp(-8) ≈ 0.05*0.000335 ≈ 0.0000168 m < 0.001 m.
     ABER: Überprüfen ob tatsächlich |x| < 0.001 gilt (Einhüllende, nicht x selbst).
     → TODO: tatsächliche Ausgabe einfügen nach Ausführen.
     Zu Frage 3: D=1.0 hat theoretisch schnellste Einschwingzeit ohne Überschwingen.
     In der Praxis ist D leicht über 1 noch etwas schneller für diese Schwelle. -->
````

````{admonition} Übung 11.4 (✩✩)
:class: tip
<!-- TODO: Aufgabe 11.4 (~25 min)
     Thema: Resonanzkurve numerisch berechnen.

     SZENARIO: Erzwungener Schwinger mit D = 0.2.
     Schleife über Omega in [0.2*omega_0, 2.0*omega_0] (z.B. 40 Werte).
     Für jeden Omega-Wert: solve_ivp für t_end = 30 s, stationäre Amplitude
     A = max(|x|) im letzten Drittel. Resonanzkurve A(Omega) plotten.
     Analytische Resonanzfrequenz: Omega_res = omega_0 * sqrt(1 - 2*D^2).

     PARAMETER:
     m=250, k=16000, D=0.2, F0=500 N, x0=0, v0=0

     VORBERECHNETE WERTE:
     omega_0 = 8.0 rad/s
     c_krit = 4000 N*s/m, c = 0.2*4000 = 800 N*s/m
     Omega_res_ana = omega_0 * sqrt(1 - 2*0.04) = 8*sqrt(0.92) ≈ 7.67 rad/s
     A_max_ana = F0 / (k * 2*D*sqrt(1-D^2)) = 500 / (16000*2*0.2*sqrt(0.96))
               ≈ 500 / (16000*0.392) ≈ 0.0798 m

     FRAGEN:
     1. Implementieren Sie f_erzwungen(t, y, Omega) und schreiben Sie die
        Schleife über Omega.
     2. Plotten Sie A(Omega) und markieren Sie omega_0 und Omega_res.
     3. Lesen Sie Omega_res numerisch ab (argmax) und vergleichen Sie mit
        der analytischen Formel Omega_res = omega_0*sqrt(1-2*D^2).
     4. Verständnisfrage: Warum liegt Omega_res leicht unter omega_0?
        Erklären Sie physikalisch in einem Satz. -->

Gegeben: Erzwungener Schwinger $m\ddot{x} + c\dot{x} + kx = F_0\sin(\Omega t)$
mit $m = 250\,\text{kg}$, $k = 16000\,\text{N/m}$, $D = 0.2$,
$F_0 = 500\,\text{N}$, $x(0) = 0$, $\dot{x}(0) = 0$.

1. Schreiben Sie `f_erzwungen(t, y, Omega)` und implementieren Sie eine
   Schleife über $\Omega \in [0.2\,\omega_0,\; 2.0\,\omega_0]$ (mindestens
   40 gleichmäßig verteilte Werte). Berechnen Sie für jeden $\Omega$-Wert
   die stationäre Amplitude als `np.max(np.abs(sol.y[0, -n:]))`, wobei `n`
   das letzte Drittel der Zeitpunkte abdeckt.
2. Plotten Sie die Resonanzkurve $A(\Omega)$ und markieren Sie $\omega_0$
   als vertikale Linie.
3. Bestimmen Sie $\Omega_\text{res}$ numerisch als `Omega_werte[np.argmax(A_werte)]`
   und vergleichen Sie mit der analytischen Formel
   $\Omega_\text{res} = \omega_0\sqrt{1 - 2D^2}$.
4. Warum liegt $\Omega_\text{res}$ leicht unterhalb von $\omega_0$?
   Erklären Sie in einem Satz.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
<!-- TODO: Lösung zu Aufgabe 11.4.
     omega_0 = 8.0 rad/s, c = 800 N*s/m
     Omega_res_ana = 8*sqrt(1-2*0.04) = 8*sqrt(0.92) ≈ 7.673 rad/s
     A_max ≈ F0/(k*2*D*sqrt(1-D^2)) ≈ 0.0798 m
     Zu Frage 4: Die Dämpfung reduziert die effektive Steifigkeit des Systems
     im Frequenzbereich leicht, sodass das Amplitudenmaximum nicht bei der
     ungedämpften Eigenfrequenz omega_0 liegt, sondern darunter.
     Für D=0: Omega_res = omega_0; für D>0: Omega_res < omega_0. -->
````
