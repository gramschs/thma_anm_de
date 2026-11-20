---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 8.2 Abkühlkonstante aus Messdaten

In Kapitel 8.1 hat uns die Betriebswirtschaft die Kostenfunktion fertig
geliefert. Jetzt stellen Sie selbst eine Kostenfunktion auf. Sie misst, wie
gut ein Modell zu Messdaten passt, und das Gradientenverfahren sucht den
Parameter, bei dem Modell und Messung am besten übereinstimmen. Bearbeiten Sie
die Teile möglichst zu zweit und der Reihe nach.

````{admonition} Projekt: Wann darf das Teil in den Karton? (✩✩)
:class: tip
Ein Betrieb wäscht Aluminiumteile und trocknet sie danach in einem Ofen bei
$100\,°\text{C}$. Anschließend kühlen die Teile auf einem Förderband mit
Lüfter in der $20\,°\text{C}$ warmen Halle ab. In Folie verpackt werden dürfen
sie erst unter $40\,°\text{C}$, sonst schlägt sich Kondenswasser in der
Verpackung nieder. Ein Thermoelement hat an einem Teil alle $5\,\text{s}$ die
Temperatur gemessen.

Für die Temperatur $T$ zur Zeit $t$ verwenden wir das **newtonsche
Abkühlungsgesetz**:

$$T(t) = T_\infty + (T_0 - T_\infty) \cdot e^{-\lambda t}.$$

| Größe | Bedeutung | Wert |
| --- | --- | --- |
| $T_0$ | Temperatur beim Verlassen des Ofens | $100\,°\text{C}$ |
| $T_\infty$ | Temperatur der Halle | $20\,°\text{C}$ |
| $\lambda$ | **Abkühlkonstante**, hängt von Form, Material und Luftstrom ab | unbekannt, in $1/\text{s}$ |
| Grenze | höchste Temperatur beim Verpacken | $40\,°\text{C}$ |

Gesucht sind die Abkühlkonstante $\lambda$ und die Zeit, die das Teil auf dem
Band bleiben muss.
````

Führen Sie zuerst die folgende Zelle aus. Sie enthält die Funktionen
`zentral()` und `gradientenverfahren()` aus Kapitel 8.1, die bekannten
Temperaturen und die Messdaten.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Funktionen aus Kapitel 8.1
def zentral(funktion, stelle, dx):
    """Zentraler Differenzenquotient von funktion an der Stelle stelle."""
    return (funktion(stelle + dx) - funktion(stelle - dx)) / (2 * dx)

def gradientenverfahren(funktion, start, lernrate, toleranz, max_iterationen):
    """Sucht ein Minimum von funktion, Start bei start.

    Gibt alle besuchten Stellen als Array zurück.
    """
    stelle = start
    verlauf = [stelle]
    for k in range(max_iterationen):
        steigung = zentral(funktion, stelle, 1e-6)
        # Abbruchkriterium: Steigung fast null, Minimum erreicht
        if abs(steigung) < toleranz:
            break
        stelle = stelle - lernrate * steigung
        verlauf.append(stelle)
    return np.array(verlauf)

# Bekannte Temperaturen in °C
T_start = 100.0
T_halle = 20.0

# Messzeitpunkte in s und gemessene Temperaturen in °C
t_mess = np.array([0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60])
T_mess = np.array([100.0, 82.6, 68.2, 56.9, 49.0, 41.9, 37.9, 35.2, 30.3,
                   27.8, 27.1, 25.5, 24.1])
```

```{admonition} Teil 1: Modell und erste Schätzung
:class: tip
Schreiben Sie wie `gesamtkosten()` in Kapitel 8.1 eine Funktion
`modell(t, lam)`, die die Temperatur nach dem Abkühlungsgesetz berechnet. Das
Wort `lambda` ist in Python für etwas anderes reserviert, deshalb heißt der
Parameter `lam`.

Zeichnen Sie die Messwerte als Punkte und das Modell für $\lambda = 0.03$ und
$\lambda = 0.08\,\text{1/s}$ als Linien. Für glatte Linien werten Sie das
Modell an 200 Zeitpunkten zwischen $0$ und $60\,\text{s}$ aus.

