---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 9.5 Übungen

Diese Aufgaben sind für das Selbststudium zuhause gedacht und wiederholen den
Stoff der Kapitel 9.1 bis 9.4. Rechnen Sie mit rund 90 Minuten
Bearbeitungszeit. Für die Handrechnungen genügt ein Taschenrechner. Führen Sie
vor den Aufgaben die folgende Zelle aus. Sie lädt NumPy und die Funktionen
`simpson()` und `quad()` aus SciPy. Mehrere Aufgaben verwenden dieselben Namen,
zum Beispiel `leistung`. Wenn Sie zu einer früheren Aufgabe zurückkehren,
führen Sie deren Zellen deshalb noch einmal der Reihe nach aus.

Der Schwierigkeitsgrad steht im Titel jeder Aufgabe:

* ✩ Verständnis: Code und Ausgaben vorhersagen und erklären (ca. 5 min)
* ✩✩ Anwendung: eigenen Code schreiben und Ergebnisse interpretieren (ca. 10 min)
* ✩✩✩ Mini-Projekt: mehrere Konzepte des Parts kombinieren (ca. 30 min)

Die Zeiten sind reine Bearbeitungszeiten. Dazu kommt Zeit zum Lesen, zum
Prüfen der Ergebnisse und für die Fehlersuche.

```{code-cell} python
import numpy as np
from scipy.integrate import simpson, quad
```

````{admonition} Aufgabe 9.1 (✩)
:class: tip
In Kapitel 7.1 hat das FTF seine Position jede halbe Sekunde gemeldet, und wir
haben daraus die Geschwindigkeit berechnet. Jetzt gehen wir den umgekehrten Weg
und berechnen aus der Geschwindigkeit die Strecke. Laut Planung der
Leitsteuerung beschleunigt das FTF $2\,\text{s}$ lang gleichmäßig mit
$0.4\,\text{m/s}^2$ und fährt dann mit $0.8\,\text{m/s}$ weiter. Gegeben ist
folgender Code:

```python
# Geschwindigkeit laut Planung in m/s, jede halbe Sekunde von 0 bis 4 s
t = np.linspace(0, 4, 9)
v_plan = np.array([0.0, 0.2, 0.4, 0.6, 0.8, 0.8, 0.8, 0.8, 0.8])
dt = 0.5

# Linkssumme und Rechtssumme der Strecke in m
strecke_links = 0.0
strecke_rechts = 0.0
for k in range(len(v_plan) - 1):
    strecke_links = strecke_links + v_plan[k] * dt
    strecke_rechts = strecke_rechts + v_plan[k + 1] * dt

# Trapezregel, Strecke in m
strecke_trapez = np.trapezoid(v_plan, t)
print(f'Linkssumme:  {strecke_links:.2f} m')
print(f'Rechtssumme: {strecke_rechts:.2f} m')
print(f'Trapezregel: {strecke_trapez:.2f} m')
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Welche drei Strecken gibt der Code aus? Rechnen Sie von Hand.
2. Laut Kapitel 7.1 hat das FTF nach $4\,\text{s}$ genau $2.4\,\text{m}$
   zurückgelegt. Welcher der drei Werte stimmt? Warum ist dieses Verfahren hier
   exakt, obwohl es sonst nur eine Näherung liefert?
3. Angenommen, die geplante Geschwindigkeit ist nur jede Sekunde bekannt, also
   bei $0$, $1$, $2$, $3$ und $4\,\text{s}$. Welche Strecke liefern dann die
   Linkssumme und die Trapezregel? Rechnen Sie von Hand.
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen zu den
   Teilaufgaben 1 und 2.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Geschwindigkeit laut Planung in m/s, jede halbe Sekunde von 0 bis 4 s
t = np.linspace(0, 4, 9)
v_plan = np.array([0.0, 0.2, 0.4, 0.6, 0.8, 0.8, 0.8, 0.8, 0.8])
dt = 0.5

# Linkssumme und Rechtssumme der Strecke in m
strecke_links = 0.0
strecke_rechts = 0.0
for k in range(len(v_plan) - 1):
    strecke_links = strecke_links + v_plan[k] * dt
    strecke_rechts = strecke_rechts + v_plan[k + 1] * dt

# Trapezregel, Strecke in m
strecke_trapez = np.trapezoid(v_plan, t)
print(f'Linkssumme:  {strecke_links:.2f} m')
print(f'Rechtssumme: {strecke_rechts:.2f} m')
print(f'Trapezregel: {strecke_trapez:.2f} m')
```
Ausgabe:

```text
Linkssumme:  2.20 m
Rechtssumme: 2.60 m
Trapezregel: 2.40 m
```

1. Die Linkssumme verwendet alle Werte außer dem letzten:
   $0.5 \cdot (0 + 0.2 + 0.4 + 0.6 + 0.8 + 0.8 + 0.8 + 0.8) = 0.5 \cdot 4.4
   = 2.2\,\text{m}$. Die Rechtssumme verwendet alle Werte außer dem ersten:
   $0.5 \cdot 5.2 = 2.6\,\text{m}$. Die Trapezregel ist der Mittelwert aus
   beiden, also $2.4\,\text{m}$.
2. Die Trapezregel trifft die $2.4\,\text{m}$ genau. Sie verbindet die Werte
   durch Strecken. Die geplante Geschwindigkeit steigt bis $2\,\text{s}$
   gleichmäßig und bleibt danach konstant, sie ist also selbst ein solcher
   Streckenzug. Ihr Knick bei $2\,\text{s}$ liegt genau auf einem der
   Zeitpunkte. Die Trapeze füllen deshalb genau die Fläche unter der
   Geschwindigkeit. Die Linkssumme liefert zu wenig, weil die Geschwindigkeit
   beim Anfahren steigt und der Wert am linken Rand der kleinste im Intervall
   ist. Die Rechtssumme liefert aus demselben Grund zu viel.
3. Mit den Geschwindigkeiten $0$, $0.4$, $0.8$, $0.8$ und $0.8\,\text{m/s}$ und
   der Schrittweite $1\,\text{s}$ liefert die Linkssumme
   $1 \cdot (0 + 0.4 + 0.8 + 0.8) = 2.0\,\text{m}$. Sie liegt jetzt
   $0.4\,\text{m}$ statt $0.2\,\text{m}$ daneben. Die Trapezregel liefert
   $1 \cdot (0/2 + 0.4 + 0.8 + 0.8 + 0.8/2) = 2.4\,\text{m}$ und bleibt exakt,
   denn der Knick bei $2\,\text{s}$ ist weiterhin einer der Zeitpunkte.
````

````{admonition} Aufgabe 9.2 (✩)
:class: tip
Ein E-Transporter lädt an der Wallbox des Betriebs. Die Wallbox protokolliert
alle $30\,\text{min}$ die Ladeleistung: $11$, $11$, $7$ und $3\,\text{kW}$. Ein
Kommilitone hat die Schleife aus Kapitel 9.1, die die Energie Trapez für Trapez
aufaddiert, für dieses Protokoll abgetippt:

```python
# Ladeleistung der Wallbox in kW, alle 30 min, und Schrittweite in h
leistung = np.array([11.0, 11.0, 7.0, 3.0])
dt = 0.5

# Aufaddierte Energie in kWh
energie = np.zeros(len(leistung))
for k in range(len(leistung)):
    trapez = (leistung[k] + leistung[k + 1]) / 2 * dt
    energie[k + 1] = energie[k] + trapez
print('Aufaddierte Energie in kWh:', energie)
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Der Code bricht mit einer Fehlermeldung ab. Bei welchem Wert von `k`
   passiert das, und warum?
2. Der Kommilitone ändert die erste Zeile der Schleife in
   `for k in range(len(leistung) - 2):`. Jetzt läuft der Code ohne
   Fehlermeldung. Was gibt er aus? Gehen Sie die Schleife von Hand durch.
3. Wie viele Trapeze gehören zu vier Messwerten? Wie lautet die Schleife
   richtig, und wie viel Energie hat der Transporter in $90\,\text{min}$
   aufgenommen?
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Ladeleistung der Wallbox in kW, alle 30 min, und Schrittweite in h
leistung = np.array([11.0, 11.0, 7.0, 3.0])
dt = 0.5

# Aufaddierte Energie in kWh: vier Werte, also drei Trapeze
energie = np.zeros(len(leistung))
for k in range(len(leistung) - 1):
    trapez = (leistung[k] + leistung[k + 1]) / 2 * dt
    energie[k + 1] = energie[k] + trapez
print('Aufaddierte Energie in kWh:', energie)
```
Ausgabe:

```text
Aufaddierte Energie in kWh: [ 0.   5.5 10.  12.5]
```

