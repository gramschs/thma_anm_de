---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 10.1 Das Euler-Verfahren

In Kapitel 7 haben wir aus bekannten Positionswerten Ableitungen berechnet,
in Kapitel 9 aus bekannten Funktionswerten Integrale. Jetzt stellen wir die
Frage anders: Ein Metallstab mit Anfangstemperatur $T_0 = 80\,°C$ liegt in
einem Raum mit Umgebungstemperatur $T_\infty = 20\,°C$. Wir wissen, dass die
Temperatur schneller fällt, wenn der Stab noch heiß ist, und immer langsamer,
je näher er der Umgebungstemperatur kommt. Das **Newtonsche Abkühlungsgesetz**
fasst das in einer einzigen Gleichung zusammen:

$$\dot{T}(t) = -k\,\bigl(T(t) - T_\infty\bigr).$$

Wir kennen also nicht $T(t)$ direkt, sondern nur die Formel für die
Änderungsrate. Eine Gleichung, die eine unbekannte Funktion und ihre Ableitung
verknüpft, heißt **gewöhnliche Differentialgleichung** (kurz DGL). Gesucht ist
der Temperaturverlauf $T(t)$ für $t \geq 0$.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können erklären, was eine **gewöhnliche Differentialgleichung 1.
  Ordnung** ist, und ein physikalisches Beispiel nennen.
* [ ] Sie können das **explizite Euler-Verfahren** aus dem
  Vorwärts-Differenzenquotienten herleiten und als Python-Schleife
  implementieren.
* [ ] Sie wissen, was **Schrittweite** und **Anfangsbedingung** bedeuten, und
  können beschreiben, wie die Schrittweite die Genauigkeit beeinflusst.
* [ ] Sie können die Euler-Lösung mit einer analytischen Referenzlösung
* vergleichen und den globalen Fehler quantifizieren.
```

+++

## Was ist die Lösung des Newtonschen Abkühlungsgesetzes?

Bevor wir das Abkühlproblem numerisch lösen, verschaffen wir uns einen
Überblick: Wie sieht die Lösung eigentlich aus? Für das Newtonsche
Abkühlungsgesetz lässt sich die Gleichung analytisch lösen. Wir trennen die Variablen

\begin{equation*}
\frac{dT}{dt}=-k\,(T-T_{\infty}) \quad\Rightarrow\quad
\frac{dT}{T-T_{\infty}}=-k\, dt,
\end{equation*}

integrieren auf beiden Seiten

\begin{equation*}
\ln |T - T_{\infty}| = -kt + \tilde{C}
\end{equation*}

und erhalten die allgemeine Lösung

\begin{equation*}
T(t) = T_{\infty} + C\, e^{-kt}.
\end{equation*}

Mit Anfangsbedingung $T(0)=T_0$ lautet die Lösung

\begin{equation*}
T(t) = T_{\infty} + (T_0 - T_{\infty}) e^{-kt}.
\end{equation*}

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

# --- Physikalisches Modell: Newtonsches Abkühlungsgesetz ---
# dT/dt = -k * (T - T_inf)
# Anfangstemperatur T0, Umgebungstemperatur T_inf, Abkühlkonstante k

T0    = 80.0   # Anfangstemperatur in °C
T_inf = 20.0   # Umgebungstemperatur in °C
k     = 0.1    # Abkühlkonstante in 1/min

# --- Analytische Lösung (Referenz für alle weiteren Abschnitte) ---
# Durch Trennung der Variablen ergibt sich: T(t) = T_inf + (T0 - T_inf) * exp(-k*t)
# Diese Kurve ist unsere "Wahrheit", gegen die wir die numerischen Methoden messen.
t_fein  = np.linspace(0, 50, 500)
T_exakt = T_inf + (T0 - T_inf) * np.exp(-k * t_fein)

print(f"Anfangstemperatur:        {T0:.1f} °C")
print(f"Umgebungstemperatur:      {T_inf:.1f} °C")
print(f"Temperatur nach 10 min:   {T_inf + (T0 - T_inf) * np.exp(-k * 10):.2f} °C")
print(f"Temperatur nach 30 min:   {T_inf + (T0 - T_inf) * np.exp(-k * 30):.2f} °C")
print(f"Temperatur nach 50 min:   {T_inf + (T0 - T_inf) * np.exp(-k * 50):.2f} °C")

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(t_fein, T_exakt, color='#005A94', linewidth=2,
        label='Analytische Lösung $T(t) = T_\\infty + (T_0 - T_\\infty)\\,e^{-kt}$')
ax.axhline(T_inf, color='#E87846', linewidth=1.2, linestyle='--',
           label=f'Gleichgewicht $T_\\infty = {T_inf:.0f}\\,°C$')
ax.scatter([0], [T0], color='#E60000', s=60, zorder=5,
           label=f'Anfangsbedingung $T_0 = {T0:.0f}\\,°C$')
ax.set_xlabel('Zeit in min')
ax.set_ylabel('Temperatur in °C')
ax.set_title('Newtonsches Abkühlungsgesetz: analytische Lösung')
ax.legend(fontsize=9)
ax.grid(True)
plt.tight_layout()
plt.show()
```

