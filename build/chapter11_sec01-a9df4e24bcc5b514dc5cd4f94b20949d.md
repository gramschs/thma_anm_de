---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 11.1 Systeme 1. Ordnung und freie Schwingung

In Kapitel 10 haben wir `solve_ivp` eingesetzt, um die Temperatur eines
abkühlenden Metallstabs zu berechnen. Der Zustandsvektor hatte genau einen
Eintrag: die Temperatur $T$. Nun stellen wir die Frage anders: Ein Fahrzeug
fährt über eine unebene Straße. Die Karosserie sitzt auf einer Feder, die
Auslenkung $x$ wechselt ständig, und das Newtonsche Gesetz $ma = F$ ergibt

$$m\,\ddot{x}(t) + k\,x(t) = 0.$$

Das ist eine **Differentialgleichung zweiter Ordnung**: Es taucht $\ddot{x}$
auf. `solve_ivp` erwartet aber Systeme erster Ordnung. Wir zeigen in diesem
Abschnitt, wie wir jede DGL 2. Ordnung durch eine einfache Substitution in
zwei DGLen 1. Ordnung umschreiben und damit `solve_ivp` ohne jede Änderung
einsetzen.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können eine **DGL 2. Ordnung** durch die Substitution $y_1 = x$,
  $y_2 = \dot{x}$ in ein System zweier DGLen 1. Ordnung überführen und den
  **Zustandsvektor** $\mathbf{y} = [x,\, \dot{x}]^\top$ benennen.
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

*Warum kann `solve_ivp` eine DGL zweiter Ordnung nicht direkt lösen?*
`solve_ivp` erwartet die Signatur `f(t, y)`, wobei `y` ein Vektor von
Zustandsgrößen ist und `f` deren zeitliche Ableitungen zurückgibt. Das
funktioniert nur für Systeme erster Ordnung. Wir müssen $\ddot{x}$ also
loswerden.

Die Idee ist einfach: Wir benennen die beiden Größen, die wir verfolgen
wollen, mit $y_1 = x$ (Position) und $y_2 = \dot{x}$ (Geschwindigkeit).
Dann gilt $\dot{y}_1 = y_2$ (die Ableitung der Position ist die
Geschwindigkeit) und $\dot{y}_2 = \ddot{x} = -(k/m)\,x = -(k/m)\,y_1$
(aus der DGL). Statt einer DGL 2. Ordnung haben wir jetzt zwei DGLen
1. Ordnung:

$$\frac{d}{dt}\begin{pmatrix} y_1 \\ y_2 \end{pmatrix}
= \begin{pmatrix} y_2 \\ -\dfrac{k}{m}\,y_1 \end{pmatrix}.$$

In Kapitel 10 hatte `y` einen Eintrag ($y = [T]$), hier hat `y` zwei
Einträge: $y = [x,\, \dot{x}]$. Das ist der einzige Unterschied im Code.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

# --- Modellparameter: Viertel-Fahrzeugmodell ---
# m = Viertel der Fahrzeugmasse (eine Radaufhängung)
# k_feder = Federkonstante der Radaufhängung
m       = 250.0              # Masse in kg
k_feder = 16000.0            # Federkonstante in N/m

# --- Eigenkreisfrequenz und Schwingungsdauer ---
# omega_0 = sqrt(k/m): Wie schnell schwingt das System ohne äußere Kraft?
# T_0 = 2*pi / omega_0: Zeit für eine vollständige Schwingung
omega_0 = np.sqrt(k_feder / m)    # Eigenkreisfrequenz in rad/s
f_0     = omega_0 / (2 * np.pi)   # Eigenfrequenz in Hz
T_0     = 1 / f_0                 # Schwingungsdauer in s

print(f"Eigenkreisfrequenz: omega_0 = {omega_0:.4f} rad/s")
print(f"Eigenfrequenz:      f_0     = {f_0:.4f} Hz")
print(f"Schwingungsdauer:   T_0     = {T_0:.4f} s")

# --- Analytische Lösung als Referenz ---
# Für x(0) = x0, x_dot(0) = 0 gilt: x(t) = x0 * cos(omega_0 * t)
x0    = 0.05                             # Anfangsauslenkung in m (5 cm Bodenwelle)
t_ref = np.linspace(0, 5, 500)
x_ref = x0 * np.cos(omega_0 * t_ref)    # analytische Lösung

