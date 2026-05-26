---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 9.3 Simpson-Regel und scipy.integrate.quad

In Abschnitt 9.1 haben wir die Trapezregel kennengelernt: lineare
Interpolation zwischen Messpunkten. *Geht es noch genauer?* Ja, wenn wir
statt einer Geraden eine Parabel durch je drei aufeinanderfolgende Punkte
legen. Das ist die Idee der Simpson-Regel. Außerdem lernen wir heute
`scipy.integrate.quad` kennen, das Werkzeug der Wahl, wenn nicht
Messdaten, sondern ein analytisches Modell vorliegt.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können die **Simpson-Regel** als Formel aufschreiben und als
  Python-Funktion implementieren.
* [ ] Sie wissen, warum die Simpson-Regel eine ungerade Punktanzahl
  erfordert, und können die Fehlerordnungen $O(\Delta t^2)$
  (Trapezregel) und $O(\Delta t^4)$ (Simpson) benennen.
* [ ] Sie wissen, wann Simpson der Trapezregel vorzuziehen ist und wann
  nicht.
* [ ] Sie können `scipy.integrate.quad` auf eine analytische Funktion
  anwenden und den zurückgegebenen Fehlerabschätzungswert interpretieren.
* [ ] Sie können `scipy.integrate.quad` für ein uneigentliches Integral
  mit unendlicher oberer Grenze einsetzen.
```

+++

## Die Simpson-Regel: Parabolapproximation statt Geraden

Die Trapezregel verbindet je zwei benachbarte Messpunkte durch eine
Gerade. Die **Simpson-Regel** legt stattdessen eine Parabel durch je
drei aufeinanderfolgende Punkte und integriert diese analytisch. Das
Ergebnis für ein Doppelintervall $[t_i, t_{i+2}]$ lautet:

$$\Delta s_i = \frac{\Delta t}{3}\bigl(v_i + 4\,v_{i+1} + v_{i+2}\bigr).$$

Der mittlere Punkt geht mit dem vierfachen Gewicht ein. Summiert man
über alle Doppelintervalle, erhält man die zusammengesetzte
Simpson-Regel:

$$s \approx \frac{\Delta t}{3}\bigl(v_0 + 4v_1 + 2v_2 + 4v_3 + \cdots
+ 2v_{n-3} + 4v_{n-2} + v_{n-1}\bigr).$$

Weil immer drei Punkte ein Doppelintervall bilden, muss die Anzahl der
Intervalle gerade sein, also $n$ (Anzahl Punkte) ungerade.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

# --- Bremsdaten aus Abschnitt 9.2 (gleicher Seed = identische Werte) ---
np.random.seed(12)

v0_bremsung = 100 / 3.6   # Anfangsgeschwindigkeit in m/s
k_bremsung  = 0.35        # Abklingkonstante in 1/s
T_bremsung  = 8.0         # Messdauer in s
dt_sensor   = 0.5         # Abtastrate in s

t_mess  = np.arange(0, T_bremsung + dt_sensor, dt_sensor)
v_exakt = v0_bremsung * np.exp(-k_bremsung * t_mess)
v_mess  = np.maximum(v_exakt + np.random.normal(0, 0.3, size=len(t_mess)), 0.0)

# np.arange(0, 8.5, 0.5) ergibt 17 Punkte: ungerade, gut für Simpson
print(f"Anzahl Messpunkte: {len(t_mess)}  (ungerade: {len(t_mess) % 2 == 1})")
```

