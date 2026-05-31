---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 11.3 Gedämpfte und erzwungene Schwingung

<!-- TODO: Motivationsabsatz (~100 Wörter)
     Brücke zu 11.1: Der ungedämpfte Schwinger schwingt ewig (Ellipse im Phasenporträt).
     Eine reale Fahrzeugfederung hat aber einen Stoßdämpfer: die Energie wird
     dissipiert. Zu wenig Dämpfung → Passagier schaukelt lange nach;
     zu viel Dämpfung → Feder wirkt kaum. Zusätzlich: Fahrbahnunebenheiten als
     periodische externe Kraft. Welche Kombination aus k und c ist optimal?
     Zentrales Modell: m*x_ddot + c*x_dot + k*x = F0*sin(Omega*t). -->

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie kennen die **Dämpfungszahl** $D = c\,/\,(2\sqrt{km})$ und können
  die drei Fälle $D < 1$ (schwingend), $D = 1$ (aperiodischer Grenzfall)
  und $D > 1$ (Kriechfall) im Zeitsignal $x(t)$ unterscheiden.
* [ ] Sie können erklären, warum $D = 1$ die kürzeste Einschwingzeit ohne
  Überschwingen liefert.
* [ ] Sie können eine erzwungene Schwingung mit `solve_ivp` lösen und die
  **stationäre Amplitude** aus dem eingeschwungenen Zustand als `max(|x|)`
  im letzten Zeitdrittel ablesen.
* [ ] Sie erkennen **Resonanz** als das Amplitudenmaximum bei $\Omega \approx \omega_0$
  und können die Resonanzfrequenz numerisch bestimmen.
```

+++

## Gedämpfter Schwinger: die drei Fälle

<!-- TODO: Unterabschnitt 1 (~10 min)
     - Gedämpfte DGL: m*x_ddot + c*x_dot + k*x = 0
     - Substitution wie in 11.1, aber f gibt jetzt [x_dot, (-k*x - c*x_dot)/m] zurück
     - Dämpfungszahl D = c/(2*sqrt(k*m)) einführen, kritische Dämpfung c_krit = 2*sqrt(k*m)
     - Kurvenschar x(t) für D in [0.2, 0.5, 1.0, 2.0], gleiche Anfangsbedingung x0=0.05 m
     - Phasenporträt: Spiralen statt Ellipsen
     - Rhetorische Frage kursiv: Welches D würden Sie für Ihren Pkw wählen? -->

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

# --- Parameter ---
m       = 250.0
k_feder = 16000.0
omega_0 = np.sqrt(k_feder / m)    # = 8.0 rad/s
c_krit  = 2 * np.sqrt(k_feder * m)   # kritische Dämpfung in N*s/m
print(f"omega_0 = {omega_0:.2f} rad/s")
print(f"c_krit  = {c_krit:.1f} N*s/m")

# --- Rechte Seite: gedämpfter Schwinger ---
def f_gedaempft(t, y, c):
    # y[0] = x (Position), y[1] = x_dot (Geschwindigkeit)
    # m*x_ddot + c*x_dot + k*x = 0
    # TODO: x_ddot = (-k*y[0] - c*y[1]) / m
    x_dot  = y[1]
    x_ddot = None   # TODO
    return [x_dot, x_ddot]

# TODO: Kurvenschar für D in [0.2, 0.5, 1.0, 2.0] plotten
# Hinweis: c = D * c_krit, dann solve_ivp mit lambda t, y: f_gedaempft(t, y, c)
# TODO: Phasenporträt (sol.y[0] vs. sol.y[1]) für alle vier D-Werte
```

<!-- TODO: Prosatext (~80 Wörter):
     Erklärung der drei Fälle anhand der Plot-Ausgabe.
     Formel für gedämpfte Eigenfrequenz: omega_d = omega_0 * sqrt(1 - D^2)
     (nur für D < 1 reell). Warum D=1 die kürzeste Einschwingzeit hat.
     Hinweis auf das Phasenporträt: einwärts spiralisierend statt Ellipse. -->

