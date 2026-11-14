---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 7.2 Wie klein darf die Schrittweite sein?

In Kapitel 7.1 hat das FTF schlagartig beschleunigt und ebenso schlagartig
damit aufgehört. Solche Sprünge der Beschleunigung spürt die Ladung als Stoß.
Die Änderung der Beschleunigung, also ihre Ableitung nach der Zeit, heißt
**Ruck**. Die Leitsteuerung plant die Anfahrt deshalb sanfter: Die
Geschwindigkeit steigt in $2\,\text{s}$ ohne Knick von $0$ auf
$0.8\,\text{m/s}$. Mit $t$ in Sekunden und $s$ in Metern gilt für die Strecke

$$s(t) = 0.4 \left(t - \frac{2}{\pi} \sin\left(\frac{\pi t}{2}\right)\right)
\quad \text{für } 0 \le t \le 2.$$

Anders als bei Messwerten können wir $s$ jetzt an jeder Stelle auswerten und
die Schrittweite frei wählen. Messrauschen gibt es dabei nicht. In Kapitel 7.1
war es der Grund, warum eine kleine Schrittweite nicht immer besser war. *Wird
die Ableitung ohne Messrauschen immer genauer, je kleiner die Schrittweite ist?*

Weil wir die exakte Ableitung kennen, können wir den Fehler genau messen. Die
Schrittweite, die sich dabei bewährt, verwenden wir in Kapitel 8 für
Funktionen, deren Ableitung wir nicht als Formel aufschreiben wollen.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie können Differenzenquotienten als Python-Funktionen schreiben, die
  eine andere Funktion als Argument erhalten.
* [ ] Sie können den Fehler eines Differenzenquotienten über der Schrittweite
  doppelt logarithmisch darstellen und die **Fehlerordnung** ablesen.
* [ ] Sie können mit der Taylor-Entwicklung erklären, warum der zentrale
  Differenzenquotient die Fehlerordnung 2 hat und der
  Vorwärtsdifferenzenquotient die Fehlerordnung 1.
* [ ] Sie wissen, warum zu kleine Schrittweiten wegen **Rundungsfehlern**
  schaden, und kennen eine gute Schrittweite für beide Differenzenquotienten.
```

## Wie genau ist der Differenzenquotient?

Zuerst schreiben wir die Formel als Python-Funktion `fahrprofil(t)`. Zum
Vergleich programmieren wir auch ihre Ableitung, die exakte Geschwindigkeit
$s'(t) = v(t) = 0.4 \left(1 - \cos\left(\frac{\pi t}{2}\right)\right)$.

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Modell der sanften Anfahrt: Strecke und exakte Geschwindigkeit
def fahrprofil(t):
    """Strecke in m bei der sanften Anfahrt, t in s von 0 bis 2."""
    return 0.4 * (t - 2 / np.pi * np.sin(np.pi * t / 2))

def geschwindigkeit(t):
    """Exakte Geschwindigkeit in m/s, die Ableitung von fahrprofil."""
    return 0.4 * (1 - np.cos(np.pi * t / 2))
```

Um die sanfte Anfahrt mit der Anfahrt aus Kapitel 7.1 zu vergleichen,
berechnen wir beide Geschwindigkeiten an 101 Zeitpunkten von $0$ bis
$2\,\text{s}$. In Kapitel 7.1 stieg die Geschwindigkeit gleichmäßig mit
$0.4\,\text{m/s}^2$, also gilt dort $v = 0.4\,t$. Weil `geschwindigkeit()` mit
NumPy-Funktionen rechnet, können wir ihr das ganze Array der Zeitpunkte auf
einmal übergeben.

```{code-cell} python
# 101 Zeitpunkte von 0 s bis 2 s
t = np.linspace(0, 2, 101)

# Geschwindigkeit in m/s: gleichmäßige Anfahrt aus Kapitel 7.1, sanfte Anfahrt
v_gleichmaessig = 0.4 * t
v_sanft = geschwindigkeit(t)
```

Das Diagramm stellt die beiden Anfahrten gegenüber.