Die Temperatur nähert sich asymptotisch der Umgebungstemperatur. Nach 50 min
ist der Unterschied bereits kleiner als 1 °C. Die Funktion $\dot{T}(t) =
-k\,(T - T_\infty)$ beschreibt eine einfache, aber wichtige Klasse von DGLen:
**autonome Differentialgleichungen 1. Ordnung**. "Autonom" bedeutet, dass die
rechte Seite nicht explizit von der Zeit $t$ abhängt, sondern nur vom
aktuellen Zustand $T$.

```{admonition} Begriffe im Überblick
:class: note
| Begriff | Bedeutung im Beispiel |
|---|---|
| **DGL 1. Ordnung** | Nur die erste Ableitung $\dot{T}$ taucht auf, keine $\ddot{T}$ |
| **Anfangsbedingung** | $T(0) = T_0 = 80\,°C$; legt die spezifische Lösungskurve fest |
| **Rechte Seite** $f(T)$ | $-k\,(T - T_\infty)$; gibt die Änderungsrate als Funktion des Zustands an |
| **Autonome DGL** | $f$ hängt nicht explizit von $t$ ab |
```

*Was passiert, wenn die Heizung im Raum läuft und $T_\infty$ selbst zeitabhängig
ist?* Dann ist die rechte Seite $f(t, T)$ und die DGL heißt nicht-autonom.
Diesen Fall behandeln wir in Abschnitt 10.3.

