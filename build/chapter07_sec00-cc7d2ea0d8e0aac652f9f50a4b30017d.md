---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# Vorbereitung

In den Kapiteln 7.1 und 7.2 berechnen wir Ableitungen aus Messwerten und aus
Formeln. Dafür brauchen Sie etwas Mathematik, Python und Physik, die Sie aus
früheren Semestern und Kapiteln kennen. Prüfen Sie vor der Vorlesung mit dem
Quiz auf dieser Seite, ob alles sitzt. Das dauert rund zehn Minuten. Für jedes
Thema, bei dem Sie Fehler machen, finden Sie unter dem Quiz, was Sie
wiederholen können.

## Das sollten Sie mitbringen

Aus der **Mathematik** sollten Sie Folgendes können:

* die Steigung einer Geraden durch zwei Punkte berechnen
* die Ableitung als Grenzwert verstehen: Aus der Sekante wird die Tangente.
* Potenzfunktionen mit der Potenzregel ableiten
* mit Zehnerpotenzen rechnen

In **Python** sollten Sie Folgendes können:

* eine `for`-Schleife mit `range()` und dem Index `k` schreiben und dabei auf
  `werte[k + 1]` zugreifen
* ein Array mit `np.zeros()` anlegen und in einer Schleife füllen
* Funktionen mit Parametern und `return` schreiben, mit dem Index `-1` und mit
  `len()` arbeiten
* mit `np.argmin()` und `np.argmax()` den Index des kleinsten oder größten
  Werts finden
* mit `plt.subplots()` mehrere Diagramme untereinander zeichnen

Aus der **Physik** sollten Sie Folgendes kennen und können:

* Strecke, Geschwindigkeit und Beschleunigung mit ihren Einheiten
* Bei einer Fahrt geradeaus in eine Richtung ist die Geschwindigkeit die
  Ableitung der Strecke nach der Zeit, die Beschleunigung die Ableitung der
  Geschwindigkeit.
* gleichmäßig beschleunigte Bewegung aus dem Stand
* zwischen m/s und km/h umrechnen

Das FTF als Anwendungsbeispiel kennen Sie aus Kapitel 6. Weitere Kenntnisse
über fahrerlose Transportfahrzeuge brauchen Sie nicht.

## Vorkenntnis-Check

Das Quiz stellt zehn Fragen zu den drei Themen. Am Ende sehen Sie, wie viele
Fragen Sie pro Thema richtig beantwortet haben. Ihr Lernstand bleibt in Ihrem
Browser gespeichert. Beim nächsten Durchlauf kommen die falsch beantworteten
Fragen zuerst. Ein Klick auf das Bild startet das Quiz, das Kreuz oben rechts
schließt es wieder.

<!-- markdownlint-disable -->
<details class="applet-overlay">
<summary>
<img src="pics/chap07_sec00_quiz_vorschau.png"
  alt="Vorschau des Vorkenntnis-Checks. Klicken Sie, um das Quiz zu starten.">
</summary>
<div class="applet-container">
<iframe
  src="https://gramschs.github.io/thma_anm_assets/interactive/chapter07/chap07_sec00_quiz.html"
  width="100%"
  frameborder="0"
  scrolling="auto">
</iframe>
</div>
</details>
<!-- markdownlint-enable -->

## Was Sie wiederholen können

### Mathematik

```{admonition} Aus der Mathematik: Differenzenquotient, Ableitung, Zehnerpotenzen
:class: note
Die Steigung der Sekante durch die Punkte $(x_1 \mid y_1)$ und
$(x_2 \mid y_2)$ ist der **Differenzenquotient**

$$m = \frac{y_2 - y_1}{x_2 - x_1}.$$

Rückt der zweite Punkt immer näher an den ersten, wird aus der Sekante die
Tangente. Ihre Steigung ist die **Ableitung**:

$$f'(x) = \lim_{\Delta x \to 0} \frac{f(x + \Delta x) - f(x)}{\Delta x}.$$

Nach der Potenzregel ist die Ableitung von $x^n$ gleich $n\,x^{n-1}$, zum
Beispiel ist die Ableitung von $0.2\,t^2$ gleich $0.4\,t$. Ableitungen weiterer
Funktionen finden Sie in der
[Tabelle von Ableitungs- und Stammfunktionen](https://de.wikipedia.org/wiki/Tabelle_von_Ableitungs-_und_Stammfunktionen).

Für Zehnerpotenzen gilt $10^a \cdot 10^b = 10^{a+b}$ und
$10^a / 10^b = 10^{a-b}$. Die Schreibweise `2.1e-03` in Python bedeutet
$2.1 \cdot 10^{-3} = 0.0021$.
```

