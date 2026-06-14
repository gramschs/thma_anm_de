---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 10.3 Das Runge-Kutta-Verfahren

In Abschnitt 10.1 haben wir das Euler-Verfahren als Schleife implementiert.
In Abschnitt 10.2 haben wir gesehen, dass zu große Schrittweiten
unphysikalische Ergebnisse erzeugen, und dass selbst bei kleinen Schrittweiten
ein merklicher Fehler bleibt. Beide Probleme lassen sich mit
`scipy.integrate.solve_ivp` lösen. Die Funktion löst dieselbe DGL, wählt die
Schrittweite aber automatisch und nutzt intern das
**Runge-Kutta-Verfahren der Ordnung 4/5** (RK45): ein Verfahren, das pro Schritt
zwar mehr Rechenaufwand kostet als Euler, aber eine drastisch höhere Genauigkeit
erreicht.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können `scipy.integrate.solve_ivp` mit dem Solver `RK45` aufrufen
  und die Rückgabefelder `sol.t`, `sol.y` und `sol.success` auslesen.
* [ ] Sie wissen, warum `f(t, y)` eine bestimmte Signatur haben muss, und
  können diese Funktion für eine skalare DGL korrekt schreiben.
* [ ] Sie können erklären, was **adaptive Schrittweite** bedeutet, und
  begründen, warum sie effizienter ist als eine feste Schrittweite.
* [ ] Sie können eine **nicht-autonome Differentialgleichung** (rechte Seite
  hängt explizit von $t$ ab) mit `solve_ivp` lösen.
```

+++

## solve_ivp für das Abkühlproblem

*Wie viel einfacher kann man eine DGL lösen?* Wir nehmen dasselbe Kühlproblem
aus Abschnitt 10.1 und lösen es mit `solve_ivp`. Dazu importieren wir zunächst
die notwendigen Module und definieren die Abkühlung als Funktion.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

# --- Dieselben Parameter wie in Abschnitt 10.1 ---
T0_wert = 80.0   # Anfangstemperatur in °C
T_inf   = 20.0   # Umgebungstemperatur in °C
k       = 0.1    # Abkühlkonstante in 1/min

# --- Rechte Seite der DGL als Python-Funktion ---
# Pflichtformat: f(t, y)  ->  t ist Skalar, y ist Array
# Auch für eine skalare DGL: y = [T_aktuell], Rückgabe = [dT/dt]
# Wichtig: t steht an erster Stelle, auch wenn die DGL autonom ist (kein t auf der rechten Seite).
def f_abkuehlung(t, y):
    T_aktuell = y[0]                      # Temperatur aus dem Zustandsvektor
    dTdt = -k * (T_aktuell - T_inf)       # Newtonsche Abkühlungsrate
    return [dTdt]                         # Rückgabe als Liste mit einem Eintrag
```

`solve_ivp` erfordert einige Argumente. Wir übergeben nicht nur die
Differentialgleichung als Funktion `f_abkuehlung`, sondern auch das
Integrationsintervall `t_span`, die Anfangsbedingung `y0` und die Zeitpunkte
`t_eval`, zu denen die Lösung der Differentialgleichung ausgewertet werden soll.

```{code-cell} python
# --- solve_ivp Aufruf ---
# fun:    rechte Seite der DGL, Signatur f(t, y)
# t_span: Integrationsintervall als Tupel (Anfang, Ende)
# y0:     Anfangsbedingung als Liste; y0 = [T0] für eine skalare DGL
# t_eval: gewünschte Ausgabezeitpunkte (intern werden andere Punkte berechnet)
t_auswertung = np.linspace(0, 50, 501)
sol = solve_ivp(fun=f_abkuehlung, t_span=(0, 50), y0=[T0_wert], t_eval=t_auswertung)
```

Nachdem die Differentialgleichung gelöst wurde, können wir die Lösung über die
Attribute einsehen.

```{code-cell} python
# --- Rückgabe auslesen ---
# sol.t:    Ausgabezeitpunkte (= t_auswertung, da t_eval angegeben)
# sol.y:    Array der Form (Anzahl_Zustände, Anzahl_Zeitpunkte)
# sol.y[0]: erster Zustand (Temperatur) über die Zeit
# sol.success: True wenn die Integration erfolgreich war
print(f"Integration erfolgreich:  {sol.success}")
print(f"sol.y.shape:              {sol.y.shape}  "
      f"--> {sol.y.shape[0]} Zustand, {sol.y.shape[1]} Zeitpunkte")
```

