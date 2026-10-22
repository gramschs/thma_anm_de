---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 5.4 Eine Platine bestücken

In Kapitel 5.3 haben wir aus den beiden Gelenkwinkeln des SCARA-Roboters
berechnet, wo Greifer und Bauteil stehen. In diesem Projekt drehen wir die
Frage um: Welche Gelenkwinkel bringen ein Bauteil an einen vorgegebenen Platz
auf der Platine? Bearbeiten Sie die Teilaufgaben möglichst zu zweit und der
Reihe nach, jeder Teil baut auf dem vorherigen auf.

````{admonition} Projekt: Eine Platine bestücken (✩✩)
:class: tip
Der SCARA-Roboter aus Kapitel 5.1 und 5.3 bestückt eine Platine mit Bauteilen.
Er holt jedes Bauteil in der Grundstellung $\alpha = \beta = 0$ an einer
Zuführung ab und setzt es auf einen Bestückplatz. Das Bauteil sitzt mittig
unter dem Greifer. Gelenk 1 dreht zwischen $0°$ und $180°$. Gelenk 2 darf nur
gegen den Uhrzeigersinn abknicken, also zwischen $0°$ und $150°$.

| Objekt | Lage und Größe |
| --- | --- |
| Gelenk 1 | im Ursprung $(0, 0)$ |
| Arm 1 und Arm 2 | $l_1 = 0.4\,\text{m}$ und $l_2 = 0.3\,\text{m}$ lang |
| Bauteil | $8\,\text{cm} \times 4\,\text{cm}$, in Grundstellung längs zum Arm |
| Zuführung | $x$ von $0.62\,\text{m}$ bis $0.78\,\text{m}$, $y$ von $-0.06\,\text{m}$ bis $0.06\,\text{m}$ |
| Platine | $x$ von $0.25\,\text{m}$ bis $0.50\,\text{m}$, $y$ von $0.25\,\text{m}$ bis $0.45\,\text{m}$ |
| Bestückplätze | $P_1 = (0.30, 0.35)$, $P_2 = (0.40, 0.40)$, $P_3 = (0.45, 0.30)$ |
````

Führen Sie zuerst die beiden folgenden Zellen aus. Die erste enthält die
vorgegebene Zeichenfunktion aus Kapitel 5.1. Die zweite enthält die Importe,
die drei Funktionen `transformationsmatrix`, `transformiere` und
`drehung_um_punkt` aus Kapitel 5.3 und die Daten aus der Tabelle.