Welche der beiden Schätzungen kühlt zu langsam ab, welche zu schnell? Wo etwa
liegt das gesuchte $\lambda$?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 1
:class: tip
:class: dropdown
Die Funktion ist gebaut wie `gesamtkosten()` in Kapitel 8.1. Weil `np.exp()`
mit Arrays rechnet, funktioniert `modell()` auch für ein Array von Zeitpunkten.
Punkte zeichnet `ax.scatter()`.
```python
def modell(t, lam):
    """Temperatur in °C zur Zeit t nach dem Abkühlungsgesetz."""
    return ___

# 200 Zeitpunkte für glatte Modellkurven
t_fein = np.linspace(___, ___, 200)

fig, ax = plt.subplots(figsize=(7, 4))
ax.scatter(t_mess, T_mess, color='black', label='Messwerte')
for lam in [0.03, 0.08]:
    ax.plot(t_fein, ___, label=f'Modell mit λ = {lam} 1/s')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Temperatur in °C')
ax.set_title('Abkühlung eines Aluminiumteils')
ax.legend()
ax.grid(True)
plt.show()
```
````

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
```python
def modell(t, lam):
    """Temperatur in °C zur Zeit t nach dem Abkühlungsgesetz."""
    return T_halle + (T_start - T_halle) * np.exp(-lam * t)

# 200 Zeitpunkte für glatte Modellkurven
t_fein = np.linspace(0, 60, 200)

fig, ax = plt.subplots(figsize=(7, 4))
ax.scatter(t_mess, T_mess, color='black', label='Messwerte')
for lam in [0.03, 0.08]:
    ax.plot(t_fein, modell(t_fein, lam), label=f'Modell mit λ = {lam} 1/s')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Temperatur in °C')
ax.set_title('Abkühlung eines Aluminiumteils')
ax.legend()
ax.grid(True)
plt.show()
```
Mit $\lambda = 0.03\,\text{1/s}$ liegt die Modellkurve über den Messwerten,
das Modell kühlt zu langsam ab. Mit $\lambda = 0.08\,\text{1/s}$ liegt sie
darunter, das Modell kühlt zu schnell ab. Je größer $\lambda$, desto schneller
nähert sich die Temperatur der Hallentemperatur. Das gesuchte $\lambda$ liegt
also zwischen den beiden Schätzungen, nach Augenmaß etwa bei
$0.05\,\text{1/s}$.
````

````{admonition} Teil 2: Kostenfunktion aus den Messdaten
:class: tip
Wie gut ein $\lambda$ passt, messen wir an den Abweichungen zwischen Messwert
und Modell. Wir quadrieren jede Abweichung und bilden den Mittelwert über alle
$n$ Messungen. Das ergibt die **mittlere quadratische Abweichung** (englisch
mean squared error, kurz MSE):

$$K(\lambda) = \frac{1}{n} \sum_{i=1}^{n} \bigl(T_{\text{mess},i} - T(t_i)\bigr)^2.$$

Durch das Quadrieren heben sich positive und negative Abweichungen nicht
gegenseitig auf, und große Abweichungen zählen stärker als kleine.

Schreiben Sie eine Funktion `kosten(lam)`, die $K(\lambda)$ mit `np.mean()`
berechnet. Werten Sie sie für 300 Werte von $\lambda$ zwischen $0.01$ und
$0.12\,\text{1/s}$ aus und zeichnen Sie $K$ über $\lambda$. Anders als
`gesamtkosten()` in Kapitel 8.1 verarbeitet `kosten()` nur eine einzelne Zahl
$\lambda$, denn das Array `t_mess` steckt schon in der Funktion. Die 300 Werte
berechnen Sie deshalb in einer Schleife.

Wo etwa liegt das Minimum? Welche Einheit hat $K$?
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 2
:class: tip
:class: dropdown
`modell(t_mess, lam)` liefert die Modellwerte an allen 13 Messzeitpunkten auf
einmal. Die 300 Werte von $K$ sammeln wir wie in Kapitel 7.1 in einem Array,
das wir vorher mit `np.zeros()` anlegen.
```python
def kosten(lam):
    """Mittlere quadratische Abweichung in °C² zwischen Messung und Modell."""
    abweichung = ___
    return np.mean(___)

# K für 300 Werte von lambda
lam_werte = np.linspace(___, ___, 300)
kosten_werte = np.zeros(len(lam_werte))
for k in range(len(lam_werte)):
    kosten_werte[k] = ___

fig, ax = plt.subplots(figsize=(7, 4))
ax.plot(lam_werte, kosten_werte)
ax.set_xlabel('λ in 1/s')
ax.set_ylabel('K in °C²')
ax.set_title('Mittlere quadratische Abweichung')
ax.grid(True)
plt.show()
```
````