Als nächstes analysieren wir die Genauigkeit der numerischen Lösung und
vergleichen dazu die Temperatur nach 10 min Abkühlung mit der exakten Lösung.

```{code-cell} python
# --- Genauigkeitsvergleich bei t = 10 min ---
# t_auswertung[100] = 10.0 min (linspace(0,50,501), Abstand = 0.1 min)
T_exakt_10  = T_inf + (T0_wert - T_inf) * np.exp(-k * 10.0)
T_ivp_10    = sol.y[0, 100]

# Euler mit dt = 5 min als Referenz (zwei Schritte bis t = 10 min)
T_euler_05  = T0_wert + 5.0 * (-k * (T0_wert - T_inf))   # t = 5 min
T_euler_10  = T_euler_05 + 5.0 * (-k * (T_euler_05 - T_inf))  # t = 10 min

print(f"Temperatur bei t = 10 min:")
print(f"  Analytisch:         {T_exakt_10:.4f} °C")
print(f"  solve_ivp (RK45):   {T_ivp_10:.4f} °C  "
      f"  Fehler: {abs(T_ivp_10 - T_exakt_10):.2e} °C")
print(f"  Euler (dt = 5 min): {T_euler_10:.4f} °C  "
      f"  Fehler: {abs(T_euler_10 - T_exakt_10):.4f} °C")
```

Das Ergebnis ist eindeutig: `solve_ivp` erreicht einen Fehler von unter
0.01 °C, während Euler mit $\Delta t = 5\,\text{min}$ mehr als 7 °C daneben
liegt. Wie viele Schritte `solve_ivp` dafür intern benötigt, sehen wir im
nächsten Unterabschnitt.

```{code-cell} python
# Euler mit dt = 5 min für den Vergleichsplot
T0_wert = 80.0; T_inf = 20.0; k = 0.1
n_e = 10; dt_e = 5.0
t_euler = np.linspace(0, 50, n_e + 1)
T_euler = np.zeros(n_e + 1)
T_euler[0] = T0_wert
for i in range(n_e):
    T_euler[i+1] = T_euler[i] + dt_e * (-k * (T_euler[i] - T_inf))

# Analytische Referenz
t_eval = sol.t
T_exakt = T_inf + (T0_wert - T_inf) * np.exp(-k * t_eval)

# Visualisierung
fig, ax = plt.subplots()
ax.plot(t_eval, T_exakt, label='analytisch (Referenz)')
ax.plot(t_eval, sol.y[0], linestyle=':', label='solve_ivp (RK45)')
ax.plot(t_euler, T_euler, marker='o', label='Euler ($\\Delta t = 5$ min)')
ax.set_xlabel('Zeit in min')
ax.set_ylabel('Temperatur in °C')
ax.set_title('Vergleich: Euler vs. solve_ivp vs. analytisch')
ax.legend()
ax.grid(True)
plt.tight_layout()
plt.show()
```

Die gestrichelte Euler-Linie weicht sichtbar von der analytischen Referenz
ab; die gepunktete `solve_ivp`-Linie liegt darauf.

**Hinweis**: `solve_ivp` erwartet immer die Signatur `f(t, y)`, wobei `t` an
erster Stelle steht und `y` ein Array ist. Für eine skalare DGL ist `y` ein
Array mit einem Element. Häufige Fehler sind:

* `f(y)` ohne `t`,
* `f(y, t)` in falscher Reihenfolge, oder
* `y` wie einen Skalar behandeln ohne `y[0]` zu schreiben.

```{admonition} Mini-Übung
:class: tip
1. Was ist `sol.y.shape` in unserem Beispiel? Erklären Sie die beiden
   Dimensionen. Was würde `sol.y.shape` sein, wenn die DGL drei Zustandsgrößen
   hätte?
2. Was erwartet man für den Temperaturverlauf, wenn die Anfangstemperatur
   unter der Umgebungstemperatur liegt? Probieren Sie `y0=[5.0]` aus.
   Sagen Sie das Ergebnis zuerst voraus, bevor Sie den Code ausführen.
3. Warum gibt `f_kuehl` eine Liste `[dTdt]` zurück und keinen Skalar `dTdt`,
   obwohl die DGL nur eine Zustandsgröße hat?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: `sol.y.shape = (1, 501)`. Die erste Dimension (1) ist die Anzahl
der Zustandsgrößen (hier nur Temperatur), die zweite (501) ist die Anzahl der
Ausgabezeitpunkte. Bei drei Zustandsgrößen wäre `sol.y.shape = (3, 501)`;
der Zugriff auf den dritten Zustand erfolgte dann über `sol.y[2]`.

Zu Frage 2: Die Temperatur steigt von 5 °C auf 20 °C, weil die rechte Seite
$f = -k(T - T_\infty) = -k(5 - 20) = +1.5\,°C/\text{min}$ positiv ist.
Die DGL treibt $T$ immer in Richtung $T_\infty$, unabhängig davon ob man
von oben oder unten startet.

```python
from scipy.integrate import solve_ivp
import numpy as np

