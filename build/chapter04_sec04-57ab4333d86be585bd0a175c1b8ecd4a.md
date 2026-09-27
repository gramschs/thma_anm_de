---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.4 Dachbinder einer Stahlhalle unter Schneelast

In Kapitel 4.3 haben wir die Lager eingebaut, Verschiebungen, Lagerkräfte und
Stabkräfte berechnet und den Kranausleger auf Spannung und Knicken geprüft.
In diesem Kapitel setzen wir die drei Funktionen ein, um einen Dachbinder zu
berechnen und zu bewerten. Bearbeiten Sie die Teilaufgaben möglichst zu
zweit und der Reihe nach.

````{admonition} Projekt: Dachbinder einer Stahlhalle (✩✩)
:class: tip
Das Dach einer kleinen Stahlhalle ruht auf Dachbindern mit $6\,\text{m}$
Spannweite und $2\,\text{m}$ Firsthöhe. Nach starkem Schneefall drückt auf
jeden der drei oberen Knoten eine Last von $2000\,\text{N}$ nach unten. Der
Binder liegt links auf einem Festlager und rechts auf einem Loslager, wie der
Träger in Kapitel 3.2.

```{figure} pics/chap04_dachbinder.svg
:alt: Dachbinder aus elf Stäben mit sieben Knoten, Festlager links, Loslager rechts und drei Schneelasten auf den oberen Knoten
:align: center

Der Dachbinder mit Knotennummern, Stabnummern und der Schneelast.
(Quelle: eigene Abbildung; Lizenz [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0))
```

| Knoten | $x$ in m | $y$ in m | Bemerkung |
| --- | --- | --- | --- |
| 0 | 0.0 | 0.0 | Festlager |
| 1 | 2.0 | 0.0 | frei |
| 2 | 4.0 | 0.0 | frei |
| 3 | 6.0 | 0.0 | Loslager |
| 4 | 2.0 | 4/3 | frei, Schneelast |
| 5 | 3.0 | 2.0 | frei, Schneelast (First) |
| 6 | 4.0 | 4/3 | frei, Schneelast |

| Stab | verbindet Knoten | Bauteil |
| --- | --- | --- |
| 0, 1, 2 | 0 und 1, 1 und 2, 2 und 3 | Untergurt |
| 3, 4, 5, 6 | 0 und 4, 4 und 5, 5 und 6, 6 und 3 | Obergurt |
| 7, 8 | 1 und 4, 2 und 6 | Pfosten |
| 9, 10 | 1 und 5, 2 und 5 | Diagonalen |

Alle Stäbe sind Rundstäbe aus Baustahl S235 mit $2\,\text{cm}$ Durchmesser,
$E = 2.1 \cdot 10^{11}\,\text{N/m}^2$ und der Streckgrenze
$R_e = 235\,\text{N/mm}^2$.
````

Führen Sie zuerst die beiden folgenden Zellen aus. Die erste enthält die
vorgegebene Zeichenfunktion, die zweite die Importe und die drei Funktionen
aus den Kapiteln 4.1 und 4.3.

Die Funktion `berechne_verschiebungen` hat einen zusätzlichen Parameter
`loslager_indizes`. Ein Loslager wie das Lager B in Kapitel 3.2 hält den
Knoten nur in senkrechter Richtung fest, waagerecht kann er sich frei
bewegen. Deshalb ersetzt die Funktion an einem Loslager nur die Gleichung für
$u_y$. Auch `zeichne_fachwerk` kennt diesen Parameter und zeichnet Loslager
mit einem zusätzlichen Strich.

%%ZEICHNE%%

%%BAUE%%