1. Mit `range(len(leistung))` läuft `k` von 0 bis 3. Bei `k = 3` greift
   `leistung[k + 1]` auf `leistung[4]` zu. Das Array hat aber nur vier Einträge
   mit den Indizes 0 bis 3. Python bricht deshalb mit dieser Meldung ab:

   ```text
   IndexError: index 4 is out of bounds for axis 0 with size 4
   ```

   Die drei Trapeze sind zu diesem Zeitpunkt schon aufaddiert. Rechts vom
   letzten Messwert gibt es aber kein Intervall mehr.
2. Mit `range(len(leistung) - 2)` läuft `k` nur von 0 bis 1. Für `k = 0` ist
   das Trapez $(11 + 11)/2 \cdot 0.5 = 5.5\,\text{kWh}$, für `k = 1` kommen
   $(11 + 7)/2 \cdot 0.5 = 4.5\,\text{kWh}$ hinzu. Der letzte Eintrag bleibt
   null, so wie ihn `np.zeros()` angelegt hat. Der Code gibt aus:

   ```text
   Aufaddierte Energie in kWh: [ 0.   5.5 10.   0. ]
   ```

   Das Trapez von $60$ bis $90\,\text{min}$ fehlt. Dieser Fehler ist
   tückischer als der erste, denn es erscheint keine Fehlermeldung. Wer nur
   `energie[-1]` ausgibt, liest $0\,\text{kWh}$ ab.
3. Vier Messwerte begrenzen drei Intervalle, also drei Trapeze. Allgemein
   gehören zu $n$ Messwerten $n - 1$ Intervalle. Richtig ist deshalb
   `range(len(leistung) - 1)` wie in Kapitel 9.1. Der Transporter hat in
   $90\,\text{min}$ laut Trapezregel etwa $12.5\,\text{kWh}$ aufgenommen.
````

````{admonition} Aufgabe 9.3 (✩)
:class: tip
Ein kugelförmiger Behälter hat innen den Durchmesser $1\,\text{m}$, also den
Radius $R = 0.5\,\text{m}$. Wie beim Weinfass in Kapitel 9.3 zerlegen wir ihn
in waagerechte Scheiben. In der Höhe $z$ über dem tiefsten Punkt ist der
Querschnitt ein Kreis mit der Fläche

$$A(z) = \pi\,\bigl(R^2 - (z - R)^2\bigr).$$

Gegeben ist folgender Code:

```python
# Radius des Behälters in m
radius = 0.5

# Querschnittsfläche in m² an drei und an fünf Höhen in m
hoehe_3 = np.linspace(0, 1, 3)
flaeche_3 = np.pi * (radius**2 - (hoehe_3 - radius)**2)
hoehe_5 = np.linspace(0, 1, 5)
flaeche_5 = np.pi * (radius**2 - (hoehe_5 - radius)**2)

# Volumen in L mit der Trapezregel und der Simpson-Regel
trapez_3 = np.trapezoid(flaeche_3, hoehe_3) * 1000
trapez_5 = np.trapezoid(flaeche_5, hoehe_5) * 1000
simpson_3 = simpson(flaeche_3, x=hoehe_3) * 1000
print(f'Trapezregel, drei Höhen:   {trapez_3:.1f} L')
print(f'Trapezregel, fünf Höhen:   {trapez_5:.1f} L')
print(f'Simpson-Regel, drei Höhen: {simpson_3:.1f} L')
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Welche drei Werte stehen in `flaeche_3`? Berechnen Sie damit von Hand das
   Volumen mit der Trapezregel und mit der Keplerschen Fassregel.
2. Eine Kugel hat das Volumen $\frac{4}{3}\pi R^3$. Welche der beiden Regeln
   trifft es genau, und warum? Denken Sie daran, wie $A(z)$ von $z$ abhängt.
3. Schätzen Sie ohne weitere Handrechnung mit der Fehlerordnung der
   Trapezregel, welchen Wert `trapez_5` hat.
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Radius des Behälters in m
radius = 0.5

# Querschnittsfläche in m² an drei und an fünf Höhen in m
hoehe_3 = np.linspace(0, 1, 3)
flaeche_3 = np.pi * (radius**2 - (hoehe_3 - radius)**2)
hoehe_5 = np.linspace(0, 1, 5)
flaeche_5 = np.pi * (radius**2 - (hoehe_5 - radius)**2)

# Volumen in L mit der Trapezregel und der Simpson-Regel
trapez_3 = np.trapezoid(flaeche_3, hoehe_3) * 1000
trapez_5 = np.trapezoid(flaeche_5, hoehe_5) * 1000
simpson_3 = simpson(flaeche_3, x=hoehe_3) * 1000
print(f'Trapezregel, drei Höhen:   {trapez_3:.1f} L')
print(f'Trapezregel, fünf Höhen:   {trapez_5:.1f} L')
print(f'Simpson-Regel, drei Höhen: {simpson_3:.1f} L')
```
Ausgabe:

```text
Trapezregel, drei Höhen:   392.7 L
Trapezregel, fünf Höhen:   490.9 L
Simpson-Regel, drei Höhen: 523.6 L
```

1. Die drei Höhen sind $0$, $0.5$ und $1\,\text{m}$. Ganz unten und ganz oben
   ist der Querschnitt nur ein Punkt, in der Mitte ein Kreis mit dem Radius
   $0.5\,\text{m}$. `flaeche_3` enthält also $0$, $\pi \cdot 0.25 \approx
   0.785$ und $0\,\text{m}^2$. Die Trapezregel mit der Schrittweite
   $0.5\,\text{m}$ liefert $0.5 \cdot \pi \cdot 0.25 = \pi/8 \approx
   0.3927\,\text{m}^3$, also $392.7\,\text{L}$. Die Fassregel liefert
   $\frac{1}{6} \cdot 4 \cdot \pi \cdot 0.25 = \pi/6 \approx
   0.5236\,\text{m}^3$, also $523.6\,\text{L}$. `simpson()` rechnet mit drei
   Werten genau die Fassregel.
2. Die Kugel hat das Volumen $\frac{4}{3}\pi \cdot 0.5^3 = \pi/6 \approx
   0.5236\,\text{m}^3$. Die Fassregel trifft es genau. Multiplizieren wir
   $A(z)$ aus, ergibt sich $A(z) = \pi\,(2Rz - z^2)$. Die Querschnittsfläche
   ist also eine Parabel in $z$. Die Fassregel legt eine Parabel durch die drei
   Punkte, und das ist genau $A(z)$ selbst. Die Trapezregel schneidet die nach
   oben gewölbte Kurve dagegen ab wie beim Weinfass und liefert
   $130.9\,\text{L}$ zu wenig, ein Viertel des Volumens.
3. Mit fünf Höhen halbiert sich die Schrittweite. Bei Fehlerordnung 2 sinkt
   der Fehler auf ein Viertel, also auf $130.9 / 4 \approx 32.7\,\text{L}$. Die
   Schätzung ist $523.6 - 32.7 = 490.9\,\text{L}$. Hier trifft sie sogar genau,
   denn bei einer Parabel ist der Fehler der Trapezregel genau proportional zum
   Quadrat der Schrittweite.
````

````{admonition} Aufgabe 9.4 (✩✩)
:class: tip
In einer Hydraulikleitung mit dem Innenradius $R = 25\,\text{mm}$ strömt Öl
laminar. Ein Messgerät hat die Strömungsgeschwindigkeit in elf Abständen von
der Rohrachse gemessen, von der Achse bis zur Wand:

```python
# Abstand von der Rohrachse in mm und Strömungsgeschwindigkeit in m/s
abstand = np.linspace(0, 25, 11)
geschwindigkeit = np.array([2.00, 1.98, 1.92, 1.82, 1.68, 1.50,
                            1.28, 1.02, 0.72, 0.38, 0.00])
```

In der Mitte strömt das Öl am schnellsten, an der Wand haftet es. Den
Querschnitt zerlegen wir in dünne Ringe. Schneiden wir einen Ring mit dem
Radius $r$ und der Breite $\Delta r$ auf und biegen ihn gerade, entsteht fast
ein Rechteck mit der Länge $2\pi r$ und der Breite $\Delta r$. Durch den Ring
fließt pro Sekunde also ungefähr das Volumen $v(r) \cdot 2\pi r\,\Delta r$.
Der **Volumenstrom** ist deshalb

$$Q = \int_0^R v(r) \cdot 2\pi r\,\mathrm{d}r.$$

1. Berechnen Sie $Q$ mit `np.trapezoid()` und mit `simpson()` in Litern pro
   Minute. Achten Sie auf die Einheiten.
2. Eine Kollegin rechnet einfacher: Mittelwert der elf Messwerte mal
   Querschnittsfläche $\pi R^2$. Welchen Volumenstrom erhält sie? Warum ist ihr
   Ergebnis zu groß?