T_inf = 20.0; k = 0.1
def f_kuehlung(t, y):
    return [-k * (y[0] - T_inf)]

sol_kalt = solve_ivp(f_kuehlung, t_span=(0, 50), y0=[5.0],
                     t_eval=np.linspace(0, 50, 501))
print(f"T(0) = {sol_kalt.y[0, 0]:.1f} °C")
print(f"T(30) = {sol_kalt.y[0, 300]:.2f} °C")
```

Ausgabe:
```code
T(0) = 5.0 °C
T(30) = 19.25 °C
```

Zu Frage 3: `solve_ivp` ist für Systeme von Differentialgleichungen ausgelegt.
Es erwartet immer eine Liste oder ein Array als Rückgabe, auch wenn nur eine
Zustandsgröße vorliegt. Das ermöglicht dieselbe Schnittstelle für skalare
Differentialgleichungen und für Systeme wie in Kapitel 11.
````

+++

## Adaptive Schrittweite: wie RK45 intern arbeitet

Das Euler-Verfahren benutzt eine feste, vom Anwender gewählte Schrittweite.
RK45 geht anders vor: Es schätzt bei jedem Schritt den lokalen Fehler ab
und verkleinert oder vergrößert die Schrittweite automatisch. Wo die
Lösung sich schnell ändert, werden kleine Schritte gewählt; wo sie fast
konstant ist, können große Schritte genügen.

```{code-cell} python
# --- solve_ivp OHNE t_eval: zeigt die tatsächlich verwendeten internen Schritte ---
sol_intern = solve_ivp(f_abkuehlung, t_span=(0, 50), y0=[T0_wert])

print(f"Anzahl interner Schritte (RK45):  {len(sol_intern.t)}")
print()
print("Schrittweiten (Differenz benachbarter interner Zeitpunkte):")
dt_intern = np.diff(sol_intern.t)
for i, (ti, dti) in enumerate(zip(sol_intern.t[:-1], dt_intern)):
    print(f"  Schritt {i+1:2d}: t = {ti:5.2f} --> {sol_intern.t[i+1]:5.2f} min  "
          f"(Δt = {dti:.4f} min)")

print()
print(f"Kleinste Schrittweite:  {dt_intern.min():.4f} min")
print(f"Größte Schrittweite:    {dt_intern.max():.4f} min")
```

Zusätzlich visualisieren wir die Schrittweiten.

```{code-cell} python
# --- Visualisierung der internen Schrittweiten ---
t_fein  = np.linspace(0, 50, 500)
T_fein  = T_inf + (T0_wert - T_inf) * np.exp(-k * t_fein)

fig, ax = plt.subplots()
ax.plot(t_fein, T_fein, label='analytisch')
ax.scatter(sol_intern.t, sol_intern.y[0], color='red',
                label=f'Interne RK45-Schritte ({len(sol_intern.t)} Punkte)')
ax.set_xlabel('Zeit in min')
ax.set_ylabel('Temperatur in °C')
ax.set_title('RK45: interne Auswertungspunkte')
ax.legend()
ax.grid(True)
plt.tight_layout()
plt.show()
```

Die intern benötigten Schritte vergleichen wir mit der Anzahl an Schritten, die
das Euler-Verfahren für eine festgelegte Schrittweit braucht.

```{code-cell} python
# --- Effizienzvergleich: Anzahl Schritte vs. Fehler bei t = 10 min ---
T_exakt_10 = T_inf + (T0_wert - T_inf) * np.exp(-k * 10.0)

# Euler: verschiedene dt
print(f"{'Methode                  ':}  {'Schritte    ':}  {'Fehler bei t=10 [°C]'}")
print("-" * 62)
for dt in [5.0, 2.0, 0.5, 0.1]:
    n  = int(50.0 / dt)
    T  = T0_wert
    for _ in range(int(10.0 / dt)):
        T = T + dt * (-k * (T - T_inf))
    print(f"Euler (dt = {dt:.1f} min)       {int(50.0/dt)} \t\t {abs(T - T_exakt_10):.4f}")

