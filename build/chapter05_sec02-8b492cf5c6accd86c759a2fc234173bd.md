---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 5.2 Schwenkarm am Förderband

In Kapitel 5.1 haben wir mit der Drehmatrix berechnet, wohin sich der Arm
eines SCARA-Roboters dreht. In diesem Kapitel setzen wir sie in einer
Verpackungsanlage ein, in der ein Schwenkarm mit nur einem Gelenk arbeitet.
Bearbeiten Sie die Teilaufgaben möglichst zu zweit und der Reihe nach, jeder
Teil baut auf dem vorherigen auf.

````{admonition} Projekt: Schwenkarm am Förderband (✩✩)
:class: tip
In einer Verpackungsanlage kommen Gehäusedeckel auf einem Förderband an. Ein
Schwenkarm nimmt jeden Deckel vom Band und legt ihn in eine Kiste. Der Arm hat
ein einziges Drehgelenk im Ursprung und ist vom Gelenk bis zum Greifer
$L = 0.6\,\text{m}$ lang. In der Grundstellung $\alpha = 0$ zeigt er entlang
der $x$-Achse, und der Greifer steht über dem Deckel auf dem Band. Der Arm
kann gegen den Uhrzeigersinn von $0°$ bis $180°$ schwenken.

```{figure} pics/chap05_schwenkarm.svg
:alt: Draufsicht auf einen Schwenkarm mit Gelenk im Ursprung, der in Grundstellung über einem Deckel auf einem Förderband steht und auf einer gestrichelten Kreisbahn um den Winkel alpha zu einer Kiste schwenkt
:align: center

Der Schwenkarm von oben: Er nimmt den Deckel vom Förderband und schwenkt ihn
auf einer Kreisbahn in die Kiste.
(Quelle: eigene Abbildung; Lizenz [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0))
```

| Objekt | Lage und Größe |
| --- | --- |
| Gelenk | im Ursprung $(0, 0)$ |
| Greifer in Grundstellung | bei $(0.6, 0)$ |
| Deckel | $10\,\text{cm} \times 6\,\text{cm}$, Mitte unter dem Greifer, lange Kante längs zum Arm |
| Förderband | $20\,\text{cm}$ breit, von $x = 0.5\,\text{m}$ bis $x = 0.7\,\text{m}$ |
| Kiste | Innenmaß $16\,\text{cm} \times 16\,\text{cm}$, Mitte bei $(-0.46, 0.39)$, Kanten parallel zu den Achsen |
````

Führen Sie zuerst die beiden folgenden Zellen aus. Die erste enthält die
vorgegebene Zeichenfunktion aus Kapitel 5.1. Zwei ihrer Parameter haben wir
dort noch nicht genutzt: `umgebung` zeichnet Flächen wie Förderband und Kiste,
`bahn` zeichnet eine Folge von Greiferpositionen. Die zweite Zelle enthält die
Importe, die Funktion `drehmatrix` aus Kapitel 5.1 und die Punkte aus der
Tabelle.

```{code-cell} python
:tags: [hide-input]
@@ZEICHNE_ROBOTER@@
```

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Funktion aus Kapitel 5.1
def drehmatrix(winkel):
    """Drehmatrix für eine Drehung um den Ursprung, winkel in Radiant."""
    return np.array([
        [np.cos(winkel), -np.sin(winkel)],
        [np.sin(winkel),  np.cos(winkel)],
    ])

# Schwenkarm in Grundstellung in m, Zeile = [x, y]
arm = np.array([
    [0.0, 0.0],   # Gelenk
    [0.6, 0.0],   # Greifer
])

# Ecken des Deckels unter dem Greifer in m
deckel = np.array([
    [0.55, -0.03],
    [0.65, -0.03],
    [0.65,  0.03],
    [0.55,  0.03],
])

# Ecken von Förderband und Kiste (Innenmaß) in m
foerderband = np.array([[0.5, -0.8], [0.7, -0.8], [0.7, 0.8], [0.5, 0.8]])
kiste = np.array([[-0.54, 0.31], [-0.38, 0.31], [-0.38, 0.47], [-0.54, 0.47]])
kiste_mitte = np.array([-0.46, 0.39])