````{admonition} Lösung Teil 2
:class: tip
:class: dropdown
```python
def kosten(lam):
    """Mittlere quadratische Abweichung in °C² zwischen Messung und Modell."""
    abweichung = T_mess - modell(t_mess, lam)
    return np.mean(abweichung**2)

# K für 300 Werte von lambda
lam_werte = np.linspace(0.01, 0.12, 300)
kosten_werte = np.zeros(len(lam_werte))
for k in range(len(lam_werte)):
    kosten_werte[k] = kosten(lam_werte[k])

fig, ax = plt.subplots(figsize=(7, 4))
ax.plot(lam_werte, kosten_werte)
ax.set_xlabel('λ in 1/s')
ax.set_ylabel('K in °C²')
ax.set_title('Mittlere quadratische Abweichung')
ax.grid(True)
plt.show()
```
Die Kostenfunktion hat ein deutliches Minimum bei etwa $0.05\,\text{1/s}$. Wir
mitteln quadrierte Temperaturdifferenzen, $K$ hat also die Einheit
$°\text{C}^2$. Die Form erinnert an Kapitel 8.1: Links vom Minimum fällt die
Kurve steil, rechts steigt sie flacher an.
````

````{admonition} Teil 3: Gradientenverfahren
:class: tip
Suchen Sie das Minimum mit `gradientenverfahren()`, Start bei
$\lambda = 0.03\,\text{1/s}$, Toleranz $10^{-3}$ und höchstens 100
Iterationen.

Die Lernrate 200 aus Kapitel 8.1 passt hier nicht. Berechnen Sie zuerst mit
`zentral()` die Steigung von `kosten()` beim Startwert. Der erste Schritt soll
$\lambda$ um etwa $0.02\,\text{1/s}$ ändern. Welche Lernrate folgt daraus?

Vergleichen Sie dann die Lernraten $10^{-6}$, $2 \cdot 10^{-6}$,
$5 \cdot 10^{-6}$ und $10^{-5}$. Das Konvergenzdiagramm dafür ist schon
vorbereitet, Sie ergänzen nur die Lücken:

```python
# Konvergenzdiagramm für vier Lernraten
fig, ax = plt.subplots(figsize=(7, 4))
for lernrate in [1e-6, 2e-6, 5e-6, 1e-5]:
    verlauf = gradientenverfahren(___, ___, lernrate, 1e-3, 100)
    anzahl = len(verlauf) - 1
    print(f'Lernrate {lernrate:.0e}: {anzahl} Iterationen, '
          f'λ = {verlauf[-1]:.5f} 1/s')
    ax.plot(verlauf, marker='.', label=f'Lernrate {lernrate:.0e}')
ax.set_xlim(0, 40)
ax.set_ylim(0, 0.15)
ax.set_xlabel('Iteration')
ax.set_ylabel('λ in 1/s')
ax.set_title('Konvergenz des Gradientenverfahrens')
ax.legend()
ax.grid(True)
plt.show()
```

Welches $\lambda$ finden Sie, und welche Lernrate kommt mit den wenigsten
Iterationen aus?
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 3
:class: tip
:class: dropdown
Ein Schritt ist das Produkt aus Lernrate und Steigung. Soll er
$0.02\,\text{1/s}$ lang sein, teilen wir $0.02$ durch den Betrag der Steigung.
```python
# Steigung beim Startwert und passende Lernrate
steigung_start = zentral(___, ___, 1e-6)
lernrate_schaetzung = ___ / abs(steigung_start)
print(f'Steigung beim Start: {steigung_start:.0f} °C² s')
print(f'Geschätzte Lernrate: {lernrate_schaetzung:.1e}')
```
In das Gerüst für das Konvergenzdiagramm setzen Sie die Kostenfunktion und
den Startwert ein.
````