```{code-cell} python
# Diagramm: Geschwindigkeit der beiden Anfahrten
fig, ax = plt.subplots(figsize=(7, 4))
ax.plot(t, v_gleichmaessig, color='gray', linestyle='--',
        label='Anfahrt aus Kapitel 7.1')
ax.plot(t, v_sanft, label='sanfte Anfahrt')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Geschwindigkeit in m/s')
ax.set_title('Zwei Anfahrten auf 0.8 m/s')
ax.legend()
ax.grid(True)
plt.show()
```

Beide Anfahrten erreichen nach $2\,\text{s}$ die Geschwindigkeit
$0.8\,\text{m/s}$. Die sanfte Anfahrt beginnt und endet aber waagerecht. Die
Beschleunigung steigt also allmählich an und fällt allmählich wieder ab.

Für die Differenzenquotienten schreiben wir zwei Funktionen. Sie erhalten die
Funktion, die sie ableiten sollen, als ersten Parameter `funktion`. In Python
können wir eine Funktion wie eine Zahl an eine andere Funktion übergeben. Dabei
schreiben wir ihren Namen ohne Klammern, denn wir übergeben die Funktion selbst
und keinen Funktionswert. Beim Aufruf `vorwaerts(fahrprofil, 1.5, 0.1)` steht
`funktion` für `fahrprofil`, und `funktion(t + dt)` berechnet
`fahrprofil(1.6)`.

```{code-cell} python
# Differenzenquotienten für eine beliebige Funktion
def vorwaerts(funktion, t, dt):
    """Vorwärtsdifferenzenquotient von funktion an der Stelle t."""
    return (funktion(t + dt) - funktion(t)) / dt

def zentral(funktion, t, dt):
    """Zentraler Differenzenquotient von funktion an der Stelle t."""
    return (funktion(t + dt) - funktion(t - dt)) / (2 * dt)
```

Wir werten bei $t = 1.5\,\text{s}$ aus, kurz bevor das FTF seine
Endgeschwindigkeit erreicht, und probieren drei Schrittweiten. Als Fehler
nehmen wir den Betrag der Abweichung vom exakten Wert.

```{code-cell} python
# Stelle der Auswertung in s und exakte Geschwindigkeit dort
t_0 = 1.5
exakt = geschwindigkeit(t_0)
print(f'Exakte Geschwindigkeit bei t = {t_0} s: {exakt:.6f} m/s')

# Fehler = Betrag der Abweichung vom exakten Wert,
# fahrprofil ohne Klammern: Wir übergeben die Funktion selbst
for dt in [0.1, 0.01, 0.001]:
    fehler_vorwaerts = abs(vorwaerts(fahrprofil, t_0, dt) - exakt)
    fehler_zentral = abs(zentral(fahrprofil, t_0, dt) - exakt)
    print(f'dt = {dt:5}:  Fehler vorwärts {fehler_vorwaerts:.2e} m/s, '
          f'zentral {fehler_zentral:.2e} m/s')
```

Die Schreibweise `2.10e-02` steht für $2.10 \cdot 10^{-2}$. Wird die
Schrittweite zehnmal kleiner, sinkt der Fehler des Vorwärtsdifferenzenquotienten
auf ein Zehntel, der des zentralen Differenzenquotienten auf ein Hundertstel.
Der Fehler ist also ungefähr proportional zu $\Delta t$ beziehungsweise zu
$\Delta t^2$. Den Exponenten nennen wir **Fehlerordnung**: Der
Vorwärtsdifferenzenquotient hat die Fehlerordnung 1, der zentrale die
Fehlerordnung 2. Schon bei $\Delta t = 0.1\,\text{s}$ ist der zentrale
Differenzenquotient genauer als der Vorwärtsdifferenzenquotient bei
$\Delta t = 0.01\,\text{s}$.