3. Für laminare Strömung in einem langen, geraden Rohr gilt
   $v(r) = v_\text{max}\,(1 - r^2/R^2)$ mit $v_\text{max} = 2\,\text{m/s}$.
   Schreiben Sie den Integranden $v(r) \cdot 2\pi r$ als Python-Funktion und
   berechnen Sie $Q$ mit `quad()`.
   Welche der beiden Regeln aus Teilaufgabe 1 trifft diesen Wert, und warum?
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
1. Wir rechnen den Abstand in Meter um. Dann hat der Integrand die Einheit
   $\text{m/s} \cdot \text{m} = \text{m}^2/\text{s}$, und das Integral hat die
   Einheit $\text{m}^3/\text{s}$. Ein Kubikmeter pro Sekunde sind
   $1000\,\text{L/s}$ oder $60\,000\,\text{L/min}$.

   ```python
   # Abstand von der Rohrachse in mm und Strömungsgeschwindigkeit in m/s
   abstand = np.linspace(0, 25, 11)
   geschwindigkeit = np.array([2.00, 1.98, 1.92, 1.82, 1.68, 1.50,
                               1.28, 1.02, 0.72, 0.38, 0.00])

   # Teilaufgabe 1: Abstand in m und Integrand v(r) mal 2 pi r in m²/s
   r = abstand / 1000
   integrand = geschwindigkeit * 2 * np.pi * r

   # Teilaufgabe 1: Volumenstrom von m³/s in L/min umgerechnet
   q_trapez = np.trapezoid(integrand, r) * 1000 * 60
   q_simpson = simpson(integrand, x=r) * 1000 * 60
   print(f'Trapezregel:   {q_trapez:.2f} L/min')
   print(f'Simpson-Regel: {q_simpson:.2f} L/min')
   ```

   Ausgabe:

   ```text
   Trapezregel:   116.63 L/min
   Simpson-Regel: 117.81 L/min
   ```

   Durch die Leitung fließen knapp $118\,\text{L}$ Öl pro Minute. Die beiden
   Regeln unterscheiden sich um etwa $1.2\,\text{L/min}$, also um etwa 1
   Prozent.
2. Die Kollegin mittelt die Messwerte und multipliziert mit der Fläche:

   ```python
   # Teilaufgabe 2: Mittelwert der Messwerte mal Querschnittsfläche
   v_mittel = np.mean(geschwindigkeit)
   flaeche_rohr = np.pi * 0.025**2
   q_mittel = v_mittel * flaeche_rohr * 1000 * 60
   print(f'Mittelwert der Messwerte: {v_mittel:.2f} m/s')
   print(f'Volumenstrom damit:       {q_mittel:.2f} L/min')
   ```

   Ausgabe:

   ```text
   Mittelwert der Messwerte: 1.30 m/s
   Volumenstrom damit:       153.15 L/min
   ```

   Das sind 30 Prozent mehr als mit der Simpson-Regel. Der Mittelwert zählt
   alle elf Messstellen gleich. Die Ringe werden aber nach außen immer größer,
   denn ihr Umfang $2\pi r$ wächst mit dem Abstand. Außen strömt das Öl
   langsam, aber durch große Ringe. Innen strömt es schnell, aber durch kleine
   Ringe. Der einfache Mittelwert gewichtet die schnelle Mitte deshalb zu
   stark. Der
   Faktor $2\pi r$ im Integranden sorgt dafür, dass jede Geschwindigkeit mit
   der Fläche ihres Rings zählt. Teilen wir den Volumenstrom aus
   Teilaufgabe 1 durch die Querschnittsfläche, beträgt die mittlere
   Geschwindigkeit nur $1.0\,\text{m/s}$, die Hälfte der Geschwindigkeit in
   der Mitte.
3. Der Integrand wird zur Funktion des Abstands $r$ in Meter:

   ```python
   # Teilaufgabe 3: laminares Profil als Funktion, r in m
   def integrand_laminar(r):
       """v(r) mal 2 pi r in m²/s für den Abstand r in m."""
       v = 2.0 * (1 - r**2 / 0.025**2)
       return v * 2 * np.pi * r

   # Teilaufgabe 3: Volumenstrom mit quad(), von m³/s in L/min umgerechnet
   ergebnis, fehlerschaetzung = quad(integrand_laminar, 0, 0.025)
   q_quad = ergebnis * 1000 * 60
   print(f'quad(): {q_quad:.2f} L/min')
   ```

   Ausgabe:

   ```text
   quad(): 117.81 L/min
   ```

   Die Simpson-Regel trifft den Wert von `quad()`, die Trapezregel liegt 1
   Prozent darunter. Der Integrand
   $v(r) \cdot 2\pi r = 2\pi v_\text{max}\,(r - r^3/R^2)$ ist ein Polynom
   dritten Grades. Nach Kapitel 9.3 ist die Fassregel für jedes solche Polynom
   exakt, also auch die Simpson-Regel, die sie auf jedes der fünf
   Doppelintervalle anwendet. Die Messwerte liegen hier genau auf dem
   laminaren Profil, deshalb stimmt `simpson()` mit `quad()` überein.
