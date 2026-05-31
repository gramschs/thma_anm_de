---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 11.1 Systeme 1. Ordnung und freie Schwingung

<!-- TODO: Motivationsabsatz (~100 Wörter)
     Brücke zu Kap. 10: Bisher ein Zustand (T oder v). Jetzt zwei: Position x
     und Geschwindigkeit x_dot. Einführendes Beispiel: Fahrzeugfeder ohne Dämpfer.
     Zentrales Modell: m*x_ddot + k*x = 0, Parameter m=250 kg, k=16000 N/m.
     Analytische Lösung x(t) = x0 * cos(omega_0 * t) als Referenz vorstellen. -->

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können eine **DGL 2. Ordnung** durch die Substitution $y_1 = x$,
  $y_2 = \dot{x}$ in ein System zweier DGLen 1. Ordnung überführen und
  den Zustandsvektor $\mathbf{y} = [x,\, \dot{x}]^\top$ benennen.
* [ ] Sie können `solve_ivp` für ein zweidimensionales System aufrufen, die
  Anfangsbedingung als `y0=[x0, v0]` angeben und `sol.y[0]` (Position) sowie
  `sol.y[1]` (Geschwindigkeit) korrekt auslesen.
* [ ] Sie kennen die **Eigenfrequenz** $\omega_0 = \sqrt{k/m}$ und können sie
  aus dem Zeitsignal $x(t)$ ablesen.
* [ ] Sie können ein **Phasenporträt** ($\dot{x}$ über $x$) erstellen und
  erklären, warum es beim ungedämpften Schwinger eine Ellipse ergibt.
```

+++

## Von der DGL 2. Ordnung zum System 1. Ordnung

<!-- TODO: Unterabschnitt 1 (~10 min)
     - DGL m*x_ddot + k*x = 0 einführen (Federschwinger ohne Dämpfung)
     - Substitution y1 = x, y2 = x_dot erklären und motivieren
     - System in Vektorform aufschreiben: d/dt [y1, y2] = [y2, -k/m * y1]
     - Vergleich mit Kap. 10: dort y = [T] (1 Eintrag), jetzt y = [x, x_dot] (2 Einträge)
     - Rhetorische Frage kursiv, Rückverweis auf Kap. 10 -->

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

# --- Modellparameter: Viertel-Fahrzeugmodell ---
m       = 250.0              # Masse in kg
k_feder = 16000.0            # Federkonstante in N/m
omega_0 = np.sqrt(k_feder / m)   # Eigenkreisfrequenz in rad/s
f_0     = omega_0 / (2 * np.pi)  # Eigenfrequenz in Hz

print(f"Eigenkreisfrequenz: omega_0 = {omega_0:.4f} rad/s")
print(f"Eigenfrequenz:      f_0     = {f_0:.4f} Hz")
print(f"Schwingungsdauer:   T_0     = {1/f_0:.4f} s")

# TODO: analytische Lösung x(t) = x0 * cos(omega_0 * t) berechnen und plotten
```

<!-- TODO: Prosatext nach dem Code (~80 Wörter):
     Erklärung der ausgegebenen Werte, Bedeutung der Eigenfrequenz für die
     Fahrzeugtechnik, Überleitung zur numerischen Lösung. -->

```{admonition} Mini-Übung
:class: tip
<!-- TODO: Mini-Übung (5 min), mind. 1 Verständnisfrage:
     1. Welche physikalische Bedeutung hat omega_0?
     2. Warum braucht man ZWEI Anfangsbedingungen (x0 und v0)?
     3. Verständnisfrage: Was passiert mit T_0, wenn k verdoppelt wird? -->
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
<!-- TODO: Lösung zur Mini-Übung -->
````

+++

## solve_ivp für den Federschwinger

<!-- TODO: Unterabschnitt 2 (~10 min)
     - f(t, y) mit 2 Rückgabewerten: [y[1], -k/m * y[0]]
     - solve_ivp Aufruf mit y0=[x0, v0], sol.y.shape = (2, n)
     - sol.y[0] = Position, sol.y[1] = Geschwindigkeit
     - Vergleich mit analytischer Lösung x(t) = x0*cos(omega_0*t)
     - Hinweis-Box: Signatur f(t, y), Rückgabe immer Liste mit ZWEI Einträgen -->

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

m       = 250.0
k_feder = 16000.0
omega_0 = np.sqrt(k_feder / m)

# Anfangsbedingungen
x0 = 0.05    # Anfangsauslenkung in m (5 cm)
v0 = 0.0     # Anfangsgeschwindigkeit in m/s

# --- Rechte Seite der DGL als System ---
def f_schwinger(t, y):
    # y[0] = x (Position in m)
    # y[1] = x_dot (Geschwindigkeit in m/s)
    # TODO: x_ddot = ... (aus m*x_ddot + k*x = 0)
    x_dot  = y[1]
    x_ddot = None   # TODO
    return [x_dot, x_ddot]

# TODO: solve_ivp aufrufen, sol.y.shape ausgeben
# TODO: Vergleich mit analytischer Lösung plotten
# TODO: Zweiten Plot: sol.y[1] (Geschwindigkeit) über die Zeit
```