```{admonition} Mini-Übung
:class: tip
<!-- TODO: Mini-Übung (5 min):
     1. Berechnen Sie c für D = 0.3 von Hand (c = D * c_krit).
     2. Verständnisfrage: Für D = 1.5 schwingt das System nicht. Wie erkennen
        Sie das im Phasenporträt?
     3. Nach wie vielen Sekunden ist |x| < 0.001 m für D = 0.2 vs. D = 1.0?
        Schätzen Sie aus dem Plot ab. -->
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
<!-- TODO: Lösung zur Mini-Übung.
     c_krit = 2*sqrt(16000*250) = 4000 N*s/m
     D=0.3: c = 0.3 * 4000 = 1200 N*s/m
     Phasenporträt D>1: monoton in Ursprung, keine Spirale (kein Vorzeichen-
     wechsel von x_dot). -->
````

+++

## Erzwungene Schwingung und Resonanz

<!-- TODO: Unterabschnitt 2 (~10 min)
     - Erzwungene DGL: m*x_ddot + c*x_dot + k*x = F0*sin(Omega*t)
     - f gibt jetzt [x_dot, (-k*x - c*x_dot + F0*sin(Omega*t))/m] zurück
     - Nichtautonom (Omega*t in f → t wird gebraucht)
     - Zwei Phasen: Einschwingvorgang (Transient) + stationärer Zustand
     - Stationäre Amplitude: A = max(|x|) im letzten Drittel der Simulation
     - Resonanz bei Omega ≈ omega_0: Amplitudenmaximum zeigen
     - Parameter: F0 = 500 N, D = 0.2, Omega variabel -->

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
from scipy.integrate import solve_ivp
style.use('seaborn-v0_8')

m       = 250.0
k_feder = 16000.0
omega_0 = np.sqrt(k_feder / m)
c_krit  = 2 * np.sqrt(k_feder * m)
D       = 0.2
c       = D * c_krit
F0      = 500.0    # Kraftamplitude in N

# --- Erzwungener Schwinger (nichtautonom) ---
def f_erzwungen(t, y, Omega):
    # m*x_ddot + c*x_dot + k*x = F0*sin(Omega*t)
    x_dot  = y[1]
    x_ddot = None   # TODO: (-k*y[0] - c*y[1] + F0*np.sin(Omega*t)) / m
    return [x_dot, x_ddot]

# TODO: Simulation für Omega = omega_0 (Resonanz), t_end = 30 s
# TODO: stationäre Amplitude A = np.max(np.abs(sol.y[0, -n:])) für letztes Drittel
# TODO: Plot x(t) mit markiertem Transient-Bereich und stationärem Bereich
```

<!-- TODO: Prosatext (~80 Wörter):
     Erklärung Transient vs. stationärer Zustand.
     Bei Resonanz Omega = omega_0: maximale stationäre Amplitude, Energie-
     übertrag optimal. Formel für stationäre Amplitude (zur Verifikation):
     A_stat = F0 / (k * sqrt((1-r^2)^2 + (2*D*r)^2)) mit r = Omega/omega_0.
     Verweis auf Aufgabe 11.4 (Resonanzkurve als Schleife). -->

```{admonition} Mini-Übung
:class: tip
<!-- TODO: Mini-Übung (5 min):
     1. Ist f_erzwungen autonom oder nichtautonom? Woran erkennen Sie das?
     2. Warum muss t_end für die erzwungene Schwingung länger gewählt werden
        als für die freie Schwingung?
     3. Verständnisfrage: Was passiert bei D → 0 und Omega = omega_0 mit der
        stationären Amplitude? -->
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

## Methodenübersicht Kapitel 10–11

<!-- TODO: Unterabschnitt 3 (~8 min)
     Zusammenfassungstabelle: Wann Euler / solve_ivp, 1 Zustand / 2 Zustände / nichtautonom.
     Tabelle mit Spalten: Problem | DGL-Typ | Zustandsvektor | f-Signatur | Empfehlung

     Zeilen:
     - Abkühlung/RC/Zerfall: 1. Ordnung, y=[T], f(t,y)=[...], solve_ivp
     - Federschwinger: 2. Ordnung autonom, y=[x,v], f(t,y)=[v,...], solve_ivp
     - Erzwungene Schwingung: 2. Ordnung nichtautonom, y=[x,v], f(t,y) nutzt t, solve_ivp
     - Einfache Schätzaufgaben: Euler als Lehrbeispiel (sec01/02 Kap. 10)

     Kurzer Ausblick: Was kommt nach DGL? Systeme höherer Ordnung (z.B. 2 gekoppelte
     Massen), partielle DGLen (FEM). -->

```{admonition} Methodenübersicht: DGLen in Kapitel 10 und 11
:class: note
<!-- TODO: Tabelle einfügen:
| Problem | Zustandsvektor | `y0` | `sol.y` |
|---|---|---|---|
| Abkühlung, RC-Laden, Zerfall | $y = [T]$ | `[T0]` | `sol.y[0]`: Zustand |
| Federschwinger (frei) | $y = [x, \dot{x}]$ | `[x0, v0]` | `sol.y[0]`: Pos., `sol.y[1]`: Geschw. |
| Gedämpfter Schwinger | $y = [x, \dot{x}]$ | `[x0, v0]` | wie oben |
| Erzwungene Schwingung | $y = [x, \dot{x}]$ | `[x0, v0]` | wie oben, `f` nutzt `t` |
-->
```

+++

## Zusammenfassung und Ausblick

<!-- TODO: Zusammenfassung (~80 Wörter):
     - Dämpfungszahl D steuert qualitatives Verhalten: schwingend / Grenzfall / kriechend
     - Erzwungene Schwingung: nichtautonom, stationäre Amplitude aus letztem Zeitdrittel
     - Resonanz: maximale Amplitude bei Omega ≈ omega_0
     - Methodenübersicht: solve_ivp mit y0=[x0, v0] funktioniert für alle Fälle
     - Ausblick auf sec04 (Resonanzkurve numerisch), sec05 (Parameterstudie Fahrzeugfederung) -->