````

````{admonition} Aufgabe 9.5 (✩✩✩) Mini-Projekt: Lohnt sich die Simpson-Regel?
:class: tip
In Kapitel 9.1 hat die Trapezregel aus dem Fünf-Minuten-Protokoll der
Schnellladesäule $54.44\,\text{kWh}$ berechnet. Der Zähler der Säule hat
$54.72\,\text{kWh}$ gemessen. Beim Weinfass in Kapitel 9.3 war die
Simpson-Regel mit denselben Messwerten viel genauer als die Trapezregel. Ein
Kommilitone schlägt deshalb vor, auch das Protokoll der Säule mit der
Simpson-Regel auszuwerten. Prüfen Sie den Vorschlag.

**Teil 1:** Das Fünf-Minuten-Protokoll aus Kapitel 9.1 lautet:

```python
# Fünf-Minuten-Protokoll: Zeitpunkte in min und Ladeleistung in kW
t_5 = np.linspace(0, 30, 7)
leistung_5 = np.array([150.0, 150.0, 150.0, 112.5, 79.4, 61.4, 50.0])
```

Berechnen Sie die Energie mit `np.trapezoid()` und mit `simpson()`. Um wie
viel weichen beide vom Zähler ab?

**Teil 2:** Die Ladeleistung folgt einer Formel. Bis $12\,\text{min}$ beträgt
sie $150\,\text{kW}$, danach gilt mit $t$ in Minuten

$$P(t) = \frac{1350}{t - 3}\,\text{kW}.$$

Die Protokolle aus Kapitel 9.1 enthalten die auf eine Nachkommastelle
gerundeten Werte dieser Formel. Die Funktion `ladeleistung()` berechnet die
Formel für einen einzelnen Zeitpunkt. Die Funktion `protokoll()` erzeugt daraus
ein Protokoll mit einem Eintrag alle `takt` Minuten. Sie ruft `ladeleistung()`
dazu in einer Schleife für jeden Zeitpunkt auf und gibt wie `quad()` zwei
Ergebnisse zurück, die Zeitpunkte und die Ladeleistungen.

```python
# Ladekurve als Formel
def ladeleistung(t):
    """Ladeleistung in kW zum Zeitpunkt t in min."""
    if t <= 12:
        return 150.0
    return 1350 / (t - 3)

def protokoll(takt):
    """Protokoll von 0 bis 30 min, ein Eintrag alle takt Minuten.

    takt muss 30 Minuten ohne Rest teilen. Gibt die Zeitpunkte in min und
    die Ladeleistung in kW zurück.
    """
    anzahl = round(30 / takt)
    t = np.linspace(0, 30, anzahl + 1)
    leistung = np.zeros(len(t))
    for k in range(len(t)):
        leistung[k] = ladeleistung(t[k])
    return t, leistung
```

Berechnen Sie die Energie aus der Formel mit `quad()`. Passt sie zum Zähler?
Erzeugen Sie dann mit `protokoll(3)` ein Protokoll im Drei-Minuten-Takt.
Berechnen Sie, um wie viel die Trapezregel und die Simpson-Regel vom Wert aus
`quad()` abweichen.

**Teil 3:** Berechnen Sie die beiden Abweichungen in einer Schleife für die
Takte $5$, $2.5$ und $1.25\,\text{min}$ sowie $3$, $1.5$ und
$0.75\,\text{min}$. Um welchen Faktor sinken die Abweichungen, wenn sich der
Takt halbiert? Passt das zu den Fehlerordnungen aus Kapitel 9.3?

**Abschlussfrage:** Warum ist die Simpson-Regel im Fünf-Minuten-Takt schlechter
als die Trapezregel, im Drei-Minuten-Takt aber viel besser? Denken Sie an den
Knick bei $12\,\text{min}$ und an die Doppelintervalle. Was antworten Sie dem
Kommilitonen?
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
**Teil 1:** Wir rechnen die Zeitpunkte wie in Kapitel 9.1 in Stunden um:

```python
# Fünf-Minuten-Protokoll: Zeitpunkte in min und Ladeleistung in kW
t_5 = np.linspace(0, 30, 7)
leistung_5 = np.array([150.0, 150.0, 150.0, 112.5, 79.4, 61.4, 50.0])

# Teil 1: Energie in kWh, Zeitpunkte in h umgerechnet
energie_trapez = np.trapezoid(leistung_5, t_5 / 60)
energie_simpson = simpson(leistung_5, x=t_5 / 60)

# Teil 1: Abweichungen vom Zähler in kWh
abweichung_trapez = energie_trapez - 54.72
abweichung_simpson = energie_simpson - 54.72
print(f'Trapezregel:   {energie_trapez:.2f} kWh')
print(f'Simpson-Regel: {energie_simpson:.2f} kWh')
print(f'Abweichung Trapezregel:   {abweichung_trapez:+.2f} kWh')
print(f'Abweichung Simpson-Regel: {abweichung_simpson:+.2f} kWh')
```

Ausgabe:

```text
Trapezregel:   54.44 kWh
Simpson-Regel: 54.29 kWh
Abweichung Trapezregel:   -0.28 kWh
Abweichung Simpson-Regel: -0.43 kWh
```

Anders als beim Weinfass liegt die Simpson-Regel hier weiter vom Zähler
entfernt als die Trapezregel. Für dieses Protokoll würde der Vorschlag das
Ergebnis also verschlechtern.

**Teil 2:** Die Funktionen übernehmen wir aus der Aufgabe. `quad()` erhält
die Zeitpunkte in Minuten, das Ergebnis in $\text{kW} \cdot \text{min}$ teilen
wir durch 60:

```python
# Ladekurve als Formel
def ladeleistung(t):
    """Ladeleistung in kW zum Zeitpunkt t in min."""
    if t <= 12:
        return 150.0
    return 1350 / (t - 3)

def protokoll(takt):
    """Protokoll von 0 bis 30 min, ein Eintrag alle takt Minuten.

    takt muss 30 Minuten ohne Rest teilen. Gibt die Zeitpunkte in min und
    die Ladeleistung in kW zurück.
    """
    anzahl = round(30 / takt)
    t = np.linspace(0, 30, anzahl + 1)
    leistung = np.zeros(len(t))
    for k in range(len(t)):
        leistung[k] = ladeleistung(t[k])
    return t, leistung

# Teil 2: Energie aus der Formel, von kW·min in kWh umgerechnet
ergebnis, fehlerschaetzung = quad(ladeleistung, 0, 30)
energie_quad = ergebnis / 60
print(f'quad():                   {energie_quad:.4f} kWh')

# Teil 2: Drei-Minuten-Takt, Abweichungen vom Wert aus quad() in kWh
t_3, leistung_3 = protokoll(3)
abweichung_trapez = np.trapezoid(leistung_3, t_3 / 60) - energie_quad
abweichung_simpson = simpson(leistung_3, x=t_3 / 60) - energie_quad
print(f'Abweichung Trapezregel:   {abweichung_trapez:+.4f} kWh')
print(f'Abweichung Simpson-Regel: {abweichung_simpson:+.4f} kWh')
```

Ausgabe:

```text
quad():                   54.7188 kWh
Abweichung Trapezregel:   +0.1830 kWh
Abweichung Simpson-Regel: +0.0074 kWh
```

Auf zwei Nachkommastellen gerundet liefert `quad()` genau den Wert des Zählers.
Den Wert von `quad()` verwenden wir als exakten Vergleichswert für die Formel.
Im Drei-Minuten-Takt liegt die Simpson-Regel nur $0.0074\,\text{kWh}$ daneben,
die Trapezregel mit $0.18\,\text{kWh}$ fast 25-mal so weit. Die Trapezregel
liefert jetzt zu viel: Nach dem Knick hängt die Ladekurve durch wie die
Druckkurve des Kompressors in Kapitel 9.4, und die Trapeze sind zu groß.

**Teil 3:** In der Schleife erzeugen wir für jeden Takt ein neues Protokoll:

```python
# Teil 3: Abweichungen vom Wert aus quad() in kWh für sechs Takte
for takt in [5.0, 2.5, 1.25, 3.0, 1.5, 0.75]:
    t, leistung = protokoll(takt)
    abweichung_trapez = np.trapezoid(leistung, t / 60) - energie_quad
    abweichung_simpson = simpson(leistung, x=t / 60) - energie_quad
    print(f'Takt {takt:4.2f} min: Trapezregel {abweichung_trapez:+.1e}, '
          f'Simpson-Regel {abweichung_simpson:+.1e}')
```

Ausgabe:

```text
Takt 5.00 min: Trapezregel -2.8e-01, Simpson-Regel -4.3e-01
Takt 2.50 min: Trapezregel -1.8e-02, Simpson-Regel +6.9e-02
Takt 1.25 min: Trapezregel -2.0e-02, Simpson-Regel -2.1e-02
Takt 3.00 min: Trapezregel +1.8e-01, Simpson-Regel +7.4e-03
Takt 1.50 min: Trapezregel +4.6e-02, Simpson-Regel +5.4e-04
Takt 0.75 min: Trapezregel +1.2e-02, Simpson-Regel +3.5e-05
```

Beim Drei-Minuten-Takt und seinen Halbierungen sinkt die Abweichung der
Trapezregel bei jeder Halbierung auf etwa ein Viertel. Die Abweichung der
Simpson-Regel sinkt um den Faktor 14 und dann 15. Das passt zu den
Fehlerordnungen 2 und 4 aus Kapitel 9.3. Der Faktor der Simpson-Regel liegt
noch etwas unter 16 und nähert sich diesem Wert, je kleiner der Takt wird.

Beim Fünf-Minuten-Takt und seinen Halbierungen gibt es kein solches Muster.
Von $5$ auf $2.5\,\text{min}$ sinkt die Abweichung der Trapezregel um den
Faktor 15, von $2.5$ auf $1.25\,\text{min}$ wächst sie sogar leicht. Die
Simpson-Regel wechselt das Vorzeichen. Bei $1.25\,\text{min}$ sind beide Regeln
gleich gut, die Simpson-Regel hat ihren Vorsprung verloren.

**Abschlussfrage:** Im Fünf-Minuten-Takt bilden die Zeitpunkte $10$, $15$ und
$20\,\text{min}$ ein Doppelintervall, und der Knick bei $12\,\text{min}$ liegt
mitten darin. Die Parabel durch die drei Werte ist glatt und kann der Ecke
nicht folgen. Sie schneidet die Ecke ab wie die Trapeze, in diesem
Doppelintervall fehlt der Simpson-Regel sogar etwas mehr Energie als der
Trapezregel. Im Doppelintervall von $20$ bis $30\,\text{min}$ verläuft die
Kurve glatt. Dort ist die Simpson-Regel viel genauer, die Trapezregel liefert
dort zu viel. Dieser Überschuss gleicht einen Teil der fehlenden Energie an der
Ecke aus. Die Trapezregel liegt also näher am Zähler, weil sich ihre Fehler
teilweise aufheben.

Im Drei-Minuten-Takt ist $12\,\text{min}$ der Zeitpunkt mit dem Index 4. Die
Doppelintervalle reichen von $0$ bis $6$, von $6$ bis $12$, von $12$ bis
$18\,\text{min}$ und so weiter. Der Knick liegt genau auf der Grenze zwischen
zwei Doppelintervallen. In jedem Doppelintervall verläuft die Kurve glatt, und
die Parabeln passen so gut wie beim Weinfass. Dasselbe gilt für die Takte $1.5$
und $0.75\,\text{min}$, deshalb zeigen sich dort die Fehlerordnungen. Wie in
Kapitel 9.3 setzen sie voraus, dass sich der Integrand in jedem Doppelintervall
oft genug ableiten lässt.

Dem Kommilitonen antworten wir: Ihren Vorsprung behält die Simpson-Regel hier,
wenn der Knick auf der Grenze zwischen zwei Doppelintervallen liegt. Wann die
Ladeelektronik drosselt, hängt aber vom Auto und vom Ladezustand bei Ankunft
ab. Die Säule kennt diesen Zeitpunkt vorher nicht. Bei einem Knick an
unbekannter Stelle bringt die Simpson-Regel deshalb keinen verlässlichen
Vorteil. Mehr hilft es, öfter zu protokollieren. Im Minutentakt aus
Kapitel 9.1 lag schon die Trapezregel nur $0.02\,\text{kWh}$ neben dem Zähler.
Für glatte Kurven wie beim Weinfass bleibt die Simpson-Regel meist die bessere
Wahl.
````