````{admonition} Lösung Teil 3
:class: tip
:class: dropdown
```python
# Steigung beim Startwert und passende Lernrate
steigung_start = zentral(kosten, 0.03, 1e-6)
lernrate_schaetzung = 0.02 / abs(steigung_start)
print(f'Steigung beim Start: {steigung_start:.0f} °C² s')
print(f'Geschätzte Lernrate: {lernrate_schaetzung:.1e}')

# Konvergenzdiagramm für vier Lernraten
fig, ax = plt.subplots(figsize=(7, 4))
for lernrate in [1e-6, 2e-6, 5e-6, 1e-5]:
    verlauf = gradientenverfahren(kosten, 0.03, lernrate, 1e-3, 100)
    anzahl = len(verlauf) - 1
    print(f'Lernrate {lernrate:.0e}: {anzahl} Iterationen, '
          f'λ = {verlauf[-1]:.5f} 1/s')
    ax.plot(verlauf, marker='.', label=f'Lernrate {lernrate:.0e}')
ax.set_xlim(0, 40)
ax.set_ylim(0, 0.15)
ax.set_xlabel('Iteration')
ax.set_ylabel('λ in 1/s')
ax.set_title('Konvergenz des Gradientenverfahrens')
ax.legend()
ax.grid(True)
plt.show()
```
Ausgabe:

```text
Steigung beim Start: -19224 °C² s
Geschätzte Lernrate: 1.0e-06
Lernrate 1e-06: 30 Iterationen, λ = 0.05038 1/s
Lernrate 2e-06: 15 Iterationen, λ = 0.05038 1/s
Lernrate 5e-06: 74 Iterationen, λ = 0.05038 1/s
Lernrate 1e-05: 100 Iterationen, λ = 0.12026 1/s
```
Die Steigung ist sehr groß, $\lambda$ selbst aber klein. Damit ein Schritt
nur einige Hundertstel lang wird, muss die Lernrate deshalb winzig sein. Die
Abschätzung liefert $10^{-6}$. Drei der vier Lernraten finden
$\lambda = 0.0504\,\text{1/s}$, am schnellsten $2 \cdot 10^{-6}$ mit 15
Iterationen. Mit $5 \cdot 10^{-6}$ springt das Verfahren im ersten Schritt
auf $0.126\,\text{1/s}$, weit über das Minimum hinaus, und braucht lange für
den Rückweg. Mit $10^{-5}$ springt es ungeordnet hin und her und erreicht das
Abbruchkriterium in 100 Iterationen nicht, das Ergebnis ist unbrauchbar.
````

````{admonition} Teil 4: Ergebnis prüfen und Wartezeit berechnen
:class: tip
Speichern Sie das gefundene $\lambda$ in einer Variablen `lam_optimal` und
zeichnen Sie das Modell mit diesem Wert zusammen mit den Messwerten.

Löst man das Abkühlungsgesetz nach der Zeit auf, ergibt sich die Zeit, nach
der das Teil die Grenze von $40\,°\text{C}$ erreicht:

$$t_{40} = \frac{1}{\lambda} \cdot \ln \frac{T_0 - T_\infty}{40\,°\text{C} - T_\infty}.$$

Den natürlichen Logarithmus berechnet `np.log()`. Wie lange muss das Teil auf
dem Band bleiben?
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Tipp Teil 4
:class: tip
:class: dropdown
Der letzte Eintrag von `verlauf` ist das gefundene $\lambda$. Die Zeichnung
entsteht wie in Teil 1, nur mit einer einzigen Modellkurve.
```python
# Bestes lambda aus dem Gradientenverfahren
verlauf = gradientenverfahren(kosten, 0.03, 2e-6, 1e-3, 100)
lam_optimal = ___

fig, ax = plt.subplots(figsize=(7, 4))
ax.scatter(t_mess, T_mess, color='black', label='Messwerte')
ax.plot(t_fein, ___, label=f'Modell mit λ = {lam_optimal:.4f} 1/s')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Temperatur in °C')
ax.set_title('Angepasstes Abkühlungsmodell')
ax.legend()
ax.grid(True)
plt.show()

# Zeit bis 40 °C und Länge der Kühlstrecke
t_40 = ___
strecke = ___
print(f'Zeit bis 40 °C: {t_40:.1f} s')
print(f'Kühlstrecke:    {strecke:.2f} m')
```
````

