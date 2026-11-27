---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 9.1 Energie aus der Ladekurve

In Kapitel 7.1 haben wir aus der Strecke des FTF seine Geschwindigkeit
berechnet, also abgeleitet. Jetzt gehen wir den umgekehrten Weg. Ein E-Auto
lädt an einer Schnellladesäule. Die App der Säule protokolliert alle fünf
Minuten die Ladeleistung in Kilowatt. Auf der Abrechnung steht aber die
geladene Energie in Kilowattstunden. *Wie berechnen wir aus den protokollierten
Leistungen die geladene Energie?*

Die Energie ist das Integral der Leistung über der Zeit, anschaulich die
Fläche unter der Ladekurve. Die Leistung kennen wir aber nur zu einzelnen
Zeitpunkten, als Zahlenfolge und nicht als Formel. Wir nähern das Integral
deshalb aus den Messwerten an. Diese Näherung heißt **numerische Integration**
oder auch **Quadratur**.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können die Energie als Fläche unter der Leistungskurve deuten und
  sie mit der **Linkssumme** und der **Rechtssumme** annähern.
* [ ] Sie können begründen, welche der beiden Summen bei fallender Leistung zu
  viel und welche zu wenig Energie liefert.
* [ ] Sie können die **Trapezregel** erklären und als Schleife programmieren,
  die die Energie Schritt für Schritt aufaddiert.
* [ ] Sie können `np.trapezoid()` einsetzen und wissen, welche Einheit das
  Ergebnis hat.
```

## Wie viel Energie wurde geladen?

Das E-Auto hat einen Akku mit $75\,\text{kWh}$ und kommt mit 10 Prozent
Ladezustand an der Säule an. Der **Ladezustand** gibt an, welcher Anteil der
$75\,\text{kWh}$ im Akku gespeichert ist. Nach einer halben Stunde fährt das
Auto weiter. Die Abrechnung der Säule zeigt $54.72\,\text{kWh}$. Diesen Wert
misst ein Zähler in der Säule laufend, nicht nur alle fünf Minuten. Wir
prüfen, ob das Protokoll der App dazu passt. Ladeverluste vernachlässigen wir
dabei: Die Energie, die die Säule liefert, kommt vollständig im Akku an.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Zeitpunkte der Protokolleinträge in min
t = np.array([0.0, 5.0, 10.0, 15.0, 20.0, 25.0, 30.0])
# Ladeleistung in kW
leistung = np.array([150.0, 150.0, 150.0, 112.5, 79.4, 61.4, 50.0])
```

Ein Diagramm zeigt, wie sich die Ladeleistung während der halben Stunde
ändert.

```{code-cell} python
# Diagramm: Ladeleistung über der Zeit
fig, ax = plt.subplots(figsize=(7, 4))
ax.plot(t, leistung, marker='o')
ax.set_xlabel('Zeit in min')
ax.set_ylabel('Ladeleistung in kW')
ax.set_title('Protokoll der Schnellladesäule')
ax.grid(True)
plt.show()
```

In den ersten zehn Minuten zeigt das Protokoll die volle Leistung. Danach
sinkt sie, je voller der Akku wird. Beim Schnellladen drosselt die
Ladeelektronik ab einem bestimmten Ladezustand den Strom, um den Akku zu
schonen. Bei diesem Auto liegt die Grenze bei 50 Prozent.

Bei konstanter Leistung ist die Energie einfach Leistung mal Zeit. In den
ersten zehn Minuten, also in $1/6\,\text{h}$, nimmt der Akku deshalb diese
Energie auf:

$$E = 150\,\text{kW} \cdot \frac{1}{6}\,\text{h} = 25\,\text{kWh}.$$

Im Diagramm ist das die Fläche eines Rechtecks mit Höhe $150\,\text{kW}$ und
Breite $10\,\text{min}$. Ändert sich die Leistung, wird aus dem Rechteck die
Fläche unter der Kurve. Mathematisch ist das ein bestimmtes Integral:

$$E = \int_{0}^{T} P(t)\,\mathrm{d}t.$$

```{admonition} Aus der Mathematik: Integral als Fläche
:class: note
Das bestimmte Integral $\int_a^b f(t)\,\mathrm{d}t$ ist die Fläche zwischen
dem Graphen von $f$ und der $t$-Achse im Bereich von $a$ bis $b$. Liegt der
Graph unter der Achse, zählt die Fläche dort negativ. Hat der Integrand eine
Einheit, hat das Integral die Einheit des Integranden mal die Einheit von $t$,
hier also Kilowatt mal Stunden. Die Grundlagen dazu können Sie in der
[Vorbereitung](chapter09_sec00.md) wiederholen.
```

Die einfachste Näherung zerlegt die Fläche in Rechtecke, eines pro
Zeitintervall zwischen zwei Protokolleinträgen. Als Höhe nehmen wir die
Leistung am Anfang des Intervalls, also am linken Rand. Das ergibt die
**Linkssumme**. Mit $n$ Intervallen der Breite $\Delta t$ und den Messwerten
$P_0, P_1, \dots, P_n$ lautet sie

$$E \approx \bigl(P_0 + P_1 + \dots + P_{n-1}\bigr)\,\Delta t.$$

Der letzte Messwert $P_n$ kommt nicht vor, denn er ist der rechte Rand des
letzten Intervalls. Damit Kilowattstunden herauskommen, rechnen wir die
Schrittweite von Minuten in Stunden um.

```{code-cell} python
# Schrittweite in h: 5 min sind 5/60 h
dt = 5 / 60

# Linkssumme: Rechteckhöhe ist die Leistung am Anfang des Intervalls
energie_links = 0.0
for k in range(len(leistung) - 1):
    energie_links = energie_links + leistung[k] * dt
print(f'Linkssumme: {energie_links:.2f} kWh')
```

Die Linkssumme liefert $58.61\,\text{kWh}$, fast $4\,\text{kWh}$ mehr als die
Abrechnung. Um zu sehen, woher die Abweichung kommt, zeichnen wir die
Rechtecke ein. `ax.bar()` zeichnet Balken an den Stellen `t[:-1]`, das sind
alle Zeitpunkte außer dem letzten. Mit `align='edge'` beginnt jeder Balken an
seinem Zeitpunkt, mit `width=5` ist er fünf Minuten breit.

```{code-cell} python
# Diagramm: Rechtecke der Linkssumme und Messwerte
fig, ax = plt.subplots(figsize=(7, 4))
ax.bar(t[:-1], leistung[:-1], width=5, align='edge', alpha=0.3,
       edgecolor='black', label='Rechtecke der Linkssumme')
ax.plot(t, leistung, marker='o', label='Messwerte')
ax.set_xlabel('Zeit in min')
ax.set_ylabel('Ladeleistung in kW')
ax.set_title('Linkssumme')
ax.legend()
ax.grid(True)
plt.show()
```

Ab $10\,\text{min}$ ragen die Rechtecke über die Kurve hinaus. Weil die
Leistung fällt, ist der Wert am linken Rand der größte im Intervall. Die
Linkssumme liefert bei fallender Leistung deshalb zu viel Energie.