```{admonition} Teil 1: Den Dachbinder beschreiben und zeichnen
:class: tip
Legen Sie die Materialwerte, `knoten_pos`, `lager_indizes` für das Festlager,
`loslager_indizes` für das Loslager, `staebe` und `kraft_vektor` an. Halten
Sie bei der Stabliste die Reihenfolge der Stabnummern ein. Zeichnen Sie den
Binder mit

`zeichne_fachwerk(knoten_pos, staebe, lager_indizes, loslager_indizes=loslager_indizes, kraft_vektor=kraft_vektor)`

und vergleichen Sie den Plot mit der Skizze.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
```python
# Material: Rundstäbe aus S235 mit 2 cm Durchmesser
elastizitaetsmodul = 2.1e11                  # in N/m²
durchmesser = 0.02                           # in m
querschnitt = np.pi * durchmesser**2 / 4     # in m²
streckgrenze = 235.0                         # in N/mm²

# Knotenkoordinaten in m
knoten_pos = np.array([
    [0.0, 0.0],       # Knoten 0: Festlager
    [2.0, 0.0],       # Knoten 1
    [4.0, 0.0],       # Knoten 2
    [6.0, 0.0],       # Knoten 3: Loslager
    [2.0, 4 / 3],     # Knoten 4
    [3.0, 2.0],       # Knoten 5: First
    [4.0, 4 / 3],     # Knoten 6
])
anzahl_knoten = len(knoten_pos)

lager_indizes = [0]      # Festlager
loslager_indizes = [3]   # Loslager

staebe = np.array([
    [0, 1], [1, 2], [2, 3],           # Stäbe 0 bis 2: Untergurt
    [0, 4], [4, 5], [5, 6], [6, 3],   # Stäbe 3 bis 6: Obergurt
    [1, 4], [2, 6],                   # Stäbe 7 und 8: Pfosten
    [1, 5], [2, 5],                   # Stäbe 9 und 10: Diagonalen
])

# Schneelast: Fy an den Knoten 4, 5 und 6 steht an den Indizes 9, 11 und 13
kraft_vektor = np.zeros(2 * anzahl_knoten)
kraft_vektor[9] = -2000.0
kraft_vektor[11] = -2000.0
kraft_vektor[13] = -2000.0

zeichne_fachwerk(knoten_pos, staebe, lager_indizes,
                 loslager_indizes=loslager_indizes, kraft_vektor=kraft_vektor,
                 titel='Dachbinder unter Schneelast')
```
Die Koordinaten der Knoten 4 und 6 schreiben wir als `4 / 3`, dann rechnet
Python den Wert exakt genug aus. Die Schneelast an Knoten $n$ steht an Index
$2n + 1$, also an den Indizes 9, 11 und 13.
````

```{admonition} Teil 2: Vorhersagen, dann rechnen
:class: tip
1. Überlegen Sie ohne Code: Steht der Untergurt unter Zug oder unter Druck?
   Und der Obergurt? Notieren Sie Ihre Vorhersage.
2. Berechnen Sie mit den drei Funktionen die Steifigkeitsmatrix, die
   Verschiebungen und die Stabkräfte. Geben Sie die Stabkräfte mit Zug oder
   Druck aus und zeichnen Sie den Binder mit Stabkräften und 200-fach
   überhöhten Verschiebungen. Stimmt Ihre Vorhersage?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 2
:class: tip
:class: dropdown
```python
K = baue_steifigkeitsmatrix(knoten_pos, staebe, elastizitaetsmodul, querschnitt)
u = berechne_verschiebungen(K, kraft_vektor, lager_indizes, loslager_indizes)
stabkraefte = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                   querschnitt, u)

for s in range(len(staebe)):
    art = 'Zug' if stabkraefte[s] > 0 else 'Druck'
    print(f'Stab {s:2d}: N = {stabkraefte[s]:8.1f} N  ({art})')
print(f'Absenkung des Firsts: {u[11] * 1000:.3f} mm')

zeichne_fachwerk(knoten_pos, staebe, lager_indizes,
                 loslager_indizes=loslager_indizes, kraft_vektor=kraft_vektor,
                 verschiebung=u, skalierung=200, stabkraefte=stabkraefte,
                 titel='Dachbinder, Verschiebungen 200-fach überhöht')
```
Ausgabe:

```text
%%AUSGABE2%%
```