fig, ax = plt.subplots(figsize=(8, 3))
ax.plot(t_ref, x_ref * 100, color='#005A94')
ax.axhline(0, color='black', linewidth=0.8, linestyle='--')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Auslenkung in cm')
ax.set_title('Federschwinger ohne Dämpfung: analytische Lösung')
plt.tight_layout()
plt.show()
```

Die **Eigenfrequenz** $\omega_0 = \sqrt{k/m} = 8\,\text{rad/s}$ beschreibt,
wie schnell die Feder die Masse zurückzieht. Die Karosserie schwingt etwa
1,3 Mal pro Sekunde. Ohne Dämpfung würde das Fahrzeug nach einer Bodenwelle
endlos weiter wippen. Die analytische Lösung $x(t) = x_0 \cos(\omega_0 t)$
ist unsere Referenz für die numerische Überprüfung im nächsten Unterabschnitt.

```{admonition} Mini-Übung
:class: tip
1. Was passiert mit der Schwingungsdauer $T_0$, wenn die Federkonstante $k$
   verdoppelt wird? Antwort ohne Code.
2. Warum braucht man für diese DGL **zwei** Anfangsbedingungen ($x_0$ und
   $v_0 = \dot{x}(0)$), obwohl die Abkühlung aus Kapitel 10 nur eine
   brauchte ($T_0$)?
3. Berechnen Sie $T_0$ für $k = 32000\,\text{N/m}$ (doppelte Federsteifigkeit)
   ohne Code.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: Verdoppelt man $k$, wird $\omega_0^{neu} = \sqrt{2k/m} =
\sqrt{2}\,\omega_0 \approx 1{,}41\cdot\omega_0$. Die Schwingungsdauer
$T_0 = 2\pi/\omega_0$ wird kürzer; das System schwingt schneller.

Zu Frage 2: Eine DGL 2. Ordnung hat zwei Integrationskonstanten. Für eine
eindeutige Lösung braucht man daher zwei Anfangsbedingungen: die Auslenkung
$x_0$ und die Geschwindigkeit $v_0$ zum Startzeitpunkt. Die Abkühlung ist
nur erster Ordnung und hat deshalb nur eine Integrationskonstante.

Zu Frage 3: $\omega_0^{neu} = \sqrt{32000/250} = \sqrt{128} \approx
11{,}31\,\text{rad/s}$, $T_0^{neu} = 2\pi/11{,}31 \approx 0{,}556\,\text{s}$.
````

+++

## solve_ivp für den Federschwinger

*Wie schreibt man `f(t, y)` für eine DGL zweiter Ordnung?* Wir übersetzen
das System von oben direkt in Python: `y[0]` ist die Position $x$, `y[1]`
ist die Geschwindigkeit $\dot{x}$. Die Funktion gibt beide Ableitungen
zurück, genau wie in Kapitel 10, nur mit zwei statt einem Eintrag.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

# --- Parameter (identisch zu oben) ---
m       = 250.0
k_feder = 16000.0
omega_0 = np.sqrt(k_feder / m)

# --- Anfangsbedingungen ---
x0 = 0.05   # Anfangsauslenkung in m (Bodenwelle)
v0 = 0.0    # Anfangsgeschwindigkeit in m/s (aus der Ruhe)

# --- Rechte Seite als System 1. Ordnung ---
# y[0] = x       (Position in m)
# y[1] = x_dot   (Geschwindigkeit in m/s)
#
# Ableitung von y[0]: dx/dt = x_dot = y[1]
# Ableitung von y[1]: d(x_dot)/dt = x_ddot = -k/m * x = -(k_feder/m) * y[0]
#
# WICHTIG: Rückgabe in derselben Reihenfolge wie y0 = [x0, v0]
def f_schwinger(t, y):
    x_dot  = y[1]                        # Ableitung der Position = Geschwindigkeit
    x_ddot = -(k_feder / m) * y[0]      # Ableitung der Geschwindigkeit = Beschleunigung
    return [x_dot, x_ddot]

# --- solve_ivp Aufruf: y0 hat jetzt ZWEI Einträge ---
t_eval = np.linspace(0, 5, 501)
sol = solve_ivp(fun=f_schwinger, t_span=(0, 5), y0=[x0, v0], t_eval=t_eval)

# --- Rückgabe auslesen ---
# sol.y.shape = (2, 501): Zeile 0 = Position, Zeile 1 = Geschwindigkeit
print(f"Integration erfolgreich: {sol.success}")
print(f"sol.y.shape = {sol.y.shape}  --> {sol.y.shape[0]} Zustände, "
      f"{sol.y.shape[1]} Zeitpunkte")

# --- Vergleich mit analytischer Lösung ---
x_analytisch = x0 * np.cos(omega_0 * t_eval)

fig, axes = plt.subplots(2, 1, figsize=(8, 5), sharex=True)

# Position: numerisch vs. analytisch
axes[0].plot(t_eval, sol.y[0] * 100, color='#005A94', label='solve_ivp (RK45)')
axes[0].plot(t_eval, x_analytisch * 100, color='#E60000',
             linestyle='--', linewidth=1.5, label='analytisch')