```{admonition} Mini-Übung (✩)
:class: tip
1. Mit der Leistung am Ende des Intervalls als Rechteckhöhe entsteht die
   **Rechtssumme**. Ändern Sie die Schleife und berechnen Sie die
   Rechtssumme.
2. Beantworten Sie ohne Code: Liefert die Rechtssumme zu viel oder zu wenig
   Energie? Begründen Sie in einem Satz.
3. Beantworten Sie ohne Code: Warum sind die Rechtecke der Linkssumme und der
   Rechtssumme in den ersten zehn Minuten gleich?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
1. Statt `leistung[k]` verwenden wir `leistung[k + 1]`, den Wert am rechten
   Rand des Intervalls:

   ```python
   # Teilaufgabe 1: Rechtssumme
   energie_rechts = 0.0
   for k in range(len(leistung) - 1):
       energie_rechts = energie_rechts + leistung[k + 1] * dt
   print(f'Rechtssumme: {energie_rechts:.2f} kWh')
   ```

   Die Rechtssumme liefert $50.27\,\text{kWh}$.
2. Die Rechtssumme liefert zu wenig Energie. Bei fallender Leistung ist der
   Wert am rechten Rand der kleinste im Intervall, die Rechtecke bleiben also
   unter der Kurve.
3. In den ersten zehn Minuten ist die Leistung konstant $150\,\text{kW}$. Linker
   und rechter Rand haben denselben Wert, beide Rechtecke sind gleich hoch und
   stimmen genau mit der Fläche unter der Kurve überein.
````

## Rechtecke oder Trapeze?

Die Linkssumme liefert zu viel, die Rechtssumme zu wenig. Eine bessere Idee
liefert das Diagramm selbst: `ax.plot()` verbindet benachbarte Messpunkte
durch Strecken. Die Fläche unter einer solchen Strecke ist ein **Trapez**.
Seine Fläche ist der Mittelwert der beiden Randwerte mal die Breite:

$$\Delta E_k = \frac{P_k + P_{k+1}}{2}\,\Delta t.$$

Addieren wir alle Trapeze, erhalten wir die **Trapezregel**:

$$E \approx \Bigl(\frac{P_0}{2} + P_1 + \dots + P_{n-1} + \frac{P_n}{2}\Bigr)
\,\Delta t.$$

Die inneren Messwerte gehören zu zwei Trapezen und zählen voll, die beiden
Randwerte gehören nur zu einem Trapez und zählen halb.

Wir addieren die Trapeze in einer Schleife auf. Dabei merken wir uns nach
jedem Intervall, wie viel Energie bis dahin geladen ist. Das Array `energie`
legen wir dazu vorher mit `np.zeros()` an. Am Anfang ist noch keine Energie
geladen, `energie[0]` bleibt also null. Jeder weitere Eintrag ist der vorige
plus das nächste Trapez.

```{code-cell} python
# Aufaddierte Energie: energie[k] ist die bis zum Zeitpunkt t[k] geladene
# Energie in kWh
energie = np.zeros(len(leistung))
for k in range(len(leistung) - 1):
    trapez = (leistung[k] + leistung[k + 1]) / 2 * dt
    energie[k + 1] = energie[k] + trapez
print('Aufaddierte Energie in kWh:', np.round(energie, 2))
```

Nach $30\,\text{min}$ sind laut Trapezregel $54.44\,\text{kWh}$ geladen. Das
sind nur $0.28\,\text{kWh}$ weniger, als der Zähler der Säule misst. Die
Linkssumme lag um fast $4\,\text{kWh}$ daneben, die Rechtssumme um gut
$4\,\text{kWh}$. Das Diagramm zeigt oben die Trapeze und unten die aufaddierte
Energie. `ax.fill_between()` füllt die Fläche zwischen dem Streckenzug und der
Zeitachse, das sind genau die Trapeze. `ax.vlines()` zeichnet an jedem
Messzeitpunkt eine senkrechte Linie von null bis zum Messwert und trennt so die
Trapeze voneinander.

```{code-cell} python
# Diagramm: Trapeze unter der Ladekurve und aufaddierte Energie
fig, ax = plt.subplots(nrows=2, ncols=1, figsize=(7, 6), sharex=True)

ax[0].fill_between(t, leistung, alpha=0.3, label='Trapeze')
ax[0].vlines(t, 0, leistung, color='gray', linewidth=1)
ax[0].plot(t, leistung, marker='o', label='Messwerte')
ax[0].set_ylabel('Ladeleistung in kW')
ax[0].set_title('Trapezregel')

ax[1].plot(t, energie, marker='o', label='Trapezregel')
ax[1].plot(30, 54.72, marker='D', color='gray', label='Abrechnung')
ax[1].set_xlabel('Zeit in min')
ax[1].set_ylabel('geladene Energie in kWh')