```{admonition} Mini-Übung (✩)
:class: tip
1. Schreiben Sie eine Funktion `rueckwaerts(funktion, t, dt)` für den
   Rückwärtsdifferenzenquotienten. Berechnen Sie für $\Delta t = 0.1\,\text{s}$
   bei $t = 1.5\,\text{s}$ die Abweichung vom exakten Wert mit Vorzeichen, also
   ohne `abs()`, für den Vorwärts- und den Rückwärtsdifferenzenquotienten.
2. Beantworten Sie ohne Code: Warum liegt der Vorwärtsdifferenzenquotient über
   der exakten Geschwindigkeit und der Rückwärtsdifferenzenquotient darunter?
3. Beantworten Sie ohne Code: Wie groß ist der Fehler des zentralen
   Differenzenquotienten für die Anfahrt aus Kapitel 7.1 mit
   $s(t) = 0.2\,t^2$ während der Beschleunigung?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
1. Der Rückwärtsdifferenzenquotient verwendet den Funktionswert eine
   Schrittweite vorher. Die Abweichung berechnen wir ohne `abs()`:

   ```python
   # Teilaufgabe 1: Rückwärtsdifferenzenquotient und Abweichung mit Vorzeichen
   def rueckwaerts(funktion, t, dt):
       """Rückwärtsdifferenzenquotient von funktion an der Stelle t."""
       return (funktion(t) - funktion(t - dt)) / dt

   abweichung_vorwaerts = vorwaerts(fahrprofil, t_0, 0.1) - exakt
   abweichung_rueckwaerts = rueckwaerts(fahrprofil, t_0, 0.1) - exakt
   # Das + im Format zeigt das Vorzeichen immer an
   print(f'Vorwärts:  {abweichung_vorwaerts:+.4f} m/s')
   print(f'Rückwärts: {abweichung_rueckwaerts:+.4f} m/s')
   ```

   Der Vorwärtsdifferenzenquotient liegt $0.0210\,\text{m/s}$ zu hoch, der
   Rückwärtsdifferenzenquotient $0.0233\,\text{m/s}$ zu niedrig.
2. Bei $t = 1.5\,\text{s}$ beschleunigt das FTF noch. Der
   Vorwärtsdifferenzenquotient ist die mittlere Geschwindigkeit von
   $1.5\,\text{s}$ bis $1.6\,\text{s}$, und in dieser Zeit ist das FTF
   schneller als bei $1.5\,\text{s}$. Der Rückwärtsdifferenzenquotient mittelt
   über die Zeit davor, als das FTF langsamer war. Das kennen wir aus Kapitel
   7.1.
3. Für $s(t) = 0.2\,t^2$ ist der zentrale Differenzenquotient für jede
   Schrittweite exakt, sein Fehler ist null. Bei der Anfahrt aus Kapitel 7.1
   gilt diese Formel aber nur von $0$ bis $2\,\text{s}$. Exakt ist der zentrale
   Differenzenquotient deshalb nur, solange $t - \Delta t$ und $t + \Delta t$
   beide in diesem Zeitintervall liegen. In Kapitel 7.1 hat er so bei $0.5$,
   $1$ und $1.5\,\text{s}$ genau die geplanten Werte geliefert, am Knick bei
   $2\,\text{s}$ dagegen $0.75$ statt $0.8\,\text{m/s}$. Warum er exakt ist,
   zeigt der nächste Abschnitt.
````

## Wie lesen wir die Fehlerordnung ab?

Drei Schrittweiten zeigen den Trend, ein Diagramm zeigt ihn deutlicher. Dafür
brauchen wir viele Schrittweiten über mehrere Zehnerpotenzen.
`np.logspace(-1, -4, 31)` erzeugt 31 Zahlen von $10^{-1}$ bis $10^{-4}$, zehn
pro Zehnerpotenz. Für jede Schrittweite berechnen wir in einer Schleife die
beiden Fehler und sammeln sie in Arrays, die wir vorher mit `np.zeros()`
anlegen.

```{code-cell} python
# 31 Schrittweiten von 0.1 bis 0.0001, zehn pro Zehnerpotenz
dt_werte = np.logspace(-1, -4, 31)

# Fehler beider Differenzenquotienten für jede Schrittweite
fehler_vorwaerts = np.zeros(len(dt_werte))
fehler_zentral = np.zeros(len(dt_werte))
for k in range(len(dt_werte)):
    fehler_vorwaerts[k] = abs(vorwaerts(fahrprofil, t_0, dt_werte[k]) - exakt)
    fehler_zentral[k] = abs(zentral(fahrprofil, t_0, dt_werte[k]) - exakt)