<!-- TODO: Prosatext (~80 Wörter):
     Erklärung der sol.y-Struktur, warum die erste Zeile Position und die zweite
     Geschwindigkeit ist. Hinweis auf sol.y.shape = (2, n_zeitpunkte). -->

```{admonition} Hinweis: Zustandsvektor
:class: warning
<!-- TODO: Hinweis-Box:
     - f(t, y) muss immer eine Liste mit ZWEI Einträgen zurückgeben
     - y0 hat zwei Einträge: [Anfangsposition, Anfangsgeschwindigkeit]
     - Häufiger Fehler: y[0] und y[1] vertauschen oder nur einen Eintrag zurückgeben -->
```

```{admonition} Mini-Übung
:class: tip
<!-- TODO: Mini-Übung (5 min):
     1. Was ist sol.y.shape für dieses Problem mit t_eval=linspace(0,5,501)?
     2. Welche Zeile von sol.y ist die Position, welche die Geschwindigkeit?
     3. Verständnisfrage: Was erwartet man für die Amplitude, wenn v0 = 1 m/s
        und x0 = 0? Sagen Sie das qualitativ vorher. -->
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
<!-- TODO: Lösung zur Mini-Übung -->
````

+++

## Das Phasenporträt

<!-- TODO: Unterabschnitt 3 (~10 min)
     - Phasenporträt: x_dot (y-Achse) über x (x-Achse), ohne Zeit
     - ax.plot(sol.y[0], sol.y[1])
     - Für den ungedämpften Schwinger: Ellipse (Erhaltung der Energie)
     - Verschiedene Anfangsbedingungen → konzentrische Ellipsen
     - Vorausblick: In sec03 spiraliert die Kurve für D > 0 in den Ursprung -->

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

# TODO: Phasenporträt für drei verschiedene Anfangsbedingungen
# (x0, v0) in [(0.02, 0), (0.05, 0), (0.0, 0.8)]
# ax.plot(sol.y[0], sol.y[1]) für jede Anfangsbedingung
# ax.set_aspect('equal') für korrekte Ellipsenform
```

<!-- TODO: Prosatext (~80 Wörter):
     Interpretation des Phasenporträts: jeder Punkt = Systemzustand,
     Ellipse = periodische Bewegung (Energieerhaltung),
     äußere Ellipsen = mehr Energie (größere Amplitude).
     Vorausblick auf sec03: Dämpfung führt zu einwärts spiralierenden Orbits. -->

```{admonition} Mini-Übung
:class: tip
<!-- TODO: Mini-Übung (5 min):
     1. Verständnisfrage: In welchem Punkt der Ellipse ist |v| maximal,
        in welchem minimal? Was ist die physikalische Bedeutung?
     2. Was würde sich am Phasenporträt ändern, wenn k verdoppelt wird
        (bei gleichem x0)? Antwort ohne Code. -->
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
<!-- TODO: Lösung zur Mini-Übung -->
````

+++

## Zusammenfassung und Ausblick

<!-- TODO: Zusammenfassung (~80 Wörter):
     - DGL 2. Ordnung → 2D-Zustandsvektor → solve_ivp mit y0=[x0, v0]
     - sol.y[0]: Position, sol.y[1]: Geschwindigkeit
     - Eigenfrequenz omega_0 = sqrt(k/m), Phasenporträt als Ellipse
     - Ausblick: In sec03 kommt Dämpfung (c*x_dot) und externe Kraft F(t) hinzu;
       das Phasenporträt spiraliert dann in den Ursprung statt eine Ellipse zu bilden.
     - Vorwärtsverweis auf Kap. 11 in seiner Gesamtheit -->