# solve_ivp
sol_10 = solve_ivp(f_abkuehlung, t_span=(0, 10), y0=[T0_wert])
print(f"{'solve_ivp (RK45)':<25}  {len(sol_10.t):>8}  "
      f"{abs(sol_10.y[0, -1] - T_exakt_10):>22.2e}")
```

Schon mit 8 internen Schritten liegt `solve_ivp` näher an der analytischen
Lösung als Euler mit 500 Schritten. Der Unterschied liegt im Algorithmus:
Euler approximiert die Ableitung durch einen einzigen Wert am linken
Intervallrand. RK45 wertet die rechte Seite $f(t, y)$ innerhalb jedes
Schrittes an mehreren Zwischenpunkten aus, kombiniert diese gewichtet und
erzeugt so eine Näherung der Ordnung 4 (Fehler $O(\Delta t^4)$ statt
$O(\Delta t)$).

```{admonition} Methodenvergleich
:class: note
| Methode | Konvergenzordnung | Schrittwahl | Typischer Anwendungsfall |
|---|---|---|---|
| Euler | $O(\Delta t)$ | fest, manuell | Einsteiger, Konzeptverständnis |
| RK45 in `solve_ivp` | $O(\Delta t^4)$ | adaptiv, automatisch | Standardfall in der Praxis |

Halbiert man die Schrittweite, sinkt der Fehler bei Euler auf die Hälfte,
bei RK45 auf ein Sechzehntel.
```

```{admonition} Mini-Übung
:class: tip
1. Warum ist die erste Schrittweite von RK45 viel kleiner als die letzte?
   Begründen Sie in einem Satz, ohne Code auszuführen.
2. Würden Sie für $k = 1.0\,\text{min}^{-1}$ (zehnfach schnellere Abkühlung)
   mehr oder weniger interne RK45-Schritte erwarten? Testen Sie Ihre Vermutung.
3. Euler benötigt $\Delta t = 0.1\,\text{min}$ für einen Fehler unter
   $0.1\,°C$ bei $t = 10\,\text{min}$ (500 Schritte bis $t = 50$). `solve_ivp`
   erreicht denselben Bereich mit 8 Schritten. Um welchen Faktor ist das
   effizienter?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: Am Anfang ($t = 0$) ist die Abkühlrate $-k(T_0 - T_\infty) =
-6\,°C/\text{min}$ am größten; die Lösung ändert sich schnell. RK45 wählt
deshalb einen vorsichtigen ersten Schritt. Am Ende nähert sich $T$ der
Umgebungstemperatur, die Rate geht gegen null, und der Solver kann sichere
große Schritte machen.

```python
from scipy.integrate import solve_ivp

T0_wert = 80.0; T_inf = 20.0
k_schnell = 1.0   # zehnfach schnellere Abkühlung

def f_schnell(t, y):
    return [-k_schnell * (y[0] - T_inf)]

sol_schnell = solve_ivp(f_schnell, t_span=(0, 50), y0=[T0_wert])
print(f"k = 0.1: {len(solve_ivp(lambda t,y: [-0.1*(y[0]-20)], (0,50), [80]).t)} Schritte")
print(f"k = 1.0: {len(sol_schnell.t)} Schritte")
```

Für $k = 1.0$ sind mehr interne Schritte nötig, weil die Lösung am Anfang
wesentlich steiler abfällt. Der Solver muss die anfangs hohe Änderungsrate
mit kleinen Schritten auflösen, bevor er auf größere Schritte umstellen kann.

Zu Frage 3: 500 Euler-Schritte gegenüber 8 RK45-Schritten ergibt einen
Faktor von $500 / 8 = 62.5$. Da RK45 pro Schritt etwa sechs
Funktionsauswertungen benötigt, ist der Faktor bei den tatsächlichen
Funktionsauswertungen etwa $500 / 48 \approx 10$. Dennoch: für dieselbe
Genauigkeit braucht RK45 einen Bruchteil des Rechenaufwands von Euler.
````

+++

## Nicht-autonome Differentialgleichungen

Bisher hing die rechte Seite $f(T)$ nur vom aktuellen Zustand ab, nicht
explizit von der Zeit. Solche DGLen nennt man **autonom**. Wenn die rechte
Seite $f(t, T)$ auch die Zeit explizit enthält, spricht man von einer
**nicht-autonomen** DGL.