```

`ax.loglog()` zeichnet wie `ax.plot()`, teilt aber beide Achsen logarithmisch
ein: Jede Zehnerpotenz bekommt auf der Achse gleich viel Platz.
`ax.grid(True, which='both')` zeichnet auch die feinen Gitterlinien zwischen
den Zehnerpotenzen.

In einem solchen **doppelt logarithmischen Diagramm** wird aus dem
Zusammenhang $\text{Fehler} = C \cdot \Delta t^p$ eine Gerade mit der Steigung
$p$. Die Fehlerordnung ist also die Steigung der Geraden. Wir lesen sie in
Zehnerpotenzen ab: Um wie viele Zehnerpotenzen ändert sich der Fehler, wenn
sich die Schrittweite um eine Zehnerpotenz ändert? Dasselbe Werkzeug haben wir
im Exkurs in Kapitel 3.7 für die Rechenzeit verwendet, dort war die Steigung 3.

```{code-cell} python
# Diagramm: Fehler über der Schrittweite, doppelt logarithmisch
fig, ax = plt.subplots(figsize=(7, 5))
ax.loglog(dt_werte, fehler_vorwaerts, marker='.', label='vorwärts')
ax.loglog(dt_werte, fehler_zentral, marker='.', label='zentral')
ax.set_xlabel('Schrittweite in s')
ax.set_ylabel('Fehler in m/s')
ax.set_title('Fehler der Differenzenquotienten bei t = 1.5 s')
ax.legend()
ax.grid(True, which='both')
plt.show()
```

Die Achse der Schrittweite steigt wie gewohnt nach rechts an, die kleinen
Schrittweiten stehen also links. Beide Kurven sind Geraden. Über drei
Zehnerpotenzen der Schrittweite fällt der Fehler des
Vorwärtsdifferenzenquotienten um drei Zehnerpotenzen, von rund
$2 \cdot 10^{-2}$ auf rund $2 \cdot 10^{-5}$. Die Steigung ist also 1. Der
Fehler des zentralen Differenzenquotienten fällt doppelt so steil, von rund
$10^{-3}$ auf rund $10^{-9}$, die Steigung ist 2. *Warum ist der zentrale
Differenzenquotient so viel genauer?*

```{admonition} Aus der Mathematik: Taylor-Entwicklung
:class: note
Die **Taylor-Entwicklung** schreibt einen Funktionswert in der Nähe von $t$ als
Summe von Potenzen der Schrittweite:

$$s(t + \Delta t) = s(t) + \Delta t \cdot s'(t) + \frac{\Delta t^2}{2}
\cdot s''(t) + \frac{\Delta t^3}{6} \cdot s'''(t) + \ldots$$

Setzen wir das in den Vorwärtsdifferenzenquotienten ein, bleibt neben $s'(t)$
als größter Fehlerterm $\frac{\Delta t}{2} \cdot s''(t)$ übrig, proportional zu
$\Delta t$. In der Entwicklung von $s(t - \Delta t)$ wechseln die ungeraden
Potenzen von $\Delta t$ ihr Vorzeichen. Im zentralen Differenzenquotienten
heben sich deshalb die Terme mit $s''(t)$ auf. Der größte Fehlerterm ist dann
$\frac{\Delta t^2}{6} \cdot s'''(t)$, proportional zu $\Delta t^2$.
```

Für die Anfahrt ist $s''$ die Beschleunigung $a$. Der größte Fehlerterm
$a \cdot \Delta t / 2$ des Vorwärtsdifferenzenquotienten ist also ungefähr die
Änderung der Geschwindigkeit in einer halben Schrittweite. Das passt zu Kapitel
7.1: Der
Vorwärtsdifferenzenquotient liefert näherungsweise die Geschwindigkeit in der
Mitte des Intervalls, eine halbe Schrittweite zu spät. $s'''$ ist der Ruck. In
Kapitel 7.1 war die Beschleunigung konstant und der Ruck null. Deshalb war der
zentrale Differenzenquotient dort exakt, außer am Knick bei $t = 2\,\text{s}$.
An einem Knick gilt die Taylor-Entwicklung nicht, und der zentrale
Differenzenquotient kann sogar schlechter sein als der
Vorwärtsdifferenzenquotient. Bei $t = 2\,\text{s}$ liefert der
Vorwärtsdifferenzenquotient mit $(1.2 - 0.8)/0.5 = 0.8\,\text{m/s}$ genau die
geplante Geschwindigkeit, der zentrale nur $0.75\,\text{m/s}$. Die
Fehlerordnungen gelten also für kleine Schrittweiten und Funktionen ohne Knick.

```{admonition} Mini-Übung (✩)
:class: tip
1. Lesen Sie aus dem Diagramm ab: Welche Schrittweite braucht jeder der beiden
   Differenzenquotienten, damit der Fehler unter
   $1\,\text{mm/s} = 10^{-3}\,\text{m/s}$ bleibt? Prüfen Sie Ihre Ablesung mit
   `vorwaerts()` und `zentral()`.
