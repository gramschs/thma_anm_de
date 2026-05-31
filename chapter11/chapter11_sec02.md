---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 11.2 Übungen zum Federschwinger

````{admonition} Übung 11.1 (✩)
:class: tip
<!-- TODO: Aufgabe 11.1 (~20 min)
     Thema: Parameterfehler in f(t, y) entdecken und quantifizieren.

     SZENARIO: Vorgegebener Code für einen Federschwinger (m=250 kg, k=16000 N/m),
     bei dem in f die Rückgabe [y[1], -y[0]] steht statt [y[1], -(k/m)*y[0]].
     Das entspricht einem Schwinger mit omega_buggy = 1 rad/s statt omega_0 = 8 rad/s.
     Die Simulation läuft fehlerfrei durch, liefert aber falsche Ergebnisse.

     VORGEGEBENER CODE (vollständig, kein Bug im Aufruf — nur in f):
     ```python
     from scipy.integrate import solve_ivp
     import numpy as np

     m       = 250.0
     k_feder = 16000.0
     x0      = 0.05    # m
     v0      = 0.0     # m/s
     t_end   = 5.0     # s

     def f_schwinger_buggy(t, y):
         return [y[1], -y[0]]    # Zeile A

     t_eval = np.linspace(0, t_end, 501)
     sol    = solve_ivp(f_schwinger_buggy, (0, t_end), [x0, v0], t_eval=t_eval)

     T_buggy = 2 * np.pi   # abgelesene Periode aus dem Plot
     T_korrekt = 2 * np.pi / np.sqrt(k_feder / m)

     print(f"Periode buggy:   {T_buggy:.4f} s")
     print(f"Periode korrekt: {T_korrekt:.4f} s")
     print(f"Abweichung:      {abs(T_buggy - T_korrekt) / T_korrekt * 100:.1f} %")
     ```

     FRAGEN:
     1. Was berechnet Zeile A tatsächlich? Welche DGL löst der Code,
        und welche Eigenfrequenz hat sie? Antworten ohne Code.
     2. Um welchen Faktor weicht die berechnete Periode von der korrekten ab?
        Berechnen Sie den Faktor aus omega_buggy = 1 und omega_0 = sqrt(16000/250).
     3. Nennen Sie die Korrektur in Zeile A in einem Satz.
     4. Führen Sie den Code aus, korrigieren Sie Zeile A und überprüfen Sie,
        dass die korrigierte Periode mit T_korrekt übereinstimmt.

     LÖSUNG VORAB BERECHNEN:
     - omega_buggy = sqrt(1) = 1 rad/s → T_buggy = 2*pi ≈ 6.283 s
     - omega_0 = sqrt(16000/250) = 8 rad/s → T_korrekt = 2*pi/8 ≈ 0.785 s
     - Abweichung Faktor: T_buggy / T_korrekt = 8
     - Korrektur: return [y[1], -(k_feder/m) * y[0]] -->

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

1. Was berechnet Zeile A tatsächlich? Welche DGL löst der Code,
   und welche Eigenfrequenz hat sie? Antworten ohne Code.
2. Um welchen Faktor weicht die berechnete Periode von der korrekten ab?
   Berechnen Sie das Verhältnis $T_\text{buggy}/T_\text{korrekt}$ aus
   den Eigenfrequenzen.
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
<!-- TODO: Lösung zu Aufgabe 11.1.
     Zu Frage 1: Zeile A entspricht der DGL x_ddot = -x (ohne k/m-Faktor),
     also einem Schwinger mit omega_buggy = sqrt(1) = 1 rad/s.
     Zu Frage 2: T_buggy/T_korrekt = omega_0/omega_buggy = 8/1 = 8.
     Die berechnete Periode ist achtmal zu groß.
     Zu Frage 3: Korrektur: return [y[1], -(k_feder/m)*y[0]]
     Erwartete Ausgabe nach Korrektur:
       Periode buggy:   6.2832 s
       Periode korrekt: 0.7854 s
       Abweichung:      700.0 %
     Nach Korrektur:
       Periode korrekt: 0.7854 s  (beide Werte übereinstimmend) -->