for einzelachse in ax:
    einzelachse.legend()
    einzelachse.grid(True)
plt.tight_layout()
plt.show()
```

Die aufaddierte Energie steigt in den ersten zehn Minuten gleichmäßig, danach
immer flacher. Die Steigung jeder Strecke ist die mittlere Ladeleistung im
Intervall. Je höher die Leistung, desto schneller wächst also die Energie.
Für die tatsächlich geladene Energie als Funktion der Zeit gilt sogar: Ihre
Ableitung ist die Ladeleistung. Integrieren und Ableiten sind Umkehrungen
voneinander.

```{admonition} Mini-Übung (✩)
:class: tip
Ein E-Transporter lädt an der Wallbox des Betriebs. Die Wallbox
protokolliert alle $30\,\text{min}$ die Ladeleistung:

| Zeit in min | 0 | 30 | 60 | 90 |
|---|---|---|---|---|
| Ladeleistung in kW | 11 | 11 | 7 | 3 |

1. Berechnen Sie ohne Code die drei Trapezflächen und die gesamte geladene
   Energie. Achten Sie auf die Einheit der Schrittweite.
2. Prüfen Sie Ihr Ergebnis mit der Schleife aus diesem Abschnitt.
3. Beantworten Sie ohne Code: Warum liefert die Trapezregel immer genau den
   Mittelwert aus Linkssumme und Rechtssumme?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
1. Die Schrittweite beträgt $30\,\text{min} = 0.5\,\text{h}$. Die drei Trapeze
   haben die Flächen $(11 + 11)/2 \cdot 0.5 = 5.5\,\text{kWh}$,
   $(11 + 7)/2 \cdot 0.5 = 4.5\,\text{kWh}$ und
   $(7 + 3)/2 \cdot 0.5 = 2.5\,\text{kWh}$. Zusammen sind das
   $12.5\,\text{kWh}$.
2. Wir übernehmen die Schleife mit den neuen Messwerten:

   ```python
   # Teilaufgabe 2: Messwerte der Wallbox, Schrittweite in h
   leistung_wallbox = np.array([11.0, 11.0, 7.0, 3.0])
   dt_wallbox = 0.5

   # Teilaufgabe 2: aufaddierte Energie mit der Trapezregel
   energie_wallbox = np.zeros(len(leistung_wallbox))
   for k in range(len(leistung_wallbox) - 1):
       trapez = (leistung_wallbox[k] + leistung_wallbox[k + 1]) / 2 * dt_wallbox
       energie_wallbox[k + 1] = energie_wallbox[k] + trapez
   print('Aufaddierte Energie in kWh:', energie_wallbox)
   ```

   Die Ausgabe `[ 0.   5.5 10.  12.5]` bestätigt die Handrechnung.
3. Jedes Trapez hat die Fläche $(P_k + P_{k+1})/2 \cdot \Delta t$. Das ist der
   Mittelwert aus dem Rechteck der Linkssumme $P_k\,\Delta t$ und dem Rechteck
   der Rechtssumme $P_{k+1}\,\Delta t$. Weil das für jedes Intervall gilt, gilt
   es auch für die Summe. Für die Schnellladesäule ergibt sich
   $(58.61 + 50.27)/2 = 54.44\,\text{kWh}$.
````

## Was kommt bei np.trapezoid() heraus?

NumPy bietet die Trapezregel als Funktion `np.trapezoid(y, x)` an. Als erstes
Argument erhält sie die Funktionswerte, als zweites die Stellen, zu denen sie
gehören. Die Schrittweite berechnet die Funktion selbst aus den Abständen der
Stellen. In NumPy-Versionen vor 2.0 heißt die Funktion `np.trapz()`. Wir
übergeben das Protokoll der Säule so, wie es vorliegt.

```{code-cell} python
# Trapezregel mit NumPy: erst die Funktionswerte, dann die Zeitpunkte
energie_np = np.trapezoid(leistung, t)
print(f'np.trapezoid(): {energie_np:.1f}')
```