2. Beantworten Sie ohne Code: Bei $t = 1.5\,\text{s}$ beschleunigt das FTF mit
   $0.444\,\text{m/s}^2$. Schätzen Sie mit der Taylor-Entwicklung den Fehler
   des Vorwärtsdifferenzenquotienten für $\Delta t = 0.01\,\text{s}$ und
   vergleichen Sie mit der Tabelle aus dem vorigen Abschnitt.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
1. Im Diagramm schneidet die Gerade des Vorwärtsdifferenzenquotienten die Linie
   $10^{-3}$ bei knapp $5 \cdot 10^{-3}\,\text{s}$, die des zentralen bei knapp
   $10^{-1}\,\text{s}$. Wir prüfen die Ablesung mit zwei Schrittweiten knapp
   darunter:

   ```python
   # Teilaufgabe 1: Ablesung prüfen, 1 mm/s = 1e-3 m/s
   fehler_v = abs(vorwaerts(fahrprofil, t_0, 0.0045) - exakt)
   fehler_z = abs(zentral(fahrprofil, t_0, 0.09) - exakt)
   print(f'vorwärts, dt = 0.0045 s: {fehler_v:.2e} m/s')
   print(f'zentral,  dt = 0.09 s:   {fehler_z:.2e} m/s')
   ```

   Mit $\Delta t = 0.0045\,\text{s}$ und $\Delta t = 0.09\,\text{s}$ bleiben
   beide gerade unter $1\,\text{mm/s}$. Der zentrale Differenzenquotient
   erlaubt für dieselbe Genauigkeit also eine etwa zwanzigmal größere
   Schrittweite.
2. Nach der Taylor-Entwicklung ist der Fehler etwa $a \cdot \Delta t / 2 = 0.444
   \cdot 0.01 / 2$, also $2.22 \cdot 10^{-3}\,\text{m/s}$. Das stimmt mit dem
   Wert $2.21 \cdot 10^{-3}\,\text{m/s}$ aus der Tabelle fast überein. Den
   kleinen Rest erklären die höheren Terme der Taylor-Entwicklung.
````

## Was passiert bei winzigen Schrittweiten?

Nach der Taylor-Entwicklung sollte der Fehler mit kleinerer Schrittweite immer
weiter sinken. Wir prüfen das und verkleinern die Schrittweite bis $10^{-15}$.
Die Rechnung ist dieselbe wie im vorigen Abschnitt, nur mit 141 Schrittweiten.

```{code-cell} python
# 141 Schrittweiten von 0.1 bis 1e-15, zehn pro Zehnerpotenz
dt_werte = np.logspace(-1, -15, 141)

# Fehler beider Differenzenquotienten für jede Schrittweite
fehler_vorwaerts = np.zeros(len(dt_werte))
fehler_zentral = np.zeros(len(dt_werte))
for k in range(len(dt_werte)):
    fehler_vorwaerts[k] = abs(vorwaerts(fahrprofil, t_0, dt_werte[k]) - exakt)
    fehler_zentral[k] = abs(zentral(fahrprofil, t_0, dt_werte[k]) - exakt)
```

Das Diagramm zeichnen wir wie im vorigen Abschnitt, nur ohne Punkte.

```{code-cell} python
# Diagramm: Fehler bis zu sehr kleinen Schrittweiten, doppelt logarithmisch
fig, ax = plt.subplots(figsize=(7, 5))
ax.loglog(dt_werte, fehler_vorwaerts, label='vorwärts')
ax.loglog(dt_werte, fehler_zentral, label='zentral')
ax.set_xlabel('Schrittweite in s')
ax.set_ylabel('Fehler in m/s')
ax.set_title('Fehler bei sehr kleinen Schrittweiten')
ax.legend()
ax.grid(True, which='both')
plt.show()
```