zeichne_roboter(arm, bauteile=deckel, ziele=kiste_mitte,
                umgebung=[foerderband, kiste], titel='Schwenkarm in Grundstellung')
```

```{admonition} Teil 1: Den Schwenk in Stellungen zeichnen
:class: tip
Drehen Sie Arm und Deckel um $45°$, $90°$ und $135°$. Sammeln Sie die Arme in
einer Liste `arme` und die Deckel in einer Liste `deckel_liste`, jeweils
beginnend mit der Grundstellung. Mit `liste.append(...)` aus Kapitel 1.3 hängen
Sie in einer Schleife ein Element an. Zeichnen Sie alle vier Stellungen mit

`zeichne_roboter(arme, bauteile=deckel_liste, umgebung=[foerderband, kiste], beschriftungen=[...])`.

Schätzen Sie aus dem Bild: Bei welchem Winkel ungefähr steht der Greifer über
der Kistenmitte?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
```python
# Grundstellung als erstes Element, dann drei gedrehte Stellungen anhängen
arme = [arm]
deckel_liste = [deckel]
for winkel in [45, 90, 135]:
    R = drehmatrix(np.deg2rad(winkel))
    arme.append((R @ arm.T).T)
    deckel_liste.append((R @ deckel.T).T)
    print(f'{winkel:3d}°: Greifer bei {np.round(arme[-1][1], 4)} m')

zeichne_roboter(arme, bauteile=deckel_liste, umgebung=[foerderband, kiste],
                beschriftungen=['0°', '45°', '90°', '135°'],
                titel='Schwenkarm in vier Stellungen')
```
Der Greifer wandert auf einem Kreis mit Radius $0.6\,\text{m}$ über
$(0.424, 0.424)$ und $(0, 0.6)$ nach $(-0.424, 0.424)$. Bei $135°$ steht er
knapp neben der rechten unteren Ecke der Kiste. Der Ablagewinkel liegt also
etwas über $135°$. Der Deckel dreht sich bei jedem Schwenk mit: Bei $90°$ liegt
seine lange Kante parallel zur $y$-Achse.
````

```{admonition} Teil 2: Den Ablagewinkel berechnen
:class: tip
Statt zu schätzen, probieren wir alle Winkel durch. Legen Sie mit `np.linspace`
die Winkel von $0°$ bis $180°$ in Schritten von $1°$ an. Berechnen Sie in einer
Schleife für jeden Winkel die Greiferposition und mit Pythagoras ihren Abstand
zur Kistenmitte. Speichern Sie die Abstände in einem Array `abstaende`, das Sie
vorher mit `np.zeros` anlegen.

`np.argmin` funktioniert wie `np.argmax` aus Kapitel 2.4, liefert aber den
Index des kleinsten Werts. Bestimmen Sie damit den Ablagewinkel. Wie weit ist
der Greifer dort noch von der Kistenmitte entfernt? Zeichnen Sie den Arm in
Ablagestellung mit der Kistenmitte als Ziel.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 2
:class: tip
:class: dropdown
```python
# Schwenkwinkel von 0 bis 180 Grad in 1-Grad-Schritten
winkel_grad = np.linspace(0, 180, 181)
abstaende = np.zeros(len(winkel_grad))

for k in range(len(winkel_grad)):
    greifer = drehmatrix(np.deg2rad(winkel_grad[k])) @ arm[1]
    differenz = greifer - kiste_mitte
    abstaende[k] = np.sqrt(differenz[0]**2 + differenz[1]**2)

# Index des kleinsten Abstands liefert den Ablagewinkel
k_best = np.argmin(abstaende)
ablagewinkel = winkel_grad[k_best]
print(f'Ablagewinkel: {ablagewinkel:.0f}°')
print(f'Restabstand zur Kistenmitte: {abstaende[k_best] * 1000:.1f} mm')

R_ablage = drehmatrix(np.deg2rad(ablagewinkel))
arm_ablage = (R_ablage @ arm.T).T
zeichne_roboter(arm_ablage, ziele=kiste_mitte, umgebung=[foerderband, kiste],
                titel=f'Ablagestellung bei {ablagewinkel:.0f}°')