Das Ergebnis $3266.5$ ist rund sechzigmal so groß wie das unserer Schleife.
*Hat NumPy falsch gerechnet?* Nein. Die Zeitpunkte in `t` stehen in Minuten,
also hat das Ergebnis die Einheit Kilowatt mal Minuten. Teilen wir durch 60,
erhalten wir wieder $54.44\,\text{kWh}$. In unserer Schleife haben wir die
Schrittweite in Stunden umgerechnet. Bei `np.trapezoid()` rechnen wir
stattdessen die Zeitpunkte um.

```{code-cell} python
# Zeitpunkte in h umrechnen, dann ergibt sich die Energie in kWh
t_h = t / 60
energie_np = np.trapezoid(leistung, t_h)
print(f'np.trapezoid(): {energie_np:.2f} kWh')
```

```{admonition} Stolperfalle: Einheit des Ergebnisses
:class: warning
`np.trapezoid()` kennt keine Einheiten. Das Ergebnis hat die Einheit der
Funktionswerte mal die Einheit der Stellen. Rechnen Sie die Stellen deshalb
vorher in die passende Einheit um, zum Beispiel Minuten in Stunden.
```

Die Trapezregel lag mit dem Fünf-Minuten-Protokoll $0.28\,\text{kWh}$ unter
dem Zähler. Die App kann das Protokoll auch im Minutentakt exportieren. Bei
$0, 5, 10, \dots, 30\,\text{min}$ enthält es dieselben Werte wie oben.

```{code-cell} python
# Protokoll im Minutentakt: Zeitpunkte in min und Ladeleistung in kW
t_fein = np.linspace(0, 30, 31)
leistung_fein = np.array([
    150.0, 150.0, 150.0, 150.0, 150.0, 150.0, 150.0, 150.0, 150.0, 150.0,
    150.0, 150.0, 150.0, 135.0, 122.7, 112.5, 103.8, 96.4, 90.0, 84.4,
    79.4, 75.0, 71.1, 67.5, 64.3, 61.4, 58.7, 56.3, 54.0, 51.9,
    50.0,
])
```

Für das feine Protokoll rechnen wir die Zeitpunkte ebenfalls in Stunden um.

```{code-cell} python
# Trapezregel für das Protokoll im Minutentakt
energie_fein = np.trapezoid(leistung_fein, t_fein / 60)
print(f'Minutentakt: {energie_fein:.2f} kWh')
```

Mit dem Minutentakt liefert die Trapezregel $54.74\,\text{kWh}$, nur noch
$0.02\,\text{kWh}$ mehr als der Zähler. Woher der größere Fehler im
Fünf-Minuten-Takt kommt, zeigt ein Diagramm beider Protokolle.

```{code-cell} python
# Diagramm: Protokoll im Minutentakt und im Fünf-Minuten-Takt
fig, ax = plt.subplots(figsize=(7, 4))
ax.plot(t_fein, leistung_fein, marker='.', label='Minutentakt')
ax.plot(t, leistung, marker='o', label='Fünf-Minuten-Takt')
ax.set_xlabel('Zeit in min')
ax.set_ylabel('Ladeleistung in kW')
ax.set_title('Ladekurve in zwei Auflösungen')
ax.legend()
ax.grid(True)
plt.show()
```

Bei $12\,\text{min}$ erreicht der Akku 50 Prozent Ladezustand, und die
Ladeelektronik beginnt zu drosseln. Die Ladekurve hat dort einen Knick. Das
Fünf-Minuten-Protokoll verbindet $150\,\text{kW}$ bei $10\,\text{min}$ direkt
mit $112.5\,\text{kW}$ bei $15\,\text{min}$ und schneidet die Ecke ab. Allein
in diesem Intervall fehlt dadurch gut eine halbe Kilowattstunde. Was zwischen
zwei Messwerten passiert, sieht die Trapezregel nicht. Je kleiner die
Schrittweite, desto weniger fällt das in der Regel ins Gewicht. Wie schnell
der Fehler mit der Schrittweite sinkt, untersuchen wir in Kapitel 9.3.