Zunächst folgen die Kurven den Geraden aus dem vorigen Abschnitt. Ihr Minimum
erreichen sie beim Vorwärtsdifferenzenquotienten bei etwa $2 \cdot 10^{-8}$,
beim zentralen bei etwa $5 \cdot 10^{-6}$. Darunter steigt der Fehler wieder an
und wird unruhig. Bei $\Delta t = 10^{-15}$ ist er sogar größer als bei
$\Delta t = 0.1$. *Woher kommt dieser zusätzliche Fehler?*

Python speichert eine Kommazahl mit rund 16 gültigen Dezimalstellen und rundet
den Rest. Diese Abweichung heißt **Rundungsfehler**. Was er anrichtet, sehen wir
an den beiden Funktionswerten für $\Delta t = 10^{-10}$. Wir geben sie mit dem
Format `:.17f` auf 17 Nachkommastellen aus.

```{code-cell} python
# Zwei Funktionswerte im Abstand dt = 1e-10 s
dt = 1e-10
s_rechts = fahrprofil(t_0 + dt)
s_links = fahrprofil(t_0)

# Differenz und Vorwärtsdifferenzenquotient
differenz = s_rechts - s_links
v_vorwaerts = differenz / dt

# Ausgabe mit 17 Nachkommastellen, exakte Geschwindigkeit zum Vergleich
print(f's(t_0 + dt) = {s_rechts:.17f} m')
print(f's(t_0)      = {s_links:.17f} m')
print(f'Differenz   = {differenz:.17f} m')
print(f'vorwärts    = {v_vorwaerts:.17f} m/s')
print(f'exakt       = {exakt:.17f} m/s')
```

Die beiden Funktionswerte stimmen in den ersten neun Nachkommastellen überein.
Beim Abziehen werden diese Stellen zu Nullen, von den rund 16 gültigen Stellen
bleiben nur etwa sechs übrig. Dieser Verlust gültiger Stellen heißt
**Auslöschung**. Deshalb stimmt auch der Differenzenquotient nur in sechs
Stellen mit der exakten Geschwindigkeit überein.

Wie genau Python rechnet, gibt die **Maschinengenauigkeit** eps an, der
kleinste relative Abstand zwischen $1$ und der nächsten speicherbaren Zahl. Bei
Größen um 1 hat jeder Funktionswert einen Rundungsfehler von etwa eps. Wie das
Messrauschen in Kapitel 7.1 teilt der Differenzenquotient ihn durch die
Schrittweite. Der Rundungsfehler im Ergebnis ist also etwa
$\text{eps} / \Delta t$, deshalb haben die unruhigen Teile der Kurven links
ungefähr die Steigung $-1$.

Am besten ist die Schrittweite, bei der der Fehler aus der Taylor-Entwicklung
und der Rundungsfehler etwa gleich groß sind. Ohne die Faktoren $C$ gilt beim
Vorwärtsdifferenzenquotienten $\Delta t \approx \text{eps} / \Delta t$, also
die Faustregel $\Delta t \approx \sqrt{\text{eps}}$. Beim zentralen gilt
$\Delta t^2 \approx \text{eps} / \Delta t$, also
$\Delta t \approx \sqrt[3]{\text{eps}}$. NumPy liefert eps mit
`np.finfo(float).eps`, die dritte Wurzel berechnen wir als `eps ** (1 / 3)`.

```{code-cell} python
# Maschinengenauigkeit
eps = np.finfo(float).eps

# Faustregeln für die beste Schrittweite
dt_vorwaerts = np.sqrt(eps)
dt_zentral = eps ** (1 / 3)

# Ausgabe der Faustregeln
print(f'Maschinengenauigkeit eps:       {eps:.1e}')
print(f'vorwärts, Wurzel aus eps:       {dt_vorwaerts:.1e} s')
print(f'zentral, dritte Wurzel aus eps: {dt_zentral:.1e} s')
```