````{admonition} Lösung Teil 4
:class: tip
:class: dropdown
```python
# Bestes lambda aus dem Gradientenverfahren
verlauf = gradientenverfahren(kosten, 0.03, 2e-6, 1e-3, 100)
lam_optimal = verlauf[-1]

fig, ax = plt.subplots(figsize=(7, 4))
ax.scatter(t_mess, T_mess, color='black', label='Messwerte')
ax.plot(t_fein, modell(t_fein, lam_optimal),
        label=f'Modell mit λ = {lam_optimal:.4f} 1/s')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Temperatur in °C')
ax.set_title('Angepasstes Abkühlungsmodell')
ax.legend()
ax.grid(True)
plt.show()

# Zeit bis 40 °C und Länge der Kühlstrecke
t_40 = np.log((T_start - T_halle) / (40 - T_halle)) / lam_optimal
strecke = 0.05 * t_40
print(f'Zeit bis 40 °C: {t_40:.1f} s')
print(f'Kühlstrecke:    {strecke:.2f} m')
```
Ausgabe:

```text
Zeit bis 40 °C: 27.5 s
Kühlstrecke:    1.38 m
```
Die Modellkurve verläuft mitten durch die Messwerte, mal liegt ein Punkt
etwas darüber, mal etwas darunter. Nach $27.5\,\text{s}$ ist das Teil auf
$40\,°\text{C}$ abgekühlt. Bei $0.05\,\text{m/s}$ legt es in dieser Zeit
$1.38\,\text{m}$ zurück. Mit etwas Reserve sollte die Kühlstrecke also
mindestens $1.5\,\text{m}$ lang sein.
````