```{code-cell} python
:tags: [hide-input]
# Vorgegebene Zeichenfunktion: einfach ausführen, sie muss nicht verstanden werden.
def zeichne_roboter(arme, bauteile=None, ziele=None, bahn=None, umgebung=None,
                    beschriftungen=None, titel='', grenze=0.8):
    """Zeichnet einen Roboterarm in der Draufsicht.

    arme: Punkte eines Arms als Array, Zeile = [x, y], vom ersten Gelenk bis
          zum Greifer, oder eine Liste solcher Arrays für mehrere Stellungen
    bauteile: Eckpunkte des Bauteils im Greifer als Array, oder eine Liste
              mit einem Array pro Stellung (optional)
    ziele: Zielpunkte als Array, Zeile = [x, y], werden als Kreuze markiert (optional)
    bahn: Greiferpositionen als Array, Zeile = [x, y], werden als gestrichelte
          Linie mit Punkten gezeichnet (optional)
    umgebung: Eckpunkte von Flächen wie Förderband oder Kiste als Array oder
              Liste von Arrays, werden grau gefüllt (optional)
    beschriftungen: Legendentexte, einer pro Stellung (optional)
    titel: Diagrammtitel
    grenze: sichtbarer Bereich von -grenze bis +grenze in m
    """
    blau, rot, orange, grau = '#005A94', '#E60000', '#E87846', '#484949'
    farben = [blau, rot, orange, grau]
    if isinstance(arme, np.ndarray):
        arme = [arme]
    if isinstance(bauteile, np.ndarray):
        bauteile = [bauteile]
    if isinstance(umgebung, np.ndarray):
        umgebung = [umgebung]
    fig, ax = plt.subplots(figsize=(6, 6))

    # Umgebung: Flächen wie Förderband oder Kiste
    if umgebung is not None:
        for flaeche in umgebung:
            ax.fill(flaeche[:, 0], flaeche[:, 1], facecolor='#DDDEDE',
                    edgecolor=grau, linewidth=1, zorder=0)

    # Sockel im Ursprung
    ax.add_patch(plt.Circle((0, 0), 0.06, facecolor='#CCDEE9', edgecolor=blau,
                            linewidth=1.5, zorder=1))

    # Greiferbahn als gestrichelte Linie mit Punkten
    if bahn is not None:
        ax.plot(bahn[:, 0], bahn[:, 1], color=grau, linestyle='--', linewidth=1,
                marker='o', markersize=3, label='Greiferbahn', zorder=1)

    # Arme: Gelenke als Kreise, Greifer als Quadrat, Bauteil gelb gefüllt
    for k in range(len(arme)):
        farbe = farben[k % len(farben)]
        punkte = arme[k]
        label = None if beschriftungen is None else beschriftungen[k]
        ax.plot(punkte[:, 0], punkte[:, 1], color=farbe, linewidth=6,
                solid_capstyle='round', label=label, zorder=2)
        ax.scatter(punkte[:-1, 0], punkte[:-1, 1], s=50, color='white',
                   edgecolor=farbe, linewidth=2, zorder=3)
        ax.scatter(punkte[-1, 0], punkte[-1, 1], s=60, marker='s', color=farbe,
                   zorder=3)
        if bauteile is not None:
            ecken = bauteile[k]
            ax.fill(ecken[:, 0], ecken[:, 1], facecolor='#FFEC7F',
                    edgecolor=farbe, linewidth=1.5, zorder=4)

    # Zielpunkte als Kreuze
    if ziele is not None:
        ziele = np.atleast_2d(ziele)
        ax.scatter(ziele[:, 0], ziele[:, 1], marker='x', s=80, color=grau,
                   linewidth=2, zorder=5, label='Ziel')

    if beschriftungen is not None or ziele is not None or bahn is not None:
        ax.legend(loc='lower left')
    ax.set_xlim(-grenze, grenze)
    ax.set_ylim(-grenze, grenze)
    ax.set_aspect('equal')
    ax.grid(True)
    ax.set_xlabel('x in m')
    ax.set_ylabel('y in m')
    ax.set_title(titel)
    plt.show()
```

```{code-cell} python
import numpy as np
import matplotlib.pyplot as plt

# Funktionen aus Kapitel 5.3
def transformationsmatrix(winkel, tx, ty):
    """Homogene Matrix: um winkel (Radiant) drehen, dann um (tx, ty) verschieben."""
    return np.array([
        [np.cos(winkel), -np.sin(winkel), tx],
        [np.sin(winkel),  np.cos(winkel), ty],
        [0.0,             0.0,            1.0],
    ])

def transformiere(T, punkte):
    """Wendet die homogene Matrix T auf alle Punkte an, Zeile = [x, y]."""
    punkte_hom = np.ones((len(punkte), 3))
    punkte_hom[:, 0:2] = punkte
    ergebnis = (T @ punkte_hom.T).T
    return ergebnis[:, 0:2]

def drehung_um_punkt(winkel, zentrum):
    """Homogene Matrix für eine Drehung um den Punkt zentrum, winkel in Radiant."""
    T_hin = transformationsmatrix(0, -zentrum[0], -zentrum[1])
    T_dreh = transformationsmatrix(winkel, 0, 0)
    T_zurueck = transformationsmatrix(0, zentrum[0], zentrum[1])
    return T_zurueck @ T_dreh @ T_hin

# Punkte des Arms in Grundstellung in m, Zeile = [x, y]
arm = np.array([
    [0.0, 0.0],   # Gelenk 1
    [0.4, 0.0],   # Gelenk 2
    [0.7, 0.0],   # Greifer
])

# Ecken des Bauteils im Greifer in m, Grundstellung
bauteil = np.array([
    [0.66, -0.02],
    [0.74, -0.02],
    [0.74,  0.02],
    [0.66,  0.02],
])

# Ecken von Zuführung und Platine in m
zufuehrung = np.array([[0.62, -0.06], [0.78, -0.06], [0.78, 0.06], [0.62, 0.06]])
platine = np.array([[0.25, 0.25], [0.50, 0.25], [0.50, 0.45], [0.25, 0.45]])

# Bestückplätze P1, P2, P3 in m, Zeile = [x, y]
bestueckplaetze = np.array([
    [0.30, 0.35],
    [0.40, 0.40],
    [0.45, 0.30],
])

zeichne_roboter(arm, bauteile=bauteil, ziele=bestueckplaetze,
                umgebung=[zufuehrung, platine], titel='Bestückzelle in Grundstellung')
```