Der Untergurt steht unter Zug, der Obergurt unter Druck. Die Schneelast
drückt die schrägen Obergurtstäbe zusammen, und diese wollen die beiden
Auflager nach außen schieben. Der Untergurt hält die Auflager zusammen wie
ein Zugband. Die Pfosten tragen die Last der Knoten 4 und 6 auf Druck nach
unten, die Diagonalen hängen den First an den Untergurt und stehen unter Zug.
````

```{admonition} Teil 3: Lagerkräfte und Gleichgewicht
:class: tip
Berechnen Sie die Knotenkräfte $\mathbf{K} \cdot \vec{u}$ und geben Sie die
Kräfte an den Lagerknoten 0 und 3 aus.

1. Warum trägt jedes Lager genau die Hälfte der gesamten Schneelast?
2. Warum ist die waagerechte Lagerkraft am Festlager null?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 3
:class: tip
:class: dropdown
```python
knotenkraefte = K @ u

for n in [0, 3]:
    print(f'Knoten {n}: Fx = {knotenkraefte[2*n]:8.1f} N,  '
          f'Fy = {knotenkraefte[2*n + 1]:8.1f} N')
```
Ausgabe:

```text
%%AUSGABE3%%
```

Die gesamte Schneelast beträgt $3 \cdot 2000\,\text{N} = 6000\,\text{N}$.
Binder und Last sind spiegelbildlich zur Mitte, deshalb trägt jedes Lager
$3000\,\text{N}$ nach oben. Waagerecht wirken nur Lagerkräfte, denn die
Schneelast zeigt senkrecht nach unten. Das Loslager kann keine waagerechte
Kraft aufnehmen. Damit die Summe der waagerechten Kräfte null ist, muss auch
die waagerechte Kraft am Festlager null sein, genau wie beim Träger in
Kapitel 3.2.
````

```{admonition} Teil 4: Hält der Dachbinder?
:class: tip
Prüfen Sie jeden Stab mit einer Schleife:

- Berechnen Sie die Spannung $\sigma = N/A$ in N/mm² und die Auslastung
  $|\sigma| / R_e$.
- Für jeden Druckstab berechnen Sie zusätzlich die Euler-Knicklast
  $F_\text{krit} = \pi^2 E I / L^2$ mit $I = \pi d^4 / 64$ und vergleichen
  sie mit dem Betrag der Druckkraft.

Welche Stäbe bestehen einen der beiden Nachweise nicht?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 4
:class: tip
:class: dropdown
```python
traegheitsmoment = np.pi * durchmesser**4 / 64   # in m^4

print('Stab   N in N    sigma in N/mm²  Auslastung  Knicklast in N  Knicken?')
for s in range(len(staebe)):
    i, j = staebe[s]
    differenz = knoten_pos[j] - knoten_pos[i]
    stablaenge = np.sqrt(differenz[0]**2 + differenz[1]**2)

    # Spannungsnachweis für alle Stäbe
    spannung = stabkraefte[s] / querschnitt * 1e-6
    auslastung = abs(spannung) / streckgrenze

    # Knicknachweis nur für Druckstäbe
    if stabkraefte[s] < 0:
        knicklast = np.pi**2 * elastizitaetsmodul * traegheitsmoment / stablaenge**2
        knickt = 'ja' if abs(stabkraefte[s]) > knicklast else 'nein'
        print(f'{s:4d}  {stabkraefte[s]:8.1f}  {spannung:10.1f}  {auslastung * 100:9.1f} %'
              f'  {knicklast:12.1f}    {knickt}')
    else:
        print(f'{s:4d}  {stabkraefte[s]:8.1f}  {spannung:10.1f}  {auslastung * 100:9.1f} %'
              f'  {"(Zug)":>12}')
```
Ausgabe:

```text
%%AUSGABE4%%
```

Den Spannungsnachweis bestehen alle Stäbe mit großem Abstand, die höchste
Auslastung liegt unter zehn Prozent. Beim Knicken fallen aber die beiden
äußeren Obergurtstäbe 3 und 6 durch: Ihre Knicklast ist nur halb so groß wie
die Druckkraft. Der Dachbinder hält die Schneelast also nicht.
````