axes[0].set_ylabel('Auslenkung in cm')
axes[0].legend()

# Geschwindigkeit
axes[1].plot(t_eval, sol.y[1], color='#005A94', label='Geschwindigkeit ẋ(t)')
axes[1].set_ylabel('Geschwindigkeit in m/s')
axes[1].set_xlabel('Zeit in s')
axes[1].legend()

plt.suptitle('Federschwinger: solve_ivp vs. analytische Lösung')
plt.tight_layout()
plt.show()

# --- Genauigkeitscheck ---
fehler_max = np.max(np.abs(sol.y[0] - x_analytisch))
print(f"Maximaler Fehler in x(t): {fehler_max:.2e} m")
```

Die beiden Kurven liegen nahezu perfekt übereinander; der maximale Fehler
liegt im Bereich $10^{-10}\,\text{m}$, also weit unterhalb jeder
Messgrenze. Im Vergleich zu Kapitel 10 ist der einzige Unterschied: `y0`
hat zwei Einträge, und `sol.y` hat zwei Zeilen statt einer.

```{admonition} Hinweis: Zustandsvektor richtig aufstellen
:class: warning
`f(t, y)` muss immer **zwei** Einträge zurückgeben, weil `y` zwei Einträge
hat. Der häufigste Fehler ist eine vertauschte Reihenfolge:
`return [x_ddot, x_dot]` statt `return [x_dot, x_ddot]`. Die Reihenfolge
in der Rückgabe muss der Reihenfolge in `y0` entsprechen: wenn `y0 = [x0,
v0]`, dann ist `y[0]` immer die Position und `y[1]` immer die Geschwindigkeit.
```

```{admonition} Mini-Übung
:class: tip
1. Was ist `sol.y.shape`, wenn `t_eval = np.linspace(0, 5, 501)` übergeben
   wird? Was bedeuten die beiden Dimensionen?
2. Was gibt `sol.y[1, 0]` zurück? Was gibt `sol.y[1, -1]` zurück?
   Antworten Sie, bevor Sie den Code ausführen.
3. Starten Sie das System mit `x0 = 0.0` und `v0 = 1.0` (Impuls statt
   Auslenkung). Welche Amplitude von $x(t)$ erwarten Sie? Überprüfen Sie
   die Schätzung im Code.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: `sol.y.shape = (2, 501)`. Die erste Dimension (2) ist die Anzahl
der Zustände (Position und Geschwindigkeit), die zweite (501) die Anzahl der
Ausgabezeitpunkte.

Zu Frage 2: `sol.y[1, 0]` ist die Anfangsgeschwindigkeit, also `v0 = 0.0`.
`sol.y[1, -1]` ist die Geschwindigkeit zum Zeitpunkt $t = 5\,\text{s}$.
Da der Schwinger periodisch ist, ist sie betragsmäßig nahezu identisch mit
dem Wert bei $t = 0$.

Zu Frage 3: Für `x0 = 0`, `v0 = 1` gilt $x(t) = (v_0/\omega_0)\sin(\omega_0
t)$. Die Amplitude ist $v_0/\omega_0 = 1/8 = 0{,}125\,\text{m}$.

```python
sol_impuls = solve_ivp(f_schwinger, (0, 5), [0.0, 1.0], t_eval=t_eval)
print(f"Amplitude: {np.max(np.abs(sol_impuls.y[0])):.4f} m")
# Ausgabe: Amplitude: 0.1250 m
```
````

+++

## Das Phasenporträt

Bisher haben wir $x(t)$ und $\dot{x}(t)$ jeweils gegen die Zeit aufgetragen.
Es gibt eine zweite Möglichkeit: das **Phasenporträt**. Dabei tragen wir
$\dot{x}$ auf der y-Achse gegen $x$ auf der x-Achse auf, ohne die Zeit
explizit darzustellen. Jeder Punkt im Phasenporträt beschreibt einen
vollständigen Systemzustand: Wo ist die Masse, und wie schnell bewegt sie
sich?

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

m       = 250.0
k_feder = 16000.0

def f_schwinger(t, y):
    return [y[1], -(k_feder / m) * y[0]]

t_eval = np.linspace(0, 10, 2001)

# --- Drei verschiedene Anfangsbedingungen (unterschiedlich viel Energie) ---
anfangsbedingungen = [
    (0.02, 0.0),   # kleine Auslenkung, kleines x0
    (0.05, 0.0),   # mittlere Auslenkung
    (0.00, 0.8),   # Impuls statt Auslenkung (v0 != 0)
]
farben = ['#005A94', '#E60000', '#E87846']
labels = ['x₀=2 cm, v₀=0', 'x₀=5 cm, v₀=0', 'x₀=0, v₀=0.8 m/s']