````

````{admonition} Übung 11.2 (✩✩)
:class: tip
<!-- TODO: Aufgabe 11.2 (~25 min)
     Thema: Nichtlineares Pendel als System formulieren und mit Kleinwinkelnäherung vergleichen.

     SZENARIO: Das nichtlineare Pendel genügt der DGL
         phi_ddot = -(g/L) * sin(phi)
     mit g = 9.81 m/s², L = 1.0 m (Pendellänge).
     Zwei Anfangsauslenkungen: phi_0 = 5° (klein) und phi_0 = 90° (groß).
     Analytische Kleinwinkelperiode: T_0 = 2*pi*sqrt(L/g) ≈ 2.006 s.

     FRAGEN:
     1. Stellen Sie die DGL als System auf: Welche Substitution verwenden Sie?
        Schreiben Sie f(t, y) mit y[0] = phi, y[1] = phi_dot auf.
     2. Lösen Sie mit solve_ivp für beide Anfangsbedingungen (t_end = 10 s).
        Lesen Sie die Periode aus dem Plot ab (Zeit zwischen zwei Nulldurchgängen).
     3. Vergleichen Sie mit T_0 = 2*pi*sqrt(L/g). Um wie viel Prozent weicht
        die Periode für phi_0 = 90° ab?
     4. Verständnisfrage: Warum ist die Periode für phi_0 = 90° länger als T_0?
        Erklären Sie in einem Satz ohne Code.

     VORBERECHNETE WERTE:
     - T_0 = 2*pi*sqrt(1/9.81) ≈ 2.0064 s
     - T(phi_0=5°) ≈ 2.0072 s  (≈ 0.04 % Abweichung, fast gleich)
     - T(phi_0=90°) ≈ 2.365 s  (≈ 17.9 % länger als T_0)
     - Begründung: sin(phi) < phi für phi > 0, also ist die Rückstellkraft
       schwächer als in der Näherung → langsamere Schwingung. -->

Das nichtlineare Pendel der Länge $L$ genügt der DGL

$$\ddot{\phi} = -\frac{g}{L}\sin\phi.$$

Gegeben: $g = 9.81\,\text{m/s}^2$, $L = 1.0\,\text{m}$,
Anfangsgeschwindigkeit $\dot{\phi}(0) = 0$ für beide Fälle.

1. Stellen Sie die DGL als System erster Ordnung auf. Definieren Sie
   $y_1 = \phi$ und $y_2 = \dot{\phi}$ und schreiben Sie `f(t, y)` auf.
2. Lösen Sie mit `solve_ivp` für beide Anfangsbedingungen
   $\phi_0 = 5°$ und $\phi_0 = 90°$ (Winkel in Bogenmass umrechnen).
   Stellen Sie $\phi(t)$ für $t \in [0, 10\,\text{s}]$ dar.
3. Lesen Sie die Schwingungsperiode für beide Fälle aus dem Plot ab
   und vergleichen Sie mit der analytischen Kleinwinkelperiode
   $T_0 = 2\pi\sqrt{L/g}$.
4. Warum ist die Periode für $\phi_0 = 90°$ länger als $T_0$?
   Erklären Sie in einem Satz ohne Code.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
<!-- TODO: Lösung zu Aufgabe 11.2.
     Zu Frage 1: f(t, y) = [y[1], -(g/L)*np.sin(y[0])]
     Zu Frage 3: T_0 ≈ 2.006 s; T(5°) ≈ 2.007 s; T(90°) ≈ 2.365 s; Abweichung ≈ 17.9 %
     Zu Frage 4: sin(phi) < phi für phi > 0: Die tatsächliche Rückstellkraft
     ist schwächer als die lineare Näherung, daher dauert eine Schwingung länger. -->
````