```{admonition} Mini-Übung
:class: tip
1. Die analytische Lösung lautet $T(t) = T_\infty + (T_0 - T_\infty)\,e^{-kt}$.
   Was ergibt sich für $T(0)$? Überprüfen Sie, dass die Anfangsbedingung
   erfüllt ist.
2. Für welchen Wert von $T$ gilt $\dot{T} = 0$? Was bedeutet das physikalisch,
   und warum ist das im Plot erkennbar?
3. Was würde sich ändern, wenn $k$ verdoppelt wird: Ändert sich die
   Gleichgewichtstemperatur? Ändert sich die Geschwindigkeit der Abkühlung?
   Beantworten Sie die Frage ohne Code.
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: $T(0) = T_\infty + (T_0 - T_\infty)\,e^{0} = T_\infty + (T_0 -
T_\infty) = T_0 = 80\,°C$. Die Anfangsbedingung ist also genau erfüllt.

Zu Frage 2: $\dot{T} = -k(T - T_\infty) = 0$ gilt genau dann, wenn
$T = T_\infty = 20\,°C$. Das ist der Gleichgewichtszustand: Der Stab hat
Umgebungstemperatur angenommen und kühlt nicht mehr weiter ab. Im Plot
ist das die Asymptote, der sich die Kurve von oben annähert.

Zu Frage 3: Die Gleichgewichtstemperatur $T_\infty = 20\,°C$ hängt nicht
von $k$ ab, sie ändert sich also nicht. Ein größeres $k$ bedeutet eine
schnellere Abkühlung: Der Exponent $-kt$ wächst schneller, die Kurve
fällt steiler ab und erreicht die Gleichgewichtstemperatur früher.
````

+++

## Ein Schritt nach vorne: das Euler-Verfahren

Wenn wir keine analytische Lösung kennen oder die Gleichung zu kompliziert
für eine geschlossene Formel ist, müssen wir die Lösung schrittweise
berechnen. Die Idee ist dieselbe wie in Kapitel 7 beim
Vorwärts-Differenzenquotienten. Wir schreiben:

$$\dot{T}(t_i) \approx \frac{T(t_{i+1}) - T(t_i)}{\Delta t}.$$

Die DGL sagt uns, dass $\dot{T}(t_i) = f(T_i)$. Setzen wir das ein und
lösen nach $T_{i+1}$ auf:

$$T_{i+1} = T_i + \Delta t \cdot f(T_i).$$

Das ist das **explizite Euler-Verfahren**: vom bekannten Zustand $T_i$ einen
Schritt der Länge $\Delta t$ in Richtung der aktuellen Steigung gehen.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

# --- Parameter (identisch zu oben) ---
T0    = 80.0
T_inf = 20.0
k     = 0.1
t_end = 50.0
dt    = 5.0    # Schrittweite in min (bewusst grob für Visualisierung)

# --- Rechte Seite der DGL als Python-Funktion ---
def f_abkuehlung(T):
    # Änderungsrate: dT/dt = -k * (T - T_inf)
    return -k * (T - T_inf)

# --- Euler-Schleife ---
n_schritte = int(t_end / dt)            # Anzahl der Zeitschritte
t_euler    = np.zeros(n_schritte + 1)   # Zeitpunkte (inklusive t=0)
T_euler    = np.zeros(n_schritte + 1)   # Temperatur  (inklusive T0)

# Anfangsbedingung
t_euler[0] = 0.0
T_euler[0] = T0

for i in range(n_schritte):
    # Euler-Update: T[i+1] = T[i] + dt * f(T[i])
    t_euler[i + 1] = t_euler[i] + dt
    T_euler[i + 1] = T_euler[i] + dt * f_abkuehlung(T_euler[i])

# --- Analytische Referenz an denselben Zeitpunkten ---
T_exakt_euler = T_inf + (T0 - T_inf) * np.exp(-k * t_euler)

# --- Ausgabe der ersten Schritte ---
print(f"{'t [min]':>10}  {'Euler [°C]':>12}  {'Exakt [°C]':>12}  {'Fehler [°C]':>12}")
print("-" * 52)
for i in range(n_schritte + 1):
    print(f"{t_euler[i]:>10.1f}  {T_euler[i]:>12.4f}  "
          f"{T_exakt_euler[i]:>12.4f}  {T_euler[i] - T_exakt_euler[i]:>+12.4f}")

# --- Plot: Euler vs. analytisch ---
t_fein  = np.linspace(0, t_end, 500)
T_fein  = T_inf + (T0 - T_inf) * np.exp(-k * t_fein)

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(t_fein, T_fein, color='#005A94', linewidth=2, label='Analytische Lösung')
ax.plot(t_euler, T_euler, color='#E60000', linewidth=1.5, linestyle='--',
        marker='o', markersize=6, label=f'Euler-Verfahren ($\\Delta t = {dt:.0f}$ min)')
ax.set_xlabel('Zeit in min')
ax.set_ylabel('Temperatur in °C')
ax.set_title('Euler-Verfahren vs. analytische Lösung')
ax.legend()
ax.grid(True)
plt.tight_layout()
plt.show()
```

Mit $\Delta t = 5$ min ist der Fehler bereits mit bloßem Auge sichtbar.
Das Euler-Verfahren "überschießt" in jedem Schritt leicht, weil es die
Steigung am linken Rand des Intervalls als konstant annimmt. In Wirklichkeit
nimmt die Abkühlrate während des Intervalls aber kontinuierlich ab.