fig, axes = plt.subplots(1, 2, figsize=(10, 4))

for (xi, vi), farbe, label in zip(anfangsbedingungen, farben, labels):
    sol = solve_ivp(f_schwinger, (0, 10), [xi, vi], t_eval=t_eval)
    # Zeitsignal
    axes[0].plot(t_eval, sol.y[0] * 100, color=farbe, label=label)
    # Phasenporträt: x_dot über x, KEIN Zeitindex als Achse
    axes[1].plot(sol.y[0] * 100, sol.y[1], color=farbe, label=label)

axes[0].set_xlabel('Zeit in s')
axes[0].set_ylabel('Auslenkung in cm')
axes[0].set_title('Zeitsignale')
axes[0].legend(fontsize=8)

axes[1].set_xlabel('Auslenkung x in cm')
axes[1].set_ylabel('Geschwindigkeit ẋ in m/s')
axes[1].set_title('Phasenporträt')
axes[1].legend(fontsize=8)

plt.tight_layout()
plt.show()
```

Jede Anfangsbedingung ergibt eine geschlossene Ellipse im Phasenporträt.
Das ist kein Zufall: Beim ungedämpften Schwinger bleibt die mechanische
Gesamtenergie $E = \frac{1}{2}m\dot{x}^2 + \frac{1}{2}kx^2$ konstant.
Diese Gleichung beschreibt in der $x$-$\dot{x}$-Ebene genau eine Ellipse.
Größere Anfangsbedingungen bedeuten mehr Energie und damit eine größere
Ellipse. In Abschnitt 11.3 fügen wir Dämpfung hinzu. Dann verliert das
System Energie, und die Ellipsen werden zu nach innen spiralierenden Kurven.

```{admonition} Mini-Übung
:class: tip
1. In welchem Punkt der Ellipse ist $|\dot{x}|$ maximal, und in welchem
   minimal? Was bedeutet das physikalisch?
2. Warum ergibt sich eine Ellipse und kein Kreis? Von welchen
   Systemparametern hängt das Verhältnis der Halbachsen ab? Antwort ohne
   Code.
3. Was würde sich am Phasenporträt ändern, wenn $k$ verdoppelt wird und
   $x_0$ gleich bleibt?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: $|\dot{x}|$ ist maximal, wenn $x = 0$ ist (Masse passiert die
Ruhelage mit maximaler Geschwindigkeit, die gesamte Energie steckt in der
kinetischen Energie). $|\dot{x}| = 0$ gilt bei maximaler Auslenkung
$x = \pm x_{\max}$ (Umkehrpunkte, die gesamte Energie steckt in der
Federenergie).

Zu Frage 2: Die Energiegleichung $\frac{1}{2}m\dot{x}^2 + \frac{1}{2}kx^2
= E$ hat auf der $x$-Achse die Halbachse $\sqrt{2E/k}$ und auf der
$\dot{x}$-Achse die Halbachse $\sqrt{2E/m}$. Das Verhältnis ist
$\sqrt{k/m} = \omega_0$. Für $m = k$ wäre es ein Kreis.

Zu Frage 3: Die Ellipse wird in $x$-Richtung schmaler (kleinere maximale
Auslenkung bei gleicher Energie) und in $\dot{x}$-Richtung breiter. Die
Eigenfrequenz steigt auf $\omega_0^{neu} = \sqrt{2}\,\omega_0 \approx
11{,}3\,\text{rad/s}$.
````

+++

## Zusammenfassung und Ausblick

Eine DGL 2. Ordnung wie $m\ddot{x} + kx = 0$ wird durch die Substitution
$y_1 = x$, $y_2 = \dot{x}$ in ein System zweier DGLen 1. Ordnung
umgeschrieben. `solve_ivp` löst das System ohne jede Anpassung: `y0`
bekommt zwei Einträge `[x0, v0]`, `f(t, y)` gibt zwei Werte zurück.
`sol.y[0]` enthält die Position, `sol.y[1]` die Geschwindigkeit. Die
Eigenfrequenz $\omega_0 = \sqrt{k/m}$ bestimmt, wie schnell der ungedämpfte
Schwinger schwingt. Das Phasenporträt zeigt geschlossene Ellipsen, weil die
mechanische Energie erhalten bleibt.

In Abschnitt 11.3 erweitern wir das Modell um Dämpfung ($c\,\dot{x}$) und
eine externe Kraft $F(t)$. Das Phasenporträt spiraliert dann in den
Ursprung, und bei passender Anregungsfrequenz entsteht Resonanz.