Ein konkretes Beispiel: Ein Metallstab liegt morgens draußen. Die
Außentemperatur steigt linear von $T_{\infty,0} = 10\,°C$ auf
$T_{\infty,0} + r \cdot t_\text{end} = 35\,°C$ innerhalb von 50 Minuten.
Der Stab hatte anfangs noch $T_0 = 80\,°C$ (vom Aufheizen über Nacht).

$$\dot{T}(t) = -k\bigl(T(t) - \underbrace{(T_{\infty,0} + r\,t)}_{T_\infty(t)}\bigr).$$

Die rechte Seite hängt nun explizit von $t$ ab: dasselbe $T$ führt bei
verschiedenen Zeiten zu verschiedenen Ableitungen.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

# --- Parameter ---
T0_wert  = 80.0   # Anfangstemperatur Metallstab in °C
T_inf0   = 10.0   # Außentemperatur bei t = 0 in °C
r_rampe  = 0.5    # Erwärmungsrate der Umgebung in °C/min
k        = 0.1    # Abkühlkonstante in 1/min
t_end    = 50.0

# --- Rechte Seite: nicht-autonom, da f explizit von t abhängt ---
# T_aussen(t) = T_inf0 + r_rampe * t  (linearer Anstieg)
def f_kuehl_rampe(t, y):
    T_aussen = T_inf0 + r_rampe * t       # Außentemperatur zum Zeitpunkt t
    dTdt = -k * (y[0] - T_aussen)         # Newtonsche Abkühlungsrate
    return [dTdt]

# --- solve_ivp löst nicht-autonome DGL genauso wie autonome ---
t_eval = np.linspace(0, t_end, 501)
sol_rampe = solve_ivp(fun=f_kuehl_rampe, t_span=(0, t_end),
                      y0=[T0_wert], t_eval=t_eval)

# --- Analytische Lösung als Probe ---
# T(t) = T_inf0 + r*t - r/k + (T0 - T_inf0 + r/k) * exp(-k*t)
# Die Lösung folgt der steigenden Außentemperatur mit konstantem Rückstand r/k.
T_ana = T_inf0 + r_rampe*t_eval - r_rampe/k + (T0_wert - T_inf0 + r_rampe/k) * np.exp(-k*t_eval)
T_aussen_verlauf = T_inf0 + r_rampe * t_eval   # zeitlicher Verlauf der Außentemperatur

print(f"Außentemperatur: {T_inf0:.1f} °C → {T_aussen_verlauf[-1]:.1f} °C über {t_end:.0f} min")
print(f"Stabtemperatur bei t = {t_end:.0f} min:   {sol_rampe.y[0, -1]:.2f} °C")
print(f"Analytisch:                    {T_ana[-1]:.2f} °C")
print(f"Rückstand (r/k = {r_rampe}/{k}):  "
      f"{T_aussen_verlauf[-1] - sol_rampe.y[0, -1]:.2f} °C  "
      f"(theoretisch: {r_rampe/k:.1f} °C)")

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(t_eval, T_aussen_verlauf, color='#E87846', linewidth=2, linestyle='--',
        label='Außentemperatur $T_\\infty(t)$')
ax.plot(t_eval, T_ana,            color='#005A94', linewidth=2,
        label='Analytisch (Probe)')
ax.plot(t_eval, sol_rampe.y[0],   color='#E60000', linewidth=1.5, linestyle=':',
        label='solve\\_ivp (RK45)')
ax.annotate('', xy=(50, T_aussen_verlauf[-1]),
            xytext=(50, sol_rampe.y[0, -1]),
            arrowprops=dict(arrowstyle='<->', color='#484949', lw=1.5))
ax.text(51, 0.5*(T_aussen_verlauf[-1] + sol_rampe.y[0,-1]),
        f'Rückstand\n≈ {r_rampe/k:.0f} °C',
        color='#484949', fontsize=9, va='center')
ax.set_xlabel('Zeit in min')
ax.set_ylabel('Temperatur in °C')
ax.set_title('Nicht-autonome DGL: Stab folgt steigender Außentemperatur')
ax.legend()
ax.grid(True)
plt.tight_layout()
plt.show()
```

Im eingeschwungenen Zustand folgt der Stab der Außentemperatur mit einem
konstanten Rückstand von $r/k = 0.5 / 0.1 = 5\,°C$. Das ist ein allgemeines
Ergebnis: Bei einer linear ansteigenden Außentemperatur liegt die
Stabtemperatur immer um $r/k$ dahinter. Mit einem kleinen $k$ (schlechte
Wärmeleitung) oder einem schnellen Anstieg $r$ vergrößert sich der Rückstand.

```{admonition} Mini-Übung
:class: tip
1. Lesen Sie aus dem Plot ab: Wie groß ist der Rückstand des Stabs gegenüber
   der Außentemperatur am Ende (bei $t = 50$ min)? Stimmt er mit dem
   theoretischen Wert $r/k$ überein, und warum weicht er leicht ab?