```{code-cell} python
import numpy as np

def simpson(t, v):
    """Zusammengesetzte Simpson-Regel für gleichmäßiges Raster.

    Parameters
    ----------
    t : np.ndarray
        Zeitpunkte mit konstantem Abstand (gleichmäßiges Raster),
        ungerade Anzahl erforderlich. Diese Funktion setzt ein
        gleichmäßiges Raster voraus, überprüft dies aber nicht zur
        Laufzeit.
    v : np.ndarray
        Funktionswerte an den Zeitpunkten.

    Returns
    -------
    float
        Näherungswert des Integrals.
    """
    n  = len(v)
    dt = t[1] - t[0]

    # Simpson erfordert ungerade Punktanzahl (gerade Anzahl von Intervallen)
    if n % 2 == 0:
        raise ValueError(
            f"Simpson-Regel erfordert ungerade Punktanzahl, erhalten: {n}"
        )

    # Gewichtsvektor aufbauen: 1, 4, 2, 4, 2, ..., 4, 1
    gewichte         = np.ones(n)
    gewichte[1:-1:2] = 4   # ungerade Indizes (1, 3, 5, ...): Gewicht 4
    gewichte[2:-2:2] = 2   # gerade Indizes innen (2, 4, 6, ...): Gewicht 2

    return (dt / 3) * np.dot(gewichte, v)

# --- Vergleich auf den Bremsdaten ---
bremsweg_exakt   = (v0_bremsung / k_bremsung) * (1 - np.exp(-k_bremsung * T_bremsung))
bremsweg_trapez  = np.trapezoid(v_mess, t_mess)
bremsweg_simpson = simpson(t_mess, v_mess)

print(f"Exakter Bremsweg: {bremsweg_exakt:.4f} m")
print()
print(f"Trapezregel:   {bremsweg_trapez:.4f} m  "
      f"(Fehler: {bremsweg_trapez - bremsweg_exakt:+.4f} m)")
print(f"Simpson-Regel: {bremsweg_simpson:.4f} m  "
      f"(Fehler: {bremsweg_simpson - bremsweg_exakt:+.4f} m)")
```

Beide Methoden liefern hier sehr ähnliche Ergebnisse. Der Grund: bei
verrauschten Messdaten dominiert das Rauschen den Gesamtfehler, nicht
die Integrationsmethode. Wie viel genauer Simpson bei rauschfreien
Funktionen ist, zeigt der nächste Unterabschnitt.

```{admonition} Mini-Übung
:class: tip
1. Warum erfordert die Simpson-Regel eine ungerade Punktanzahl?
   Begründen Sie anhand der Formel in einem Satz, ohne Code.
2. Berechnen Sie den Bremsweg für die drei Punkte
   $t = [0, 1, 2]$ s, $v = [20, 15, 5]$ m/s mit der Simpson-Regel
   per Handrechnung. Überprüfen Sie das Ergebnis mit `simpson()`.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

t = np.array([0.0, 1.0, 2.0])
v = np.array([20.0, 15.0, 5.0])

# Handrechnung: dt = 1, Gewichte = [1, 4, 1]
# s = (1/3) * (1*20 + 4*15 + 1*5) = (1/3) * 85 ≈ 28.33 m
s_hand    = (1.0 / 3) * (1 * 20 + 4 * 15 + 1 * 5)
s_simpson = simpson(t, v)

print(f"Handrechnung:   {s_hand:.4f} m")
print(f"simpson():      {s_simpson:.4f} m")
print(f"np.trapezoid:   {np.trapezoid(v, t):.4f} m")
```

Zu Frage 1: Die Formel fasst immer drei Punkte zu einem Doppelintervall
zusammen. Bei gerader Punktanzahl bleibt am Ende ein einzelner Punkt
übrig, der kein vollständiges Doppelintervall bilden kann.

Simpson liefert $85/3 \approx 28{,}33$ m, `np.trapezoid` liefert
$27{,}5$ m. Bei diesem kleinen Datensatz ist der Unterschied gering;
bei glatten Funktionen und mehr Punkten wächst der Vorteil von Simpson
deutlich.
````

+++

## Fehlerordnung und Methodenwahl

Die Trapezregel hat Fehlerordnung $O(\Delta t^2)$, die Simpson-Regel
erreicht $O(\Delta t^4)$. Das bedeutet: halbiert man die Schrittweite,
wird der Fehler der Trapezregel ungefähr durch vier geteilt, der der
Simpson-Regel ungefähr durch sechzehn.