Das folgende Applet zeigt den Übergang von der Sekante zur Tangente an der
Parabel $f(x) = x^2 - 2x + 3$ im Punkt $x_0 = 2$. Verkleinern Sie mit dem
Schieberegler $\Delta x$ und beobachten Sie, wie sich die Sekante der Tangente
nähert.

<!-- markdownlint-disable -->
<div class="applet-container applet-750">
<iframe
  src="https://gramschs.github.io/thma_anm_assets/interactive/chapter04/sekante-tangente.html"
  width="100%"
  frameborder="0"
  scrolling="no">
</iframe>
</div>
<!-- markdownlint-enable -->

```{dropdown} Video "Mittlere Änderungsrate" von Mathematrick
<iframe width="560" height="315" src="https://www.youtube.com/embed/sXxK-JATrc0?si=bdcHhSJOUHSl3fNg"
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay;
clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
```

```{dropdown} Video "Differentialquotient" von Mathematrick
<iframe width="560" height="315" src="https://www.youtube.com/embed/_L6wmTzod_I?si=Fj3jbGIOcc3mC4wc"
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay;
clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
```

```{dropdown} Video "Ableitung" von Mathematische Methoden
<iframe width="560" height="315" src="https://www.youtube.com/embed/FW7Vd1VI3uw?si=Ij7j2mb5CIKNYUEH"
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay;
clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
```

### Python

Alle nötigen Werkzeuge kamen in früheren Kapiteln vor. Wiederholen Sie dort,
was im Quiz nicht gesessen hat:

* Schleifen mit `for` und `range()`:
  [Kapitel 1.1](../chapter01/chapter01_sec01.md)
* Funktionen mit `return`, Index `-1` und `len()`:
  [Kapitel 1.3](../chapter01/chapter01_sec03.md)
* alles auf einen Blick: [Spickzettel Python](../chapter01/chapter01_sec06.md)
* `np.zeros()` und andere Arrays: [Kapitel 2.1](../chapter02/chapter02_sec01.md)
* mehrere Diagramme mit `plt.subplots()`:
  [Kapitel 2.3](../chapter02/chapter02_sec03.md)
* `np.argmax()`: [Kapitel 2.4](../chapter02/chapter02_sec04.md)
* Array mit `np.zeros()` anlegen und in einer Schleife füllen, genau wie in
  Kapitel 7.1: Abschnitt "Wie dreht sich das FTF entlang der Bahn?" in
  [Kapitel 6.3](../chapter06/chapter06_sec03.md)

### Physik

```{admonition} Aus der Physik: Strecke, Geschwindigkeit, Beschleunigung
:class: note
Die **Geschwindigkeit** $v$ ist die Ableitung der Strecke $s$ nach der Zeit,
die **Beschleunigung** $a$ die Ableitung der Geschwindigkeit:

$$v(t) = s'(t), \qquad a(t) = v'(t) = s''(t).$$

Strecken messen wir in m, Geschwindigkeiten in m/s und Beschleunigungen in
$\text{m/s}^2$. Für die Umrechnung gilt $1\,\text{m/s} = 3.6\,\text{km/h}$.

Beschleunigt ein Fahrzeug aus dem Stand gleichmäßig mit $a$, gilt nach der Zeit
$t$

$$v = a\,t, \qquad s = \frac{a}{2}\,t^2.$$

Ein FTF, das $2\,\text{s}$ lang mit $0.4\,\text{m/s}^2$ anfährt, ist danach
$0.8\,\text{m/s}$ schnell und hat $0.8\,\text{m}$ zurückgelegt.
```