```{admonition} Teil 1: Die Stellung des Roboters berechnen
:class: tip
Schreiben Sie eine Funktion `stellung(alpha, beta)`, die für die Gelenkwinkel
$\alpha$ und $\beta$ in Radiant die Punkte des Arms als Array der Form `(3, 2)`
und die Ecken des Bauteils als Array der Form `(4, 2)` zurückgibt. Gehen Sie
vor wie in Kapitel 5.3: Arm 1 transformieren Sie mit $\mathbf{T}_\alpha$, Arm 2
und das Bauteil mit $\mathbf{T}_\alpha \cdot \mathbf{T}_\beta$. Eine Funktion
kann mehrere Ergebnisse zurückgeben. `return arm_neu, bauteil_neu` gibt beide
Arrays zurück, und `arm_neu, bauteil_neu = stellung(alpha, beta)` nimmt sie
entgegen.

Prüfen Sie Ihre Funktion mit der Stellung $\alpha = 30°$, $\beta = 60°$ aus
Kapitel 5.3. Der Greifer muss bei $(0.346, 0.500)$ stehen. Zeichnen Sie dann
die Grundstellung und die Stellungen $(30°, 60°)$ und $(60°, 90°)$ zusammen mit
Zuführung und Platine.

Die Berechnung der Stellung aus den Gelenkwinkeln heißt **Vorwärtskinematik**.
Warum ist sie eindeutig, gehört also zu jedem Winkelpaar genau eine Stellung?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 1
:class: tip
:class: dropdown
```python
def stellung(alpha, beta):
    """Punkte des Arms und Ecken des Bauteils für die Gelenkwinkel in Radiant."""
    T_alpha = transformationsmatrix(alpha, 0, 0)
    T_arm2 = T_alpha @ drehung_um_punkt(beta, arm[1])
    # Arm 1 nur mit T_alpha, Arm 2 und Bauteil mit beiden Drehungen
    arm1_neu = transformiere(T_alpha, arm[0:2])
    arm2_neu = transformiere(T_arm2, arm[1:3])
    arm_neu = np.array([arm1_neu[0], arm1_neu[1], arm2_neu[1]])
    bauteil_neu = transformiere(T_arm2, bauteil)
    return arm_neu, bauteil_neu

# Prüfung mit der Stellung aus Kapitel 5.3
arm_test, bauteil_test = stellung(np.deg2rad(30), np.deg2rad(60))
print('Greifer bei (30°, 60°) in m:', np.round(arm_test[2], 4))