```{admonition} Abschlussfrage
:class: tip
Alle vier Obergurtstäbe tragen dieselbe Druckkraft. Trotzdem knicken nur die
Stäbe 3 und 6. Warum? Und warum müssen wir den Untergurt gar nicht auf
Knicken prüfen, obwohl seine Stabkräfte ähnlich groß sind? Schlagen Sie eine
konstruktive Änderung vor, mit der der Binder hält.
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
Die Knicklast $F_\text{krit} = \pi^2 E I / L^2$ sinkt mit dem Quadrat der
Stablänge. Die Stäbe 3 und 6 sind mit $2.40\,\text{m}$ doppelt so lang wie
die Stäbe 4 und 5 mit $1.20\,\text{m}$. Ihre Knicklast ist deshalb nur ein
Viertel so groß, und das reicht für die Druckkraft von $5408\,\text{N}$ nicht
mehr aus.

Der Untergurt steht unter Zug. Ein gezogener Stab wird gestreckt und kann
nicht seitlich ausweichen, Knicken ist nur bei Druck möglich.

Eine Abhilfe ist ein zusätzlicher Knoten in der Mitte der Stäbe 3 und 6, der
über weitere Stäbe mit dem Untergurt verbunden ist. Dann halbiert sich die
Knicklänge, und die Knicklast vervierfacht sich. Alternativ nehmen wir für den
Obergurt dickere Stäbe oder Rohre. Ein Rohr hat bei gleichem Material ein viel
größeres Flächenträgheitsmoment $I$ als ein Vollstab.
````

```{admonition} Zusatzaufgabe: Warum braucht der Binder ein Loslager? (✩✩✩)
:class: tip
Wir ersetzen das Loslager an Knoten 3 durch ein zweites Festlager, also
`lager_indizes = [0, 3]` und keine Loslager. Berechnen Sie Verschiebungen,
Stabkräfte und Lagerkräfte neu und vergleichen Sie mit der bisherigen
Lagerung.

1. Wie ändern sich die Kräfte im Untergurt?
2. Welche waagerechten Kräfte treten jetzt an den Lagern auf? Was bedeutet
   das für die Hallenwände?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Zusatzaufgabe
:class: tip
:class: dropdown
```python
# Zwei Festlager, kein Loslager
u_fest = berechne_verschiebungen(K, kraft_vektor, [0, 3])
stabkraefte_fest = berechne_stabkraefte(knoten_pos, staebe, elastizitaetsmodul,
                                        querschnitt, u_fest)
knotenkraefte_fest = K @ u_fest

# Teilaufgabe 1: Untergurt im Vergleich
print('Untergurt       Fest + Los    Fest + Fest')
for s in [0, 1, 2]:
    print(f'Stab {s}:   {stabkraefte[s]:10.1f} N  {stabkraefte_fest[s]:10.1f} N')

# Teilaufgabe 2: Lagerkräfte bei zwei Festlagern
for n in [0, 3]:
    print(f'Knoten {n}: Fx = {knotenkraefte_fest[2*n]:8.1f} N,  '
          f'Fy = {knotenkraefte_fest[2*n + 1]:8.1f} N')
```
Ausgabe:

```text
%%AUSGABEZ%%
```

Mit zwei Festlagern ist der Untergurt fast kraftlos, der mittlere Stab steht
sogar leicht unter Druck. Die Aufgabe des Zugbands übernehmen jetzt die
beiden Lager: Sie drücken mit rund $4200\,\text{N}$ waagerecht gegeneinander.
Umgekehrt drückt der Binder die beiden Hallenwände mit dieser Kraft nach
außen. Mauerwerk verträgt solche Schubkräfte schlecht. Außerdem würde jede
Temperaturänderung zusätzliche Zwangskräfte erzeugen, weil sich der Binder
nicht mehr frei ausdehnen kann. Deshalb liegt ein Binder auf einer Seite auf
einem Loslager.
````
