---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 7.3 Übungen

Diese Aufgaben sind für das Selbststudium zuhause gedacht und wiederholen den
Stoff der Kapitel 7.1 und 7.2. Rechnen Sie mit rund 45 Minuten
Bearbeitungszeit. Führen Sie vor den Aufgaben die folgende Zelle aus. Sie
enthält die Funktion `ableitung()` aus Kapitel 7.1 sowie die Funktionen
`fahrprofil()` und `geschwindigkeit()` der sanften Anfahrt aus Kapitel 7.2.

Der Schwierigkeitsgrad steht im Titel jeder Aufgabe:

* ✩ Verständnis: Code und Ausgaben vorhersagen und erklären (ca. 7 min)
* ✩✩ Anwendung: eigenen Code schreiben und Ergebnisse interpretieren (ca. 25 min)

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Funktion aus Kapitel 7.1
def ableitung(werte, dt):
    """Ableitung einer Messreihe mit Schrittweite dt.

    Innen zentraler Differenzenquotient, am ersten Punkt vorwärts,
    am letzten Punkt rückwärts.
    """
    n = len(werte)
    ergebnis = np.zeros(n)
    ergebnis[0] = (werte[1] - werte[0]) / dt
    ergebnis[-1] = (werte[-1] - werte[-2]) / dt
    for k in range(1, n - 1):
        ergebnis[k] = (werte[k + 1] - werte[k - 1]) / (2 * dt)
    return ergebnis

# Funktionen aus Kapitel 7.2
def fahrprofil(t):
    """Strecke in m bei der sanften Anfahrt, t in s von 0 bis 2."""
    return 0.4 * (t - 2 / np.pi * np.sin(np.pi * t / 2))

def geschwindigkeit(t):
    """Exakte Geschwindigkeit in m/s, die Ableitung von fahrprofil."""
    return 0.4 * (1 - np.cos(np.pi * t / 2))
```

````{admonition} Aufgabe 7.1 (✩)
:class: tip
Bei der **Abschnittskontrolle** auf Autobahnen fotografieren Kameras jedes
Fahrzeug an mehreren Kontrollpunkten. Aus der Strecke zwischen zwei
Kontrollpunkten und der Zeit zwischen den beiden Aufnahmen berechnet die Anlage
die Geschwindigkeit. Gegeben ist folgender Code für ein Auto, das drei
Kontrollpunkte passiert:

```python
import numpy as np

# Lage der Kontrollpunkte in km und Durchfahrtszeiten in min
lage = np.array([0.0, 6.0, 15.0])
zeit = np.array([0.0, 2.5, 7.0])

# Geschwindigkeit auf jedem Abschnitt in km/h
v_abschnitt = np.zeros(len(lage) - 1)
for k in range(len(lage) - 1):
    v_abschnitt[k] = (lage[k + 1] - lage[k]) / (zeit[k + 1] - zeit[k]) * 60
print(v_abschnitt)

# Geschwindigkeit über die ganze Strecke in km/h
v_gesamt = (lage[-1] - lage[0]) / (zeit[-1] - zeit[0]) * 60
print(round(v_gesamt, 1))
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Was geben die beiden `print()`-Zeilen aus? Wozu dient der Faktor `60`?
2. Erlaubt sind $130\,\text{km/h}$. Erhält der Fahrer einen Bußgeldbescheid,
   wenn die Anlage beide Abschnitte einzeln auswertet? Und wenn sie nur die
   ganze Strecke auswertet?
3. Auf dem zweiten Abschnitt ergibt sich $120\,\text{km/h}$. Kann der Fahrer
   dort trotzdem schneller als $130\,\text{km/h}$ gefahren sein? Begründen Sie
   mit den Begriffen Differenzenquotient und Ableitung.
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

lage = np.array([0.0, 6.0, 15.0])
zeit = np.array([0.0, 2.5, 7.0])

v_abschnitt = np.zeros(len(lage) - 1)
for k in range(len(lage) - 1):
    v_abschnitt[k] = (lage[k + 1] - lage[k]) / (zeit[k + 1] - zeit[k]) * 60
print(v_abschnitt)