# Drei Stellungen sammeln und zeichnen
arme = []
bauteile = []
for winkelpaar in [[0, 0], [30, 60], [60, 90]]:
    arm_neu, bauteil_neu = stellung(np.deg2rad(winkelpaar[0]), np.deg2rad(winkelpaar[1]))
    arme.append(arm_neu)
    bauteile.append(bauteil_neu)
    print(f'α = {winkelpaar[0]:2d}°, β = {winkelpaar[1]:2d}°: '
          f'Greifer bei {np.round(arm_neu[2], 4)} m')

zeichne_roboter(arme, bauteile=bauteile, umgebung=[zufuehrung, platine],
                beschriftungen=['Grundstellung', '(30°, 60°)', '(60°, 90°)'],
                titel='Drei Stellungen des SCARA-Roboters')
```
Ausgabe:

```text
Greifer bei (30°, 60°) in m: [0.3464 0.5   ]
α =  0°, β =  0°: Greifer bei [0.7 0. ] m
α = 30°, β = 60°: Greifer bei [0.3464 0.5   ] m
α = 60°, β = 90°: Greifer bei [-0.0598  0.4964] m
```
Die Funktion liefert für $(30°, 60°)$ denselben Greifer wie Kapitel 5.3. Bei
$(60°, 90°)$ steht der Greifer links neben der $y$-Achse, weit weg von der
Platine. Die Vorwärtskinematik ist eindeutig, weil jeder Rechenschritt eindeutig
ist: Aus den Winkeln entstehen genau eine Matrix $\mathbf{T}_\alpha$ und genau
eine Matrix $\mathbf{T}_\beta$, und deren Produkt bringt jeden Punkt an genau
eine Stelle.
````

```{admonition} Teil 2: Die Winkel für einen Bestückplatz suchen
:class: tip
Die Steuerung kennt nur den Bestückplatz $P_1 = (0.30, 0.35)$ und muss die
Gelenkwinkel selbst finden. Diese Umkehrung der Vorwärtskinematik heißt
**inverse Kinematik**. Wir lösen sie wie in Kapitel 5.2 durch Ausprobieren,
jetzt aber für zwei Winkel.

Durchlaufen Sie in zwei verschachtelten Schleifen alle $\alpha$ von $0°$ bis
$180°$ und alle $\beta$ von $0°$ bis $150°$ in Schritten von $1°$. Berechnen Sie
jeweils mit `stellung` die Lage des Greifers und mit Pythagoras ihren Abstand
zu $P_1$. Merken Sie sich mit einer `if`-Abfrage den bisher kleinsten Abstand
und die zugehörigen Winkel. Starten Sie mit `bester_abstand = 10.0`, also mit
einem Wert, der größer ist als jeder mögliche Abstand.

Welche Winkel findet die Suche, und wie weit ist der Greifer dort noch von
$P_1$ entfernt? Zeichnen Sie die gefundene Stellung mit $P_1$ als Ziel.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 2
:class: tip
:class: dropdown
```python
ziel = bestueckplaetze[0]

# Startwert größer als jeder mögliche Abstand
bester_abstand = 10.0
for alpha_grad in np.linspace(0, 180, 181):
    for beta_grad in np.linspace(0, 150, 151):
        arm_neu, bauteil_neu = stellung(np.deg2rad(alpha_grad), np.deg2rad(beta_grad))
        differenz = arm_neu[2] - ziel
        abstand = np.sqrt(differenz[0]**2 + differenz[1]**2)
        # Kleinsten Abstand und zugehörige Winkel merken
        if abstand < bester_abstand:
            bester_abstand = abstand
            bestes_alpha = alpha_grad
            bestes_beta = beta_grad

print(f'Gefundene Winkel: α = {bestes_alpha:.0f}°, β = {bestes_beta:.0f}°')
print(f'Restabstand zu P1: {bester_abstand * 1000:.1f} mm')

arm_p1, bauteil_p1 = stellung(np.deg2rad(bestes_alpha), np.deg2rad(bestes_beta))
zeichne_roboter(arm_p1, bauteile=bauteil_p1, ziele=ziel,
                umgebung=[zufuehrung, platine], titel='Stellung für P1')
```
Ausgabe:

```text
Gefundene Winkel: α = 9°, β = 99°
Restabstand zu P1: 3.2 mm
```
Bei $\alpha = 9°$ und $\beta = 99°$ verfehlt der Greifer $P_1$ um
$3.2\,\text{mm}$. Anders als die Kistenmitte in Kapitel 5.2 kann der Greifer
$P_1$ aber genau erreichen: Mit den exakten Winkeln $\alpha \approx 9.4°$ und
$\beta \approx 99.0°$ träfe er den Punkt ohne Rest. Die $3.2\,\text{mm}$ sind
hier also allein ein Diskretisierungsfehler des $1°$-Rasters. Für einen
Bestückroboter ist das viel zu ungenau, Bauteile müssen auf Bruchteile eines
Millimeters genau sitzen. Ein zehnmal feineres Raster würde helfen, kostet aber
viel Zeit. Wir haben schon $181 \cdot 151 = 27\,331$ Stellungen berechnet. Mit
Schritten von $0.1°$ wären es $1801 \cdot 1501$, also rund 2.7 Millionen
Stellungen und damit hundertmal so viele.

Die Rastersuche ist nur die einfachste Methode, die inverse Kinematik
numerisch zu lösen. Für einen Arm mit zwei Gelenken lassen sich die Winkel
sogar exakt berechnen, mit dem Kosinussatz im Dreieck aus Arm 1, Arm 2 und der
Verbindungslinie zum Ziel. Bei komplizierteren Robotern gibt es solche Formeln
nicht immer. In Part 8 lernen wir mit dem Gradientenverfahren eine numerische
Methode kennen, die das Minimum gezielt ansteuert, statt alle Kombinationen
auszuprobieren.
````

```{admonition} Teil 3: Alle Bestückplätze anfahren
:class: tip
Verpacken Sie die Suche aus Teil 2 in eine Funktion `suche_winkel(ziel)`, die
die besten Winkel in Grad und den Restabstand zurückgibt. Bestimmen Sie damit
in einer Schleife die Winkel für alle drei Bestückplätze. Geben Sie für jeden
Platz die Winkel, den Restabstand und den Winkel $\alpha + \beta$ aus, um den
das Bauteil gegenüber der Grundstellung gedreht ist. Zeichnen Sie die drei
Stellungen mit ihren Bauteilen in ein Bild.
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Teil 3
:class: tip
:class: dropdown
```python
def suche_winkel(ziel):
    """Sucht die Gelenkwinkel in Grad, die den Greifer am nächsten an ziel bringen."""
    bester_abstand = 10.0
    for alpha_grad in np.linspace(0, 180, 181):
        for beta_grad in np.linspace(0, 150, 151):
            arm_neu, bauteil_neu = stellung(np.deg2rad(alpha_grad), np.deg2rad(beta_grad))
            differenz = arm_neu[2] - ziel
            abstand = np.sqrt(differenz[0]**2 + differenz[1]**2)
            if abstand < bester_abstand:
                bester_abstand = abstand
                bestes_alpha = alpha_grad
                bestes_beta = beta_grad
    return bestes_alpha, bestes_beta, bester_abstand

arme = []
bauteile = []
for k in range(len(bestueckplaetze)):
    alpha_grad, beta_grad, abstand = suche_winkel(bestueckplaetze[k])
    arm_neu, bauteil_neu = stellung(np.deg2rad(alpha_grad), np.deg2rad(beta_grad))
    arme.append(arm_neu)
    bauteile.append(bauteil_neu)
    print(f'P{k + 1}: α = {alpha_grad:3.0f}°, β = {beta_grad:3.0f}°, '
          f'Rest {abstand * 1000:.1f} mm, Bauteil um {alpha_grad + beta_grad:.0f}° gedreht')