2. Für eine autonome DGL reichte die Funktion `f_abkuehlung(T)` aus
   Abschnitt 10.1. Warum lässt sich `f_kuehl_rampe` nicht durch
   `f_abkuehlung` ersetzen, ohne den Code anzupassen?
3. Was würde passieren, wenn man die Aufwärmrate auf $r = 2.0\,°C/\text{min}$
   erhöht? Schätzen Sie den theoretischen Rückstand und überprüfen Sie mit
   `solve_ivp`.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
Zu Frage 1: Der Rückstand beträgt laut Ausgabe etwa 4.49 °C, der theoretische
Wert $r/k = 5.0\,°C$. Die Abweichung kommt daher, dass bei $t = 50\,\text{min}$
der anfängliche Einschwingvorgang noch nicht vollständig abgeklungen ist. Der
Stab startete bei $80\,°C$ (weit oberhalb der Außentemperatur), und die
Anfangsbedingung beeinflusst die Lösung noch für endliche Zeiten. Der
stationäre Rückstand $r/k$ gilt exakt erst für $t \to \infty$.

Zu Frage 2: `f_abkuehlung(T)` nimmt nur den aktuellen Temperaturwert als
Argument. Die Außentemperatur $T_\infty(t)$ ist darin fest auf 20 °C
einprogrammiert. Die nicht-autonome DGL braucht jedoch $t$ als Eingabe,
um $T_\infty(t) = T_{\infty,0} + r\,t$ berechnen zu können. `solve_ivp`
übergibt bei jedem Auswertungsaufruf den aktuellen Zeitpunkt an $f(t, y)$;
dieses $t$ steht in `f_abkuehlung` schlicht nicht zur Verfügung.

```python
import numpy as np
from scipy.integrate import solve_ivp

T_inf0 = 10.0; r_neu = 2.0; k = 0.1; T0_wert = 80.0

def f_rampe_neu(t, y):
    T_aussen = T_inf0 + r_neu * t
    return [-k * (y[0] - T_aussen)]

sol_neu = solve_ivp(f_rampe_neu, t_span=(0, 50), y0=[T0_wert],
                    t_eval=np.linspace(0, 50, 501))
T_aussen_50 = T_inf0 + r_neu * 50
rueckstand  = T_aussen_50 - sol_neu.y[0, -1]

print(f"Außentemperatur bei t=50: {T_aussen_50:.1f} °C")
print(f"Stabtemperatur bei t=50:  {sol_neu.y[0,-1]:.2f} °C")
print(f"Tatsächlicher Rückstand:  {rueckstand:.2f} °C")
print(f"Theoretisch (r/k):        {r_neu/k:.1f} °C")
```

Der theoretische Rückstand beträgt $r/k = 2.0 / 0.1 = 20\,°C$. Das ist ein
deutlicher Rückstand: Die Außentemperatur steigt so schnell, dass der Stab
weit hinterherhinkt.
````

+++

## Zusammenfassung und Ausblick

`scipy.integrate.solve_ivp` löst gewöhnliche Differentialgleichungen
numerisch mit dem adaptiven RK45-Verfahren. Es braucht nur die Signatur
`f(t, y)`, das Integrationsintervall `t_span` und die Anfangsbedingung
`y0` als Liste. Mit 8 internen Schritten erreicht RK45 für das
Kühlproblem eine Genauigkeit, die Euler mit 500 Schritten nicht schafft.
Nicht-autonome DGLen, bei denen die rechte Seite explizit von $t$ abhängt,
werden ohne Anpassung des Solvers gelöst, weil `f(t, y)` den Zeitpunkt
immer als Argument erhält.

In Abschnitt 10.4 vertiefen wir `solve_ivp` durch Präsenzübungen. In
Kapitel 11 werden wir DGLen zweiter Ordnung (zum Beispiel Schwingungsgleichungen)
auf Systeme erster Ordnung zurückführen und mit `solve_ivp` lösen. Dafür
wird der Zustandsvektor `y` dann zwei Einträge haben: Position und
Geschwindigkeit.