`np.trapezoid()` hat noch einen Vorteil gegenüber unserer Schleife: Die
Abstände der Stellen dürfen verschieden groß sein. Jedes Trapez erhält dann
seine eigene Breite.

```{admonition} Mini-Übung (✩)
:class: tip
1. Beantworten Sie ohne Code: Welchen Wert liefert
   `np.trapezoid(leistung, t) / 60`, und in welcher Einheit?
2. Berechnen Sie mit `np.trapezoid()` aus dem Protokoll im Minutentakt die
   Energie, die in den ersten $12\,\text{min}$ geladen wurde. `t_fein[:13]`
   liefert die ersten 13 Einträge, also die Zeitpunkte von $0$ bis
   $12\,\text{min}$. Welchen Ladezustand hat der Akku nach $12\,\text{min}$?
3. Beantworten Sie ohne Code: Nach $15\,\text{min}$ verläuft der Streckenzug
   des Fünf-Minuten-Protokolls über der Kurve des Minutentakts. Liefern die
   Trapeze dort zu viel oder zu wenig Energie?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
1. `np.trapezoid(leistung, t)` liefert $3266.5\,\text{kW} \cdot \text{min}$.
   Geteilt durch 60 ergibt das $54.44\,\text{kWh}$, denselben Wert wie mit
   den Zeitpunkten in Stunden.
2. Wir übergeben nur die ersten 13 Einträge beider Arrays:

   ```python
   # Teilaufgabe 2: Energie in den ersten 12 min
   energie_12 = np.trapezoid(leistung_fein[:13], t_fein[:13] / 60)

   # Teilaufgabe 2: Ladezustand nach 12 min, Start bei 10 Prozent
   ladezustand_12 = 10 + energie_12 / 75 * 100
   print(f'Energie nach 12 min:     {energie_12:.1f} kWh')
   print(f'Ladezustand nach 12 min: {ladezustand_12:.0f} Prozent')
   ```

   In den ersten $12\,\text{min}$ wird der Akku mit konstant $150\,\text{kW}$
   geladen, er nimmt also $150\,\text{kW} \cdot 0.2\,\text{h} = 30\,\text{kWh}$
   auf. Das sind 40 Prozent von $75\,\text{kWh}$. Der Akku hat dann 50 Prozent
   Ladezustand, genau dort beginnt die Ladeelektronik zu drosseln.
3. Liegt der Streckenzug über der Kurve, sind die Trapeze größer als die
   Fläche unter der Kurve. Die Trapezregel liefert dort zu viel Energie. Das
   gleicht einen Teil der fehlenden Energie zwischen $10$ und $15\,\text{min}$
   aus, deshalb liegt das Gesamtergebnis nur $0.28\,\text{kWh}$ unter dem
   Zähler.
````

## Zusammenfassung und Ausblick

Die Energie ist das Integral der Leistung über der Zeit, also die Fläche unter
der Ladekurve. Liegt die Leistung nur als Messreihe vor, nähern wir diese
Fläche an. Die Linkssumme und die Rechtssumme verwenden Rechtecke. Bei
fallender Leistung liefert die Linkssumme zu viel und die Rechtssumme zu wenig
Energie. Die Trapezregel verbindet die Messpunkte durch Strecken und ist der
Mittelwert aus beiden. Addieren wir die Trapeze in einer Schleife auf, erhalten
wir die bis zu jedem Zeitpunkt geladene Energie. NumPy bietet die Trapezregel
als `np.trapezoid()`. Das Ergebnis hat die Einheit der Funktionswerte mal die
Einheit der Stellen, deshalb rechnen wir Minuten vorher in Stunden um.

In Kapitel 9.2 bleibt das E-Auto länger an der Säule. Mit der aufaddierten
Energie berechnen Sie, wann der Akku 80 Prozent erreicht, und entscheiden, ob
es sich lohnt, bis 95 Prozent weiterzuladen.