zeichne_roboter(arme, bauteile=bauteile, ziele=bestueckplaetze,
                umgebung=[zufuehrung, platine], beschriftungen=['P1', 'P2', 'P3'],
                titel='Alle Bestückplätze')
```
Ausgabe:

```text
P1: α =   9°, β =  99°, Rest 3.2 mm, Bauteil um 108° gedreht
P2: α =  14°, β =  74°, Rest 3.7 mm, Bauteil um 88° gedreht
P3: α =   1°, β =  79°, Rest 3.2 mm, Bauteil um 80° gedreht
```
Alle drei Bauteile landen nahe ihrer Bestückplätze, verfehlen sie aber um
$3.2$ bis $3.7\,\text{mm}$. Wie in Teil 2 ist das ein Diskretisierungsfehler
des Rasters und für die Bestückung zu ungenau. Außerdem liegen die Bauteile
schräg und jedes anders gedreht auf der Platine. Unser Roboter hat keine
Drehachse am Greifer, das Bauteil sitzt also starr an Arm 2. Seine Längsachse
zeigt deshalb immer in Richtung von Arm 2. Arm 1 ist um $\alpha$ gegen die
$x$-Achse gedreht, und Arm 2 knickt um $\beta$ weiter ab. Das Bauteil ist
damit immer um $\alpha + \beta$ gedreht. Welche Winkel die Steuerung wählt,
legt also nicht nur die Lage, sondern auch die Orientierung des Bauteils fest.
````

```{admonition} Abschlussfrage
:class: tip
1. Wir haben für Gelenk 2 nur Winkel zwischen $0°$ und $150°$ zugelassen. Gibt
   es eine zweite Stellung, die den Greifer ebenfalls nach $P_1$ bringt? Wie
   sieht sie aus, und warum legt sich die Steuerung auf eine der beiden fest?
2. Warum kann der Roboter mit seinen zwei Gelenken die Lage und die
   Orientierung des Bauteils nicht unabhängig voneinander einstellen?
```

````{admonition} Lösung Abschlussfrage
:class: tip
:class: dropdown
1. Ja. Gelenk 1, Gelenk 2 und der Greifer bilden ein Dreieck mit den
   Seitenlängen $0.4\,\text{m}$, $0.3\,\text{m}$ und dem Abstand von $P_1$ zum
   Ursprung, $0.461\,\text{m}$. Dieses Dreieck lässt sich an der Verbindungslinie
   vom Ursprung zu $P_1$ spiegeln. In der gespiegelten Stellung knickt Gelenk 2
   im Uhrzeigersinn ab, ungefähr mit $\alpha = 89°$ und $\beta = -99°$. Ließe die
   Steuerung beide Vorzeichen von $\beta$ zu, könnte die Suche je nach Raster
   mal die eine, mal die andere Stellung finden. Der Arm würde dann zwischen
   zwei benachbarten Plätzen womöglich ganz umschwenken. Deshalb arbeitet die
   Steuerung eines SCARA-Roboters mit einer fest gewählten Armstellung.
2. Das Bauteil ist immer um $\alpha + \beta$ gedreht. Der Bestückplatz legt
   $\alpha$ und $\beta$ bis auf die Spiegelung bereits fest, und damit auch die
   Orientierung. Ein Bauteil in der Ebene hat aber drei Freiheitsgrade: die
   beiden Koordinaten und den Drehwinkel. Um alle drei unabhängig einzustellen,
   braucht der Roboter eine dritte Drehachse. Echte SCARA-Roboter haben sie
   direkt am Greifer, sie dreht das Bauteil um seine eigene Mitte. Mit
   `drehung_um_punkt` aus Kapitel 5.3 könnten wir auch diese Drehung als
   homogene Matrix an die Kette anhängen.
````

```{admonition} Zusatzaufgabe: Fährt der Greifer geradeaus? (✩✩✩)
:class: tip
Die Steuerung fährt von der Zuführung zu $P_1$, indem sie beide Gelenke
gleichzeitig und gleichmäßig dreht. Legen Sie dazu mit `np.linspace` je 21
Werte für $\alpha$ von $0°$ bis $9°$ und für $\beta$ von $0°$ bis $99°$ an.
Berechnen Sie für jedes Wertepaar mit `stellung` die Lage des Greifers und
speichern Sie sie in einem Array `bahn` der Form `(21, 2)`. Zeichnen Sie die
Bahn mit dem Parameter `bahn` von `zeichne_roboter` ein.