```{admonition} Fehlerordnungen im Überblick
:class: note
| Methode | Fehlerordnung | Halbierung von $\Delta t$ bewirkt |
|---|---|---|
| Trapezregel | $O(\Delta t^2)$ | Fehler sinkt auf ein Viertel |
| Simpson-Regel | $O(\Delta t^4)$ | Fehler sinkt auf ein Sechzehntel |

Dieselbe Logik wie in Kapitel 7: dort $O(\Delta x^2)$ für den zentralen
Differenzenquotienten, hier $O(\Delta t^4)$ für Simpson.
```

Den Vorteil von Simpson sieht man nur bei rauschfreien, glatten
Funktionen. Wir demonstrieren das kurz am analytischen Bremsmodell ohne
Rauschen:

```{code-cell} python
import numpy as np

# --- Fehlervergleich auf rauschfreier Testfunktion ---
v0_test = 100 / 3.6
k_test  = 0.35
T_test  = 8.0

# Exaktes Integral als Referenz
integral_exakt = (v0_test / k_test) * (1 - np.exp(-k_test * T_test))

# Zwei Auflösungen zum Vergleich: grob (n=9) und fein (n=17)
for n in [9, 17]:
    t = np.linspace(0, T_test, n)
    v = v0_test * np.exp(-k_test * t)   # kein Rauschen
    dt = T_test / (n - 1)

    fehler_t = abs(np.trapezoid(v, t) - integral_exakt)
    fehler_s = abs(simpson(t, v)      - integral_exakt)

    print(f"n = {n:2d}  (dt = {dt:.3f} s):  "
          f"Trapez = {fehler_t:.2e} m,  "
          f"Simpson = {fehler_s:.2e} m,  "
          f"Faktor = {fehler_t/fehler_s:.0f}x")
```

Bei rauschfreien Daten ist Simpson hier um einen Faktor 10 bis 100 genauer. Bei
den verrauschten Sensordaten aus der Übung war der Unterschied dagegen
vernachlässigbar, weil das Rauschen den Integrationsfehler überwog.

**Faustregel:** Simpson für glatte analytische Funktionen, Trapezregel
für verrauschte Messdaten.

+++

## scipy.integrate.quad für analytische Modelle

Wenn das analytische Modell bekannt ist, muss man es nicht erst auf
einem Gitter auswerten. `scipy.integrate.quad` integriert eine
Python-Funktion direkt und schätzt die Genauigkeit automatisch ab.

```{code-cell} python
from scipy.integrate import quad
import numpy as np

# --- Analytisches Bremsmodell als Python-Funktion ---
v0_modell = 100 / 3.6   # Anfangsgeschwindigkeit in m/s
k_modell  = 0.35        # Abklingkonstante in 1/s

def v_brems(t):
    """Analytisches Bremsmodell: v(t) = v0 * exp(-k*t)."""
    return v0_modell * np.exp(-k_modell * t)

# --- quad(funktion, untere_grenze, obere_grenze) ---
# Rückgabe: (Integralwert, geschätzter absoluter Fehler)
bremsweg_quad, fehlerabschaetzung = quad(v_brems, 0, 8.0)

# Analytischer Referenzwert: Stammfunktion von v0*exp(-k*t) ausgewertet von 0 bis T
bremsweg_exakt = (v0_modell / k_modell) * (1 - np.exp(-k_modell * 8.0))

print(f"scipy.integrate.quad:  {bremsweg_quad:.6f} m")
print(f"Analytisch exakt:      {bremsweg_exakt:.6f} m")
print(f"Fehlerabschätzung:     {fehlerabschaetzung:.2e} m")
print(f"Tatsächlicher Fehler:  {abs(bremsweg_quad - bremsweg_exakt):.2e} m")
```

`quad` gibt zwei Werte zurück: den Integralwert und eine Schätzung des
absoluten Fehlers. Die Fehlerabschätzung liegt hier im Bereich der
Maschinengenauigkeit, weil die Funktion glatt und gut konditioniert ist.