```
Ausgabe:
```
Ablagewinkel: 140°
Restabstand zur Kistenmitte: 4.3 mm
```
Bei $140°$ steht der Greifer bei $(-0.460, 0.386)$, nur $4.3\,\text{mm}$ neben
der Kistenmitte. Ein feineres Winkelraster würde diesen Rest etwas verkleinern,
aber nie ganz beseitigen. Die Kistenmitte liegt nämlich $0.603\,\text{m}$ vom
Gelenk entfernt und damit $3\,\text{mm}$ außerhalb des Kreises, auf dem sich
der Greifer bewegt.
````

```{admonition} Teil 3: Liegt der Deckel in der Kiste?
:class: tip
Drehen Sie den Deckel um den Ablagewinkel und speichern Sie das Ergebnis in
`deckel_ablage`. Prüfen Sie dann, ob alle vier Ecken innerhalb der Kiste
liegen. `deckel_ablage[:, 0]` liefert die Spalte 0, also die $x$-Koordinaten
aller Ecken, `deckel_ablage[:, 1]` die $y$-Koordinaten. Vergleichen Sie die
kleinsten und größten Koordinaten des Deckels mit denen der Kiste.

Wie viel Platz bleibt mindestens zwischen Deckel und Kistenwand? Zeichnen Sie
Abhol- und Ablagestellung mit Deckel in ein Bild.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 3
:class: tip
:class: dropdown
```python
# Deckel in Ablagestellung
deckel_ablage = (R_ablage @ deckel.T).T

# Kleinste und größte Koordinaten des Deckels
x_min = np.min(deckel_ablage[:, 0])
x_max = np.max(deckel_ablage[:, 0])
y_min = np.min(deckel_ablage[:, 1])
y_max = np.max(deckel_ablage[:, 1])

# Kleinste und größte Koordinaten der Kiste
kiste_x_min = np.min(kiste[:, 0])
kiste_x_max = np.max(kiste[:, 0])
kiste_y_min = np.min(kiste[:, 1])
kiste_y_max = np.max(kiste[:, 1])

passt = (x_min >= kiste_x_min and x_max <= kiste_x_max
         and y_min >= kiste_y_min and y_max <= kiste_y_max)
print('Deckel liegt ganz in der Kiste:', passt)

# Abstände zu den vier Kistenwänden: links, rechts, unten, oben
raender = np.array([x_min - kiste_x_min, kiste_x_max - x_max,
                    y_min - kiste_y_min, kiste_y_max - y_max])
print(f'Kleinster Abstand zur Wand: {np.min(raender) * 100:.1f} cm')

zeichne_roboter([arm, arm_ablage], bauteile=[deckel, deckel_ablage],
                umgebung=[foerderband, kiste], beschriftungen=['Abholen', 'Ablegen'],
                titel='Deckel in der Kiste')
```
Ausgabe:
```
Deckel liegt ganz in der Kiste: True
Kleinster Abstand zur Wand: 2.1 cm
```
Der Deckel passt in die Kiste, an der unteren Wand bleiben noch
$2.1\,\text{cm}$ Platz. Der Vergleich der kleinsten und größten Koordinaten
genügt hier, weil die Kistenwände parallel zu den Achsen liegen. Auffällig ist
die Lage des Deckels: Er hat sich mit dem Arm um $140°$ gedreht und liegt
schräg in der Kiste.
````