v_gesamt = (lage[-1] - lage[0]) / (zeit[-1] - zeit[0]) * 60
print(round(v_gesamt, 1))
```
Ausgabe:

```text
[144. 120.]
128.6
```

1. Die erste Zeile gibt die Geschwindigkeiten auf den beiden Abschnitten aus,
   die zweite die Geschwindigkeit über die ganze Strecke. Jeder Wert ist ein
   Differenzenquotient wie in Kapitel 7.1: Strecke durch Zeit. Weil die Zeit
   in Minuten vorliegt, ergibt er Kilometer pro Minute, und der Faktor 60
   rechnet in Kilometer pro Stunde um.
2. Auf dem ersten Abschnitt fährt das Auto $6\,\text{km}$ in
   $2.5\,\text{min}$, also im Mittel $144\,\text{km/h}$. Wertet die Anlage die
   Abschnitte einzeln aus, gibt es einen Bußgeldbescheid. Über die ganze
   Strecke ergeben sich nur $128.6\,\text{km/h}$, der zu schnelle erste
   Abschnitt fällt im Mittel nicht mehr auf. Das ist nicht der Mittelwert
   $(144 + 120)/2 = 132\,\text{km/h}$ der beiden Abschnitte. Das Auto ist auf
   dem langsameren Abschnitt $4.5\,\text{min}$ unterwegs, auf dem schnelleren
   nur $2.5\,\text{min}$. Der langsamere Abschnitt zählt deshalb stärker.
3. Ja. Auch die $120\,\text{km/h}$ auf dem zweiten Abschnitt sind eine
   mittlere Geschwindigkeit, ein Differenzenquotient über $4.5\,\text{min}$.
   Die momentane Geschwindigkeit, die Ableitung der Strecke nach der Zeit,
   kann zwischendurch deutlich höher gewesen sein. Die Abschnittskontrolle
   erkennt das nicht.
````

````{admonition} Aufgabe 7.2 (✩)
:class: tip
Das FTF lädt an seiner Ladestation. Die Ladestation protokolliert alle
$10\,\text{min}$ den Ladezustand des Akkus in Prozent. Gegeben ist folgender
Code:

```python
# Ladezustand in Prozent bei 0, 10, 20, 30 und 40 min
dt = 10
ladezustand = np.array([50.0, 65.0, 80.0, 90.0, 95.0])
print(ableitung(ladezustand, dt))
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Mit welchem Differenzenquotienten berechnet `ableitung()` den ersten, den
   zweiten und den letzten Wert? Was gibt die `print()`-Zeile aus, und in
   welcher Einheit?
2. Zu welchem Zeitpunkt passt der letzte Wert am besten?
3. Was sagt der Verlauf der Werte über das Laden aus?
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
dt = 10
ladezustand = np.array([50.0, 65.0, 80.0, 90.0, 95.0])
print(ableitung(ladezustand, dt))
```
Ausgabe:

```text
[1.5  1.5  1.25 0.75 0.5 ]
```

1. Der erste Wert ist der Vorwärtsdifferenzenquotient
   $(65 - 50)/10 = 1.5$, der zweite der zentrale Differenzenquotient
   $(80 - 50)/20 = 1.5$ und der letzte der Rückwärtsdifferenzenquotient
   $(95 - 90)/10 = 0.5$. Die Werte geben an, um wie viele Prozentpunkte der
   Ladezustand pro Minute steigt, die Einheit ist also Prozentpunkte pro
   Minute. Steigt der Ladezustand von 50 auf 65 Prozent, sind das 15
   Prozentpunkte, die Ladung im Akku wächst dabei aber um 30 Prozent. Die
   ersten beiden Werte sind gleich, weil der Ladezustand in beiden
   Intervallen der ersten $20\,\text{min}$ um je 15 Prozentpunkte steigt.
2. Der letzte Wert ist die mittlere Laderate von $30$ bis $40\,\text{min}$. Er
   passt deshalb am besten zu $35\,\text{min}$, der Mitte des Intervalls, und
   nicht zu $40\,\text{min}$. Genau gilt das, wenn sich die Laderate in diesem
   Intervall gleichmäßig ändert. Wie sie zwischen den Messungen verläuft,
   wissen wir nicht.
3. Die berechneten Laderaten sinken, je voller der Akku ist. Das ist typisch:
   Ab etwa 80
   Prozent verringert die Ladeelektronik den Ladestrom, um den Akku zu
   schonen.
````

````{admonition} Aufgabe 7.3 (✩)
:class: tip
Gegeben ist folgender Code für die sanfte Anfahrt aus Kapitel 7.2:

```python
def rueckwaerts(funktion, t, dt):
    """Rückwärtsdifferenzenquotient von funktion an der Stelle t."""
    return (funktion(t) - funktion(t - dt)) / dt