```{admonition} Abschlussfrage
:class: tip
Der kleinste Wert der Kostenfunktion ist nicht null, sondern
$K = 0.36\,°\text{C}^2$. Warum findet das Gradientenverfahren kein $\lambda$,
bei dem $K$ null wird? Was sagt die Wurzel aus diesem Wert über die Messung
aus?
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
$K$ wäre nur dann null, wenn die Modellkurve exakt durch alle 13 Messpunkte
liefe. Das Thermoelement misst aber nicht exakt. Jeder Messwert weicht durch
Messrauschen ein wenig von der wahren Temperatur ab, mal nach oben, mal nach
unten. Eine glatte Kurve mit einem einzigen freien Parameter kann diesen
zufälligen Schwankungen nicht folgen. Das ist auch gut so, denn wir wollen das
Abkühlverhalten des Teils beschreiben und nicht das Rauschen.

Die Wurzel aus der mittleren quadratischen Abweichung hat wieder die Einheit
$°\text{C}$. Hier beträgt sie $\sqrt{0.36\,°\text{C}^2} = 0.6\,°\text{C}$. Das
ist die typische Abweichung eines Messwerts von der Modellkurve und passt zur
Genauigkeit eines Thermoelements. Wäre sie deutlich größer, etwa mehrere Grad,
hätten wir einen Hinweis darauf, dass das Modell nicht passt, zum Beispiel
weil die Hallentemperatur nicht $20\,°\text{C}$ beträgt.
````

````{admonition} Zusatzaufgabe: Ein Ausreißer in der Messreihe (✩✩✩)
:class: tip
Durch einen Wackelkontakt am Thermoelement ist der Messwert bei
$30\,\text{s}$ um $10\,°\text{C}$ zu hoch. Erzeugen Sie die fehlerhafte
Messreihe mit

```python
T_fehler = T_mess.copy()
T_fehler[6] = T_fehler[6] + 10
```

1. Schreiben Sie eine Kostenfunktion `kosten_fehler(lam)` für die fehlerhafte
   Messreihe und bestimmen Sie $\lambda$ mit `gradientenverfahren()`.
2. Die Differenzen zwischen Messwert und Modell heißen **Residuen**. Zeichnen
   Sie die Residuen über der Zeit. Woran erkennen Sie den Ausreißer, und was
   fällt an den übrigen Residuen auf?
3. `np.delete(array, 6)` liefert eine Kopie des Arrays ohne den Eintrag mit
   Index 6. Entfernen Sie den Ausreißer aus beiden Arrays und bestimmen Sie
   $\lambda$ erneut. Vergleichen Sie die Zeit bis $40\,°\text{C}$ mit und ohne
   Ausreißer.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Zusatzaufgabe
:class: tip
:class: dropdown
1. Die Kostenfunktion ist dieselbe wie in Teil 2, nur mit `T_fehler`:

   ```python
   # Teilaufgabe 1: lambda aus der fehlerhaften Messreihe
   T_fehler = T_mess.copy()
   T_fehler[6] = T_fehler[6] + 10

   def kosten_fehler(lam):
       """Mittlere quadratische Abweichung für die fehlerhafte Messreihe."""
       abweichung = T_fehler - modell(t_mess, lam)
       return np.mean(abweichung**2)

   verlauf = gradientenverfahren(kosten_fehler, 0.03, 2e-6, 1e-3, 100)
   lam_fehler = verlauf[-1]
   print(f'λ mit Ausreißer: {lam_fehler:.4f} 1/s')
   ```

   Mit dem Ausreißer ergibt sich $\lambda = 0.0482\,\text{1/s}$ statt
   $0.0504\,\text{1/s}$.
2. Wir berechnen die Residuen mit dem $\lambda$ aus Teilaufgabe 1:

   ```python
   # Teilaufgabe 2: Residuen über der Zeit
   residuen = T_fehler - modell(t_mess, lam_fehler)
   print('Residuen in °C:', np.round(residuen, 1))

   fig, ax = plt.subplots(figsize=(7, 4))
   ax.plot(t_mess, residuen, marker='o', linestyle='none')
   ax.axhline(0, color='gray')
   ax.set_xlabel('Zeit in s')
   ax.set_ylabel('Residuum in °C')
   ax.set_title('Residuen mit Ausreißer')
   ax.grid(True)
   plt.show()
   ```

   Ausgabe:

   ```text
   Residuen in °C: [ 0.  -0.3 -1.2 -1.9 -1.5 -2.1  9.1  0.4 -1.3 -1.3 -0.1 -0.1 -0.3]
   ```

   Das Residuum bei $30\,\text{s}$ ist mit $9.1\,°\text{C}$ mehr als viermal
   so groß wie alle anderen. Auffällig ist auch, dass die Residuen von
   $10\,\text{s}$ bis $25\,\text{s}$ alle negativ sind. Der Ausreißer zieht die
   Modellkurve nach oben, sie liegt deshalb in seiner Umgebung über den
   richtigen Messwerten. Weil die Abweichungen quadriert werden, hat ein
   einzelner großer Fehler viel Gewicht.
3. Wir entfernen den Eintrag mit Index 6 aus beiden Arrays:

   ```python
   # Teilaufgabe 3: Ausreißer entfernen und lambda neu bestimmen
   t_ohne = np.delete(t_mess, 6)
   T_ohne = np.delete(T_fehler, 6)

   def kosten_ohne(lam):
       """Mittlere quadratische Abweichung ohne den Ausreißer."""
       abweichung = T_ohne - modell(t_ohne, lam)
       return np.mean(abweichung**2)

   verlauf = gradientenverfahren(kosten_ohne, 0.03, 2e-6, 1e-3, 100)
   lam_ohne = verlauf[-1]

   # Zeit bis 40 °C mit und ohne Ausreißer
   verhaeltnis = (T_start - T_halle) / (40 - T_halle)
   t_40_fehler = np.log(verhaeltnis) / lam_fehler
   t_40_ohne = np.log(verhaeltnis) / lam_ohne
   print(f'λ ohne Ausreißer: {lam_ohne:.4f} 1/s')
   print(f'Zeit bis 40 °C mit Ausreißer:  {t_40_fehler:.1f} s')
   print(f'Zeit bis 40 °C ohne Ausreißer: {t_40_ohne:.1f} s')
   ```

   Ausgabe:

   ```text
   λ ohne Ausreißer: 0.0504 1/s
   Zeit bis 40 °C mit Ausreißer:  28.8 s
   Zeit bis 40 °C ohne Ausreißer: 27.5 s
   ```

   Ohne den Ausreißer finden wir wieder $\lambda = 0.0504\,\text{1/s}$ und
   $27.5\,\text{s}$ bis $40\,°\text{C}$. Ein einziger falscher Messwert hat die
   Wartezeit um gut eine Sekunde verlängert. Hier ist der Fehler harmlos, weil
   er zu einer längeren Kühlstrecke führt. Wäre der Messwert um $10\,°\text{C}$
   zu niedrig gewesen, hätten wir die Wartezeit unterschätzt. Ein Blick auf die
   Residuen gehört deshalb zu jeder Anpassung eines Modells an Messdaten.
````