Vergleichen Sie die Länge der Bahn mit der geraden Strecke zwischen ihrem
Anfangs- und ihrem Endpunkt. Die Weglänge berechnen Sie wie in Teil 4 von
Kapitel 5.2. Warum fährt der Greifer keinen geraden Weg?
```

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung Zusatzaufgabe
:class: tip
:class: dropdown
```python
# Beide Gelenke gleichmäßig von der Grundstellung zu den Winkeln für P1 drehen
alphas = np.linspace(0, 9, 21)
betas = np.linspace(0, 99, 21)

bahn = np.zeros((21, 2))
for k in range(21):
    arm_neu, bauteil_neu = stellung(np.deg2rad(alphas[k]), np.deg2rad(betas[k]))
    bahn[k] = arm_neu[2]

# Weglänge wie in Kapitel 5.2: gerade Abstände zwischen den Bahnpunkten addieren
weg = 0.0
for k in range(len(bahn) - 1):
    differenz = bahn[k + 1] - bahn[k]
    weg += np.sqrt(differenz[0]**2 + differenz[1]**2)

# Gerade Strecke zwischen Anfangs- und Endpunkt der Bahn
differenz = bahn[-1] - bahn[0]
strecke = np.sqrt(differenz[0]**2 + differenz[1]**2)
print(f'Weg des Greifers: {weg:.3f} m')
print(f'Gerade Strecke:   {strecke:.3f} m')

arm_start, bauteil_start = stellung(0, 0)
arm_ende, bauteil_ende = stellung(np.deg2rad(9), np.deg2rad(99))
zeichne_roboter([arm_start, arm_ende], bauteile=[bauteil_start, bauteil_ende],
                ziele=bestueckplaetze[0], bahn=bahn, umgebung=[zufuehrung, platine],
                beschriftungen=['Abholen', 'Bestücken'], titel='Bahn des Greifers')
```
Ausgabe:

```text
Weg des Greifers: 0.603 m
Gerade Strecke:   0.528 m
```
Der Greifer legt $0.603\,\text{m}$ zurück, die gerade Strecke ist nur
$0.528\,\text{m}$ lang. Der Umweg beträgt also rund 14 %. Als Ziel der Strecke
nehmen wir den Endpunkt der Bahn und nicht $P_1$, denn wegen des Rasters liegt
dieser Endpunkt $3.2\,\text{mm}$ neben $P_1$. So vergleichen wir zwei Wege
zwischen denselben Punkten, und der Rasterfehler fließt nicht in den Umweg ein.
Die Steuerung dreht
die Gelenke gleichmäßig, sie führt aber nicht den Greifer geradeaus. Weil sich
$\beta$ elfmal so schnell ändert wie $\alpha$, dreht vor allem Arm 2 um
Gelenk 2, und der Greifer zieht einen Bogen um dieses Gelenk. Industrieroboter
nennen diese Fahrweise **PTP-Bewegung**, von Punkt zu Punkt. Sie ist einfach,
weil die Steuerung die Winkel nur am Start und am Ziel kennen muss. Soll der
Greifer geradeaus fahren, etwa beim Einsetzen eines Bauteils in eine Bohrung,
muss die Steuerung für viele Punkte auf der Strecke die inverse Kinematik
lösen. Wie wir einen Körper entlang einer vorgegebenen Bahn bewegen, untersuchen
wir in Part 6.
````