```{admonition} Teil 4: Wie lang ist der Weg des Greifers?
:class: tip
Für die Taktzeit der Anlage brauchen wir die Länge des Wegs, den der Greifer
vom Band zur Kiste zurücklegt. Wir nähern den Kreisbogen durch gerade Stücke
an. Schreiben Sie dazu zwei Funktionen:

1. `greiferbahn(anzahl_punkte)` erzeugt mit `np.linspace` gleichmäßig
   verteilte Winkel von $0°$ bis zum Ablagewinkel und gibt die
   Greiferpositionen als Array der Form `(anzahl_punkte, 2)` zurück.
2. `weglaenge(bahn)` addiert die geraden Abstände zwischen aufeinanderfolgenden
   Punkten der Bahn, jeweils mit Pythagoras.

Berechnen Sie die Weglänge für 8, 29 und 141 Punkte, also für Winkelschritte
von $20°$, $5°$ und $1°$. Vergleichen Sie mit der exakten Länge des
Kreisbogens $L \cdot \alpha$, wobei der Winkel $\alpha$ in Radiant einzusetzen
ist. Zeichnen Sie die Bahn mit 8 Punkten über den Parameter `bahn` ein.

Warum ist der Weg aus geraden Stücken immer kürzer als der Kreisbogen?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 4
:class: tip
:class: dropdown
```python
def greiferbahn(anzahl_punkte):
    """Greiferpositionen beim Schwenk von 0 Grad bis zum Ablagewinkel."""
    winkel = np.linspace(0, ablagewinkel, anzahl_punkte)
    bahn = np.zeros((anzahl_punkte, 2))
    for k in range(anzahl_punkte):
        bahn[k] = drehmatrix(np.deg2rad(winkel[k])) @ arm[1]
    return bahn

def weglaenge(bahn):
    """Summe der geraden Abstände zwischen aufeinanderfolgenden Punkten."""
    laenge = 0.0
    for k in range(len(bahn) - 1):
        differenz = bahn[k + 1] - bahn[k]
        laenge += np.sqrt(differenz[0]**2 + differenz[1]**2)
    return laenge

# Exakte Bogenlänge: Radius mal Winkel in Radiant
bogen = 0.6 * np.deg2rad(ablagewinkel)
print(f'Kreisbogen exakt: {bogen:.5f} m')

for anzahl in [8, 29, 141]:
    weg = weglaenge(greiferbahn(anzahl))
    print(f'{anzahl:3d} Punkte: {weg:.5f} m, Fehler {(bogen - weg) * 1000:.2f} mm')

zeichne_roboter([arm, arm_ablage], bauteile=[deckel, deckel_ablage],
                bahn=greiferbahn(8), umgebung=[foerderband, kiste],
                beschriftungen=['Abholen', 'Ablegen'], titel='Greiferbahn mit 8 Punkten')
```
Ausgabe:
```
Kreisbogen exakt: 1.46608 m
  8 Punkte: 1.45864 m, Fehler 7.43 mm
 29 Punkte: 1.46561 m, Fehler 0.47 mm
141 Punkte: 1.46606 m, Fehler 0.02 mm
```
Der Greifer legt knapp $1.47\,\text{m}$ zurück. Jedes gerade Stück ist eine
Sehne des Kreises und damit kürzer als der zugehörige Bogen, deshalb ist die
Summe immer etwas zu klein. Mit kleineren Winkelschritten wird die Näherung
schnell besser: Viertelt sich die Schrittweite von $20°$ auf $5°$, sinkt der
Fehler von $7.43\,\text{mm}$ auf $0.47\,\text{mm}$, also etwa auf ein
Sechzehntel. Der Fehler fällt mit dem Quadrat der Schrittweite. Solche
Fehlerordnungen untersuchen wir in den späteren Parts zur numerischen
Differentiation und Integration genauer.
````

```{admonition} Abschlussfrage
:class: tip
1. Die Kistenmitte liegt $0.603\,\text{m}$ vom Gelenk entfernt. Was würde
   passieren, wenn die Kiste näher am Sockel stünde, zum Beispiel mit der
   Mitte bei $(-0.30, 0.30)$?
2. Der Deckel liegt schräg in der Kiste. Zum Stapeln soll seine lange Kante
   parallel zur $x$-Achse liegen. Warum kann der Schwenkarm das nicht leisten,
   und welche zusätzliche Bewegung bräuchte er?
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
1. Der Arm hat nur ein Gelenk. Der Greifer bewegt sich deshalb immer auf einem
   Kreis mit Radius $0.6\,\text{m}$ um das Gelenk, und seine Position ist
   allein durch den Winkel $\alpha$ festgelegt. Eine Kistenmitte bei
   $(-0.30, 0.30)$ liegt nur $0.424\,\text{m}$ vom Gelenk entfernt. Der Greifer
   käme ihr nie näher als $0.6\,\text{m} - 0.424\,\text{m} = 0.176\,\text{m}$
   und würde die Kiste verfehlen. Um Punkte in einer ganzen Fläche zu
   erreichen, braucht der Arm ein zweites Gelenk am Ende des ersten Arms, wie
   der SCARA-Roboter aus Kapitel 5.1.