t_0 = 1.0
exakt = geschwindigkeit(t_0)
for dt in [0.1, 0.05, 0.025]:
    abweichung = rueckwaerts(fahrprofil, t_0, dt) - exakt
    print(f'dt = {dt:5}: Abweichung {abweichung:+.5f} m/s')
```

Notieren Sie Ihre Vermutung in einer Markdown-Zelle, bevor Sie den Code
ausführen.

1. Entwickeln Sie $s(t - \Delta t)$ mit der Taylor-Entwicklung aus Kapitel
   7.2 bis zum Term mit $\Delta t^2$. Ersetzen Sie dazu in der Entwicklung von
   $s(t + \Delta t)$ überall $\Delta t$ durch $-\Delta t$. Setzen Sie das
   Ergebnis in den Rückwärtsdifferenzenquotienten
   $(s(t) - s(t - \Delta t))/\Delta t$ ein. Welcher Fehlerterm bleibt als
   größter übrig?
2. Bei $t = 1\,\text{s}$ ist die Beschleunigung am größten, sie beträgt
   $s''(1) = 0.628\,\text{m/s}^2$. Welche Abweichungen mit Vorzeichen sagen
   Sie für die drei Schrittweiten voraus?
3. Um welchen Faktor ändert sich die Abweichung von Zeile zu Zeile?
4. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
def rueckwaerts(funktion, t, dt):
    """Rückwärtsdifferenzenquotient von funktion an der Stelle t."""
    return (funktion(t) - funktion(t - dt)) / dt

t_0 = 1.0
exakt = geschwindigkeit(t_0)
for dt in [0.1, 0.05, 0.025]:
    abweichung = rueckwaerts(fahrprofil, t_0, dt) - exakt
    print(f'dt = {dt:5}: Abweichung {abweichung:+.5f} m/s')
```
Ausgabe:

```text
dt =   0.1: Abweichung -0.03135 m/s
dt =  0.05: Abweichung -0.01570 m/s
dt = 0.025: Abweichung -0.00785 m/s
```

1. Mit $-\Delta t$ statt $\Delta t$ wechseln die ungeraden Potenzen von
   $\Delta t$ ihr Vorzeichen:

   $$s(t - \Delta t) = s(t) - \Delta t \cdot s'(t) + \frac{\Delta t^2}{2}
   \cdot s''(t) - \ldots$$

   Eingesetzt in den Rückwärtsdifferenzenquotienten ergibt sich

   $$\frac{s(t) - s(t - \Delta t)}{\Delta t} = s'(t) - \frac{\Delta t}{2}
   \cdot s''(t) + \ldots$$

   Der größte Fehlerterm ist $-\frac{\Delta t}{2} \cdot s''(t)$. Er hat
   denselben Betrag wie beim Vorwärtsdifferenzenquotienten, aber das
   umgekehrte Vorzeichen.
2. Mit $s''(1) = 0.628\,\text{m/s}^2$ sagt der Fehlerterm die Abweichungen
   $-0.0314$, $-0.0157$ und $-0.0079\,\text{m/s}$ voraus. Die berechneten
   Werte stimmen damit fast genau überein. Bei $t = 1\,\text{s}$ ist der Ruck
   null, deshalb fällt auch der nächste Term der Taylor-Entwicklung weg. Das
   negative Vorzeichen passt zu Kapitel 7.1: Der Rückwärtsdifferenzenquotient
   mittelt über die Zeit vor $t = 1\,\text{s}$, als das FTF noch langsamer
   war.