```{admonition} Mini-Übung
:class: tip
1. Verfolgen Sie den ersten Euler-Schritt von Hand.
   Die Formel lautet: $T_1 = T_0 + \Delta t \cdot f(T_0)$.
   Setzen Sie $T_0 = 80\,°C$, $\Delta t = 5$ und $f(T_0) = -k\,(T_0 - T_\infty)$
   ein. Welchen Wert erhalten Sie für $T_1$? Lesen Sie den Wert anschließend
   aus der Tabelle ab und prüfen Sie Ihre Rechnung.
2. Ist der Fehler nach dem ersten Schritt positiv oder negativ? Was bedeutet
   das physikalisch: Überschätzt oder unterschätzt das Euler-Verfahren die
   Temperatur?
3. Warum wächst der Fehler im Plot mit der Zeit, obwohl jeder einzelne
   Schritt nur einen kleinen lokalen Fehler macht?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1:

```python
T0    = 80.0
T_inf = 20.0
k     = 0.1
dt    = 5.0
f_T0  = -k * (T0 - T_inf)    # = -0.1 * 60 = -6.0 °C/min
T1    = T0 + dt * f_T0       # = 80 + 5 * (-6.0) = 50.0 °C
print(f"T1 = {T1:.4f} °C")
```

Das ergibt $T_1 = 50.0\,°C$. Der Tabellenwert bestätigt dieses Ergebnis.

Zu Frage 2: Der Fehler nach dem ersten Schritt ist positiv
($T_\text{Euler} > T_\text{exakt}$). Das bedeutet, Euler überschätzt die
Temperatur. Das Verfahren nimmt an, dass die Abkühlrate während des gesamten
Intervalls konstant bei $-6.0\,°C/\text{min}$ bleibt. Tatsächlich sinkt die
Rate aber schon während des Schritts, weil die Temperaturdifferenz kleiner
wird. Euler kühlt daher zu wenig ab.

Zu Frage 3: Die Fehler der einzelnen Schritte addieren sich auf. Da der
Euler-Wert in jedem Schritt leicht zu hoch liegt, startet der nächste
Schritt bereits von einem zu hohen Wert aus. Dieser **globale Fehler**
(kumulierter Fehler über alle Schritte) wächst daher mit der Zeit.
````

+++

## Schrittweite und Genauigkeit

*Wie genau muss die Schrittweite sein?* Das hängt vom Problem ab, aber ein
einfaches Experiment zeigt den Zusammenhang sehr deutlich.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

# --- Parameter ---
T0    = 80.0
T_inf = 20.0
k     = 0.1
t_end = 50.0

def euler_abkuehlung(T0, T_inf, k, t_end, dt):
    """Euler-Verfahren für das Newtonsche Abkühlungsgesetz.

    Parameters
    ----------
    T0    : float  Anfangstemperatur in °C
    T_inf : float  Umgebungstemperatur in °C
    k     : float  Abkühlkonstante in 1/min
    t_end : float  Simulationsende in min
    dt    : float  Schrittweite in min

    Returns
    -------
    t, T  : np.ndarray  Zeitpunkte und Temperaturwerte
    """
    n_schritte = int(t_end / dt)
    t = np.zeros(n_schritte + 1)
    T = np.zeros(n_schritte + 1)
    t[0] = 0.0
    T[0] = T0
    for i in range(n_schritte):
        # Euler-Update: T[i+1] = T[i] + dt * f(T[i])
        t[i + 1] = t[i] + dt
        T[i + 1] = T[i] + dt * (-k * (T[i] - T_inf))
    return t, T

# --- Drei Schrittweiten im Vergleich ---
schrittweiten = [10.0, 2.0, 0.5]
farben        = ['#E60000', '#E87846', '#CCDEE9']

t_fein  = np.linspace(0, t_end, 500)
T_fein  = T_inf + (T0 - T_inf) * np.exp(-k * t_fein)

fig, axes = plt.subplots(1, 2, figsize=(12, 4))

# Linkes Bild: Temperaturverläufe
axes[0].plot(t_fein, T_fein, color='#005A94', linewidth=2.5,
             label='Analytisch (Referenz)')
for dt_i, farbe in zip(schrittweiten, farben):
    t_i, T_i = euler_abkuehlung(T0, T_inf, k, t_end, dt_i)
    axes[0].plot(t_i, T_i, color=farbe, linewidth=1.5, linestyle='--',
                 marker='o', markersize=4, label=f'Euler $\\Delta t = {dt_i}$ min')
axes[0].set_xlabel('Zeit in min')
axes[0].set_ylabel('Temperatur in °C')
axes[0].set_title('Temperaturverlauf')
axes[0].legend(fontsize=9)
axes[0].grid(True)

# Rechtes Bild: Absoluter Fehler bei t = 20 min als Funktion der Schrittweite
dt_werte      = [10.0, 5.0, 2.0, 1.0, 0.5, 0.2, 0.1]
fehler_bei_20 = []
T_exakt_20    = T_inf + (T0 - T_inf) * np.exp(-k * 20)

for dt_i in dt_werte:
    t_i, T_i = euler_abkuehlung(T0, T_inf, k, t_end, dt_i)
    # Nächster Gitterpunkt zu t = 20 min
    idx = int(20.0 / dt_i)
    fehler_bei_20.append(abs(T_i[idx] - T_exakt_20))

axes[1].loglog(dt_werte, fehler_bei_20, color='#005A94', linewidth=2,
               marker='o', markersize=6)
axes[1].set_xlabel('Schrittweite $\\Delta t$ in min')
axes[1].set_ylabel('Absoluter Fehler in °C')
axes[1].set_title('Globaler Fehler bei $t = 20$ min')
axes[1].grid(True, which='both')

plt.tight_layout()
plt.show()

# --- Zahlenwerte der Fehler ---
print(f"{'dt [min]':>10}  {'Fehler bei t=20 min [°C]':>26}")
print("-" * 40)
for dt_i, fe in zip(dt_werte, fehler_bei_20):
    print(f"{dt_i:>10.1f}  {fe:>26.6f}")
```