`quad` kann auch **uneigentliche Integrale** mit unendlicher Grenze
berechnen. Dazu übergibt man `np.inf`:

```{code-cell} python
from scipy.integrate import quad
import numpy as np

v0_modell = 100 / 3.6
k_modell  = 0.35

# --- Bremsweg bis zum vollständigen Stillstand: obere Grenze = unendlich ---
# Analytisch: Integral von v0*exp(-k*t) von 0 bis inf = v0/k
bremsweg_inf, fehler_inf = quad(v_brems, 0, np.inf)

print(f"Bremsweg bis Stillstand (quad):  {bremsweg_inf:.4f} m")
print(f"Analytisch (v0/k):               {v0_modell / k_modell:.4f} m")
print(f"Bremsweg bis T = 8 s:            {bremsweg_exakt:.4f} m")
print(f"Anteil nach T = 8 s:             "
      f"{(bremsweg_inf - bremsweg_exakt) / bremsweg_inf * 100:.1f} %")
```

Das Integral bis Unendlich gibt den theoretischen Gesamtbremsweg an,
wenn das Fahrzeug dem Exponentialmodell folgend nie vollständig zum
Stillstand kommt. Bei $T = 8$ s sind bereits 93{,}9 % dieses Weges
zurückgelegt.

```{admonition} Wann welches Werkzeug?
:class: note
| Methode | Eingabe | Anwendungsfall |
|---|---|---|
| `np.trapezoid` | Messdaten `v[i]` | Sensordaten, diskrete Messung |
| `simpson()` | Messdaten `v[i]`, $n$ ungerade | Glatte Daten ohne Rauschen |
| `quad(f, a, b)` | Funktion `f(t)` | Analytisches Modell bekannt |
```

```{admonition} Mini-Übung
:class: tip
1. Berechnen Sie $\int_0^\infty e^{-t}\,\mathrm{d}t$ mit `quad` und
   `np.inf`. Welchen Wert erwarten Sie analytisch?
2. Das Ergebnis von `quad` enthält immer eine Fehlerabschätzung.
   Wofür ist dieser Wert praktisch nützlich? Nennen Sie ein Szenario,
   in dem eine große Fehlerabschätzung ein Warnsignal wäre.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
from scipy.integrate import quad
import numpy as np

# Teilaufgabe 1: uneigentliches Integral
# Analytisch: Stammfunktion -e^{-t}, von 0 bis inf: 0 - (-1) = 1
ergebnis, fehler = quad(lambda t: np.exp(-t), 0, np.inf)
print(f"Integral e^(-t) von 0 bis inf: {ergebnis:.6f}")
print(f"Fehlerabschätzung:             {fehler:.2e}")
```

Das Integral ergibt exakt 1, wie analytisch erwartet.

Zu Frage 2: Eine große Fehlerabschätzung signalisiert, dass `quad`
intern viele Auswertungen brauchte und trotzdem keine hohe Genauigkeit
erreichte. Das passiert zum Beispiel bei Funktionen mit Sprungstellen,
sehr starken Oszillationen oder Singularitäten im Integrationsbereich.
In solchen Fällen sollte man das Integral aufteilen oder eine
spezialisierte Methode verwenden.
````

+++

## Zusammenfassung und Ausblick

Die Simpson-Regel erreicht durch quadratische Interpolation die
Fehlerordnung $O(\Delta t^4)$ und ist bei glatten Funktionen der
Trapezregel deutlich überlegen. Bei verrauschten Messdaten dominiert
das Rauschen und die Trapezregel genügt. `scipy.integrate.quad`
integriert eine analytisch gegebene Funktion adaptiv und ist das
Werkzeug der Wahl, wenn das Modell bekannt ist.

In Abschnitt 9.4 wenden wir Trapezregel und `quad` auf ein neues
Problem an: die geleistete Arbeit entlang einer gemessenen nichtlinearen
Federkennlinie.