3. Halbiert sich die Schrittweite, halbiert sich auch die Abweichung. Der
   Fehler ist also proportional zu $\Delta t$, die Fehlerordnung ist 1 wie
   beim Vorwärtsdifferenzenquotienten.
````

````{admonition} Aufgabe 7.4 (✩✩)
:class: tip
Ein FTF fährt geradeaus durch die Halle. Bei $t = 0.4\,\text{s}$ meldet der
Sicherheitsscanner aus Kapitel 6.2 ein Hindernis im Schutzfeld, und das FTF
bremst voll. Auf der Ladefläche steht eine ungesicherte Gitterbox aus Stahl.
Mit der Haftreibungszahl $\mu = 0.2$ rutscht sie, sobald die Verzögerung
größer als
$\mu \cdot g = 0.2 \cdot 9.81\,\text{m/s}^2 \approx 1.96\,\text{m/s}^2$ ist.
Das FTF hat seine Position während der Fahrt zehnmal pro Sekunde
protokolliert:

```python
# Schrittweite in s und Zeitpunkte 0.0, 0.1, ..., 1.5 s
dt = 0.1
t = np.linspace(0, 1.5, 16)
# zurückgelegte Strecke in m, auf Millimeter gerundet
s = np.array([0.0, 0.1, 0.2, 0.3, 0.4, 0.488, 0.55, 0.588,
              0.6, 0.6, 0.6, 0.6, 0.6, 0.6, 0.6, 0.6])
```

1. Berechnen Sie mit `ableitung()` die Geschwindigkeit. Wie schnell war das
   FTF vor der Bremsung? Wie lang ist der Bremsweg vom Bremsbeginn bei
   $t = 0.4\,\text{s}$ bis zum Stillstand?
2. Berechnen Sie die Beschleunigung und zeichnen Sie Geschwindigkeit und
   Beschleunigung in zwei Subplots untereinander. Zeichnen Sie die Grenze, ab
   der die Gitterbox rutscht, als waagerechte Linie ein, wie die geplante
   Beschleunigung in Kapitel 7.1. Bestimmen Sie die größte Verzögerung mit
   `np.min()`. Rutscht die Gitterbox?
3. Beantworten Sie ohne Code: Das FTF bremst schlagartig voll und mit
   gleichbleibender Verzögerung. Warum steigt die berechnete Verzögerung
   trotzdem über mehrere Zeitpunkte allmählich an und fällt wieder ab?
4. Bei gleichbleibender Verzögerung $a$ beträgt der Bremsweg $v_0^2 / (2a)$.
   Prüfen Sie die Formel mit Ihren Ergebnissen aus den Teilaufgaben 1 und 2.
   Die Leitsteuerung soll die Verzögerung auf $1.96\,\text{m/s}^2$ begrenzen,
   damit die Gitterbox nicht rutscht. Wie lang wird der Bremsweg dann, und was
   bedeutet das für das Schutzfeld?

Strukturieren Sie Ihren Code mit EVA-Kommentaren.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
# Eingabe: Protokoll der Fahrt und Grenze für die Gitterbox
dt = 0.1
t = np.linspace(0, 1.5, 16)
s = np.array([0.0, 0.1, 0.2, 0.3, 0.4, 0.488, 0.55, 0.588,
              0.6, 0.6, 0.6, 0.6, 0.6, 0.6, 0.6, 0.6])
grenze = 0.2 * 9.81    # größte Verzögerung ohne Rutschen in m/s²

# Verarbeitung: Geschwindigkeit und Bremsweg ab t = 0.4 s (Index 4)
v = ableitung(s, dt)
v_0 = v[0]
bremsweg = s[-1] - s[4]

# Verarbeitung: Beschleunigung und größte Verzögerung
a = ableitung(v, dt)
verzoegerung_max = -np.min(a)

# Verarbeitung: Bremsweg nach der Formel, gemessen und begrenzt
bremsweg_formel = v_0**2 / (2 * verzoegerung_max)
bremsweg_begrenzt = v_0**2 / (2 * grenze)