2. Die Orientierung des Deckels ist fest an den Schwenkwinkel gekoppelt: Wenn
   der Arm um $140°$ schwenkt, dreht sich auch der Deckel um $140°$. Um ihn
   gerade abzulegen, müsste der Deckel zusätzlich um seinen eigenen Mittelpunkt
   gedreht werden, wegen seiner Symmetrie genügen dafür $40°$. Echte Anlagen
   haben dafür einen drehbaren Greifer. Unsere Drehmatrix dreht aber nur um
   den Ursprung. Eine Drehung um einen anderen Punkt setzt sich aus Drehen und
   Verschieben zusammen. Wie wir beides in einer einzigen Matrix beschreiben,
   lernen wir in Kapitel 5.3.
````

```{admonition} Zusatzaufgabe: Zwischenstopp an der Prüfkamera (✩✩✩)
:class: tip
Auf dem Weg zur Kiste hält der Arm bei $60°$ an einer Kamera, die den Deckel
prüft. Der Schwenk besteht dann aus zwei Drehungen: erst
$\mathbf{R}_1 = \mathbf{R}(60°)$ vom Band zur Kamera, dann
$\mathbf{R}_2 = \mathbf{R}(80°)$ von der Kamera zur Kiste. Wird erst
$\mathbf{R}_1$ und dann $\mathbf{R}_2$ angewendet, gilt
$\mathbf{R}_2 \cdot (\mathbf{R}_1 \cdot \vec{p}) = (\mathbf{R}_2 \cdot \mathbf{R}_1) \cdot \vec{p}$.
Die zuerst ausgeführte Drehung steht also rechts.

1. Berechnen Sie die Greiferposition an der Kamera.
2. Berechnen Sie das Produkt $\mathbf{R}_2 \cdot \mathbf{R}_1$ und vergleichen
   Sie es mit `np.allclose` mit $\mathbf{R}(140°)$.
3. Vertauschen Sie die Reihenfolge zu $\mathbf{R}_1 \cdot \mathbf{R}_2$.
   Ändert sich das Ergebnis?
4. Begründen Sie ohne Code, warum die Reihenfolge hier keine Rolle spielt.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Zusatzaufgabe
:class: tip
:class: dropdown
```python
R_1 = drehmatrix(np.deg2rad(60))   # vom Band zur Kamera
R_2 = drehmatrix(np.deg2rad(80))   # von der Kamera zur Kiste

# Teilaufgabe 1: Greifer an der Kamera
greifer_kamera = R_1 @ arm[1]
print('Greifer an der Kamera in m:', np.round(greifer_kamera, 4))

# Teilaufgabe 2: beide Drehungen zusammen
R_gesamt = R_2 @ R_1
print('R_2 R_1 gleich R(140°):', np.allclose(R_gesamt, drehmatrix(np.deg2rad(140))))

# Teilaufgabe 3: Reihenfolge vertauscht
print('R_1 R_2 gleich R_2 R_1:', np.allclose(R_1 @ R_2, R_2 @ R_1))
```
Ausgabe:
```
Greifer an der Kamera in m: [0.3    0.5196]
R_2 R_1 gleich R(140°): True
R_1 R_2 gleich R_2 R_1: True
```
An der Kamera steht der Greifer bei $(0.300, 0.520)$. Zwei Drehungen
hintereinander ergeben eine einzige Drehung um die Summe der Winkel,
$60° + 80° = 140°$. Beide Drehungen gehen um denselben Punkt, das Gelenk.
Weil es bei der Summe der Winkel nicht auf die Reihenfolge ankommt, ist auch
$\mathbf{R}_1 \cdot \mathbf{R}_2 = \mathbf{R}_2 \cdot \mathbf{R}_1$. Das ist
eine Besonderheit von Drehungen um denselben Punkt in der Ebene. Im
Allgemeinen ist die Matrixmultiplikation nicht vertauschbar. In Kapitel 5.3
kommen Verschiebungen dazu, und dann spielt die Reihenfolge sehr wohl eine
Rolle.
````