Der rechte Plot zeigt eine typische **Konvergenzgerade** im doppelt
logarithmischen Maßstab: Halbieren wir die Schrittweite, halbiert sich
auch der Fehler ungefähr. Das bedeutet eine **Konvergenzordnung** von
$O(\Delta t)$. Das Euler-Verfahren heißt deshalb ein Verfahren erster
Ordnung. Zum Vergleich: Die Trapezregel aus Kapitel 9 hatte Ordnung
$O(\Delta t^2)$. In Abschnitt 10.3 lernen wir Verfahren kennen, die
Ordnung 4 oder höher erreichen.

```{admonition} Mini-Übung
:class: tip
1. Lesen Sie aus der Tabelle ab: Um welchen Faktor ändert sich der Fehler,
   wenn die Schrittweite von $\Delta t = 2.0$ auf $\Delta t = 1.0$ min
   halbiert wird? Stimmt das mit der erwarteten Konvergenzordnung $O(\Delta t)$
   überein?
2. Schätzen Sie ohne Code: Welche Schrittweite brauchen Sie ungefähr, damit
   der Fehler bei $t = 20$ min unter $0.001\,°C$ liegt? Welchen Rechenaufwand
   (Anzahl Schritte) hätte das?
3. Das Euler-Verfahren hat Konvergenzordnung 1. Was bedeutet das konkret für
   die Effizienz: Wie viel mehr Rechenaufwand braucht man, um den Fehler um
   den Faktor 1000 zu verringern?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1:

```python
import numpy as np

T0    = 80.0
T_inf = 20.0
k     = 0.1
t_end = 50.0
T_exakt_20 = T_inf + (T0 - T_inf) * np.exp(-k * 20)

for dt_i in [2.0, 1.0]:
    t_i, T_i = euler_abkuehlung(T0, T_inf, k, t_end, dt_i)
    idx = int(20.0 / dt_i)
    fehler = abs(T_i[idx] - T_exakt_20)
    print(f"dt = {dt_i:.1f} min   Fehler = {fehler:.6f} °C")
```

Der Fehler halbiert sich beim Halbieren der Schrittweite (von ca. 0.44 auf
ca. 0.22 °C). Das entspricht genau der erwarteten Konvergenzordnung $O(\Delta t)$.

Zu Frage 2: Aus der Tabelle: $\Delta t = 0.1\,\text{min}$ liefert einen Fehler
von etwa $0.023\,°C$. Für $0.001\,°C$ braucht man rund die 23-fache Genauigkeit,
also grob $\Delta t \approx 0.004\,\text{min}$. Das entspricht etwa 12500 Schritten
für 50 min Simulationszeit.

Zu Frage 3: Konvergenzordnung 1 bedeutet: um den Fehler um Faktor 1000 zu
verringern, braucht man 1000-mal mehr Schritte. Das ist sehr teuer. Ein
Verfahren 4. Ordnung würde nur $1000^{1/4} \approx 5.6$-mal mehr Schritte
benötigen. Das ist der Hauptmotivation für höhere Verfahren wie Runge-Kutta,
die wir in Abschnitt 10.3 einführen.
````

+++

## Zusammenfassung und Ausblick

Eine gewöhnliche Differentialgleichung 1. Ordnung verknüpft eine unbekannte
Funktion mit ihrer Ableitung: $\dot{T} = f(t, T)$. Das explizite Euler-Verfahren
löst sie schrittweise: vom aktuellen Zustand $T_i$ wird die Steigung $f(T_i)$
berechnet und ein Schritt der Länge $\Delta t$ ausgeführt. Das Verfahren ist
einfach zu implementieren, hat aber Konvergenzordnung 1: für eine zehnfache
Genauigkeit braucht man zehnmal mehr Schritte.

In Abschnitt 10.2 vertiefen wir das Euler-Verfahren durch Präsenzübungen.
In Abschnitt 10.3 lernen wir `scipy.integrate.solve_ivp` kennen, das mit dem
Runge-Kutta-Verfahren vierter Ordnung arbeitet und bei gleichem Rechenaufwand
eine vielfach höhere Genauigkeit erreicht.