# Ausgabe
print(f'Geschwindigkeit vor der Bremsung: {v_0:.2f} m/s')
print(f'Bremsweg: {bremsweg:.3f} m')
print(f'Größte Verzögerung: {verzoegerung_max:.2f} m/s²')
if verzoegerung_max > grenze:
    print('Die Gitterbox rutscht.')
else:
    print('Die Gitterbox bleibt stehen.')
print(f'Bremsweg bei {verzoegerung_max:.2f} m/s²: {bremsweg_formel:.3f} m')
print(f'Bremsweg bei {grenze:.2f} m/s²: {bremsweg_begrenzt:.3f} m')

fig, ax = plt.subplots(nrows=2, ncols=1, figsize=(7, 6))
ax[0].plot(t, v, marker='o')
ax[0].set_ylabel('Geschwindigkeit in m/s')
ax[1].plot(t, a, marker='o', label='aus Messung')
# Grenze als Linie von t = 0 s bis t = 1.5 s
ax[1].plot([0, 1.5], [-grenze, -grenze], color='gray', linestyle='--',
           label='Grenze: Gitterbox rutscht')
ax[1].set_ylabel('Beschleunigung in m/s²')
ax[1].legend()
for einzelachse in ax:
    einzelachse.set_xlabel('Zeit in s')
    einzelachse.grid(True)
ax[0].set_title('Notbremsung des FTF')
plt.tight_layout()
plt.show()
```
Ausgabe:

```text
Geschwindigkeit vor der Bremsung: 1.00 m/s
Bremsweg: 0.200 m
Größte Verzögerung: 2.50 m/s²
Die Gitterbox rutscht.
Bremsweg bei 2.50 m/s²: 0.200 m
Bremsweg bei 1.96 m/s²: 0.255 m
```

1. Das FTF fährt vor der Bremsung mit $1.00\,\text{m/s}$. Der Bremsweg ist die
   Strecke vom Bremsbeginn bei Index 4 bis zum Stillstand, also
   $0.200\,\text{m}$.
2. Die größte Verzögerung beträgt $2.50\,\text{m/s}^2$ und liegt über der
   Grenze von $1.96\,\text{m/s}^2$. Die ungesicherte Gitterbox rutscht also.
   Im Diagramm liegt die Beschleunigung bei $0.5$, $0.6$ und $0.7\,\text{s}$
   unter der grauen Linie. Die Linie zeichnen wir wie die geplante
   Beschleunigung in Kapitel 7.1 mit `ax.plot()` durch zwei Punkte.
3. Jeder Wert der Geschwindigkeit verwendet die Positionen $0.1\,\text{s}$
   davor und danach, jeder Wert der Beschleunigung also Positionen von
   $0.2\,\text{s}$ davor bis $0.2\,\text{s}$ danach. Die Bremsung dauert nur
   $0.4\,\text{s}$. Nur bei $t = 0.6\,\text{s}$ liegt dieses Fenster ganz in
   der Bremsung, und nur dort erscheint die volle Verzögerung von
   $2.5\,\text{m/s}^2$. An den anderen Zeitpunkten mischt das Fenster Fahrt
   und Bremsung oder Bremsung und Stillstand, die berechnete Verzögerung ist
   dort kleiner. Den Sprung der Beschleunigung verschmiert das zweifache
   Ableiten wie den Knick in Kapitel 7.1.
4. Mit $v_0 = 1\,\text{m/s}$ und $a = 2.5\,\text{m/s}^2$ liefert die Formel
   $0.200\,\text{m}$, genau den Bremsweg aus Teilaufgabe 1. Begrenzt die
   Leitsteuerung die Verzögerung auf $1.96\,\text{m/s}^2$, wird der Bremsweg
   mit $0.255\,\text{m}$ gut $5\,\text{cm}$ länger. Das FTF muss also früher
   mit dem Bremsen beginnen, das Schutzfeld muss weiter nach vorne reichen.
   Das Schutzfeld aus Kapitel 6.2 reicht $1\,\text{m}$ vor die Vorderkante.
   In unserem vereinfachten Modell ohne Reaktionszeit genügt das noch.
````