Die Faustregeln $1.5 \cdot 10^{-8}$ und $6.1 \cdot 10^{-6}$ passen gut zu den
Minima im Diagramm. Sie gelten aber nur für Größen um 1, wie hier die Zeit um
$1.5\,\text{s}$ und die Strecke um $0.4\,\text{m}$. Ist die Variable viel
größer oder kleiner, wächst oder schrumpft die beste Schrittweite mit, beim
zentralen Differenzenquotienten etwa auf $\sqrt[3]{\text{eps}} \cdot |t|$.
Auch die Funktion selbst spielt eine Rolle. Die Faustregeln liefern deshalb
nur die Größenordnung.

```{admonition} Mini-Übung (✩)
:class: tip
1. In Kapitel 8 verwenden wir den zentralen Differenzenquotienten mit der
   Schrittweite $10^{-6}$. Berechnen Sie den Fehler bei $t = 1.5\,\text{s}$ für
   diese Schrittweite und vergleichen Sie ihn mit dem kleinsten Fehler im
   Diagramm.
2. Beantworten Sie ohne Code: Ein Kommilitone rechnet mit dem zentralen
   Differenzenquotienten und wählt $\Delta t = 10^{-12}$, damit die Ableitung
   möglichst genau wird. Was antworten Sie ihm? Nutzen Sie das Diagramm.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip
:class: dropdown
1. Wir berechnen den Fehler mit der Schrittweite aus Kapitel 8:

   ```python
   # Teilaufgabe 1: Schrittweite aus Kapitel 8
   fehler_kapitel_8 = abs(zentral(fahrprofil, t_0, 1e-6) - exakt)
   print(f'Fehler bei dt = 1e-6 s: {fehler_kapitel_8:.1e} m/s')
   ```

   Mit der Schrittweite $10^{-6}$ beträgt der Fehler
   $1.7 \cdot 10^{-11}\,\text{m/s}$. Der kleinste Fehler im Diagramm liegt bei
   gut $10^{-12}\,\text{m/s}$, also etwa zehnmal niedriger. Verglichen mit der
   Geschwindigkeit von $0.68\,\text{m/s}$ sind beide winzig. Wir müssen das
   Optimum also nicht genau treffen, nur seine Größenordnung.
2. Kleiner ist hier nicht genauer. $\Delta t = 10^{-12}$ liegt weit links vom
   Minimum, im unruhigen Bereich der Rundungsfehler. Der Fehler springt dort
   zwischen etwa $10^{-5}$ und $10^{-4}\,\text{m/s}$. So groß ist er auch schon
   bei $\Delta t = 0.01$. Mit einer Schrittweite nach der Faustregel, also
   $10^{-5}$ bis $10^{-6}$, wäre der Fehler rund eine Million Mal kleiner.
````

## Zusammenfassung und Ausblick

Liegt eine Funktion als Python-Funktion vor, übergeben wir sie an `vorwaerts()`
oder `zentral()` und wählen die Schrittweite frei. Für kleine Schrittweiten ist
der Fehler des Vorwärtsdifferenzenquotienten ungefähr proportional zu
$\Delta t$, der des zentralen Differenzenquotienten zu $\Delta t^2$, solange
die Funktion keinen Knick hat. Im doppelt logarithmischen Diagramm erscheint
die Fehlerordnung als Steigung, die Taylor-Entwicklung erklärt sie. Sehr kleine
Schrittweiten schaden wegen Rundungsfehlern und Auslöschung, denn der
Rundungsfehler wächst wie $\text{eps} / \Delta t$. Für Größen um 1 sind etwa
$\sqrt{\text{eps}} \approx 10^{-8}$ für den Vorwärtsdifferenzenquotienten und
$\sqrt[3]{\text{eps}}$, also $10^{-5}$ bis $10^{-6}$, für den zentralen
Differenzenquotienten gut.

Die Übungen in Kapitel 7.3 wiederholen beide Kapitel, unter anderem an der
Notbremsung eines FTF. In Kapitel 8.1 suchen wir den Parameter eines Modells,
der am besten zu Messdaten passt. Die Ableitung, die uns dabei die Richtung
weist, berechnen wir mit dem zentralen Differenzenquotienten und der
Schrittweite $10^{-6}$.
