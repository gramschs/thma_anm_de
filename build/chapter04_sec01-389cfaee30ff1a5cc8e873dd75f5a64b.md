---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 4.1 Wechselstromkreise als komplexe lineare Gleichungssysteme

In Kapitel 3.3 haben wir physikalische Gleichgewichtsbedingungen einer
Mehrschichtwand als reelles LGS aufgestellt und mit `np.linalg.solve` gelöst.
Das Werkzeug bleibt in diesem Kapitel dasselbe. Aber in der Wechselstromtechnik
erzwingen Kondensatoren und Spulen eine Phasenverschiebung zwischen Spannung und
Strom. *Wie lässt sich das in ein LGS einbauen?*

Die Antwort lautet: mit komplexen Impedanzoperatoren. Damit gelten Kirchhoffs
Gesetze genauso wie im Gleichstromkreis, nur mit komplexen Koeffizienten.
`np.linalg.solve` löst das System ohne jede Änderung am Aufruf.

Als Beispiel analysieren wir einen RLC-Schwingkreis aus drei Maschen. Wir
berechnen den Laststrom als Funktion der Kreisfrequenz und bestimmen die
Resonanzfrequenz numerisch.

## Lernziele

```{admonition} Lernziele
:class: attention
* [ ] Sie kennen die komplexen Impedanzoperatoren für Widerstand
  ($Z_R = R$), Kondensator ($Z_C = 1/(j\omega C)$) und Spule
  ($Z_L = j\omega L$) und können sie in Python mit `1j` berechnen.
* [ ] Sie können ein Wechselstromnetzwerk mit den Kirchhoffschen Regeln
  als komplexes LGS in der Form $\mathbf{A}\vec{x} = \vec{b}$ formulieren.
* [ ] Sie können das komplexe LGS mit `np.linalg.solve` lösen und Betrag
  sowie Phase der Lösung mit `np.abs` und `np.angle` auslesen.
* [ ] Sie können den Frequenzgang eines Schwingkreises berechnen und die
  Resonanzfrequenz numerisch mit `np.argmax` bestimmen.
```

## Komplexe Impedanzoperatoren

In der Wechselstromtechnik fasst man den ohmschen Widerstand und die
Phaseneigenschaft einer Komponente in einem komplexen **Impedanzoperator**
$Z$ zusammen. Für die drei Grundbausteine gilt:

$$Z_R = R \qquad Z_C = \frac{1}{j\omega C} \qquad Z_L = j\omega L$$

Das Ohmsche Gesetz gilt dann mit komplexen Größen unverändert:
$\underline{U} = Z \cdot \underline{I}$. Auch Kirchhoffs Regeln bleiben
gültig. Der einzige Unterschied zum Gleichstromkreis: alle Größen sind komplex.

Die imaginäre Einheit $j = \sqrt{-1}$ schreiben wir in Python als `1j`:

```{code-cell} python
import numpy as np

# Bauteilwerte des Schwingkreises
C1 = 220.e-6    # Kapazität C1 in F
C2 = 220.e-6    # Kapazität C2 in F
L  = 50.e-3     # Induktivität in H
R1 = 2.         # Reihenwiderstand in Ohm
RL = 100.       # Lastwiderstand in Ohm
U0 = 300.       # Quellspannung in V

# Kreisfrequenz für f = 50 Hz
omega = 2. * np.pi * 50.

# Komplexe Impedanzoperatoren
# 1j ist die imaginäre Einheit in Python (nicht j allein, das wäre eine Variable)
ZL  = 1j * omega * L                 # Spule:       rein imaginär, positiv
ZC1 = 1. / (1j * omega * C1)         # Kondensator: rein imaginär, negativ
ZC2 = 1. / (1j * omega * C2)         # Kondensator: rein imaginär, negativ

print(f"ZL  = {ZL:.2f} Ohm   |ZL|  = {np.abs(ZL):.4f} Ohm, Phase = {np.angle(ZL, deg=True):.1f}°")
print(f"ZC1 = {ZC1:.2f} Ohm  |ZC1| = {np.abs(ZC1):.4f} Ohm, Phase = {np.angle(ZC1, deg=True):.1f}°")
print(f"ZC2 = {ZC2:.2f} Ohm  |ZC2| = {np.abs(ZC2):.4f} Ohm, Phase = {np.angle(ZC2, deg=True):.1f}°")
```

Spule und Kondensator haben entgegengesetzte Vorzeichen im Imaginärteil.
Bei der **Resonanzfrequenz** $\omega_0 = 1/\sqrt{LC}$ heben sich die
Imaginärteile von $Z_L$ und $Z_C$ auf, was wir beim Frequenzgang beobachten
werden.

````{admonition} Mini-Übung
:class: tip
1. `np.angle(1j, deg=True)` gibt die Phase der imaginären Einheit zurück.
   Was erwarten Sie? Begründen Sie kurz, dann prüfen Sie mit Code.
2. Leiten Sie auf Papier her, bei welcher Kreisfrequenz $|Z_L| = |Z_{C1}|$
   gilt. Das ergibt $\omega_0 = 1/\sqrt{LC_1}$. Berechnen Sie dann die
   zugehörige Frequenz $f_0$ in Hz mit Python.
3. Warum ist die Phase von $Z_L$ genau $+90°$ und die von $Z_{C1}$ genau
   $-90°$? Lesen Sie die Antwort aus den Formeln ab, ohne Zahlen einzusetzen.
````

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# Frage 1: 1j liegt auf der positiven imaginären Achse -> Phase = +90°
print(np.angle(1j, deg=True))   # 90.0

# Frage 2: |ZL| = |ZC1|
# omega*L = 1/(omega*C1)  =>  omega^2 = 1/(L*C1)
# => omega_0 = 1/sqrt(L*C1)  =>  f0 = omega_0 / (2*pi)
L  = 50.e-3
C1 = 220.e-6
f0 = 1. / (2. * np.pi * np.sqrt(L * C1))
print(f"Resonanzfrequenz f0 = {f0:.2f} Hz")
```

Ausgabe:
```
90.0
Resonanzfrequenz f0 = 48.07 Hz
```

Zu Frage 3: $Z_L = j\omega L$ ist rein imaginär mit positivem Vorzeichen,
daher $\arg(Z_L) = +90°$. Es gilt $Z_C = 1/(j\omega C) = -j/(\omega C)$,
da $1/j = -j$. Der Imaginärteil ist negativ, daher $\arg(Z_C) = -90°$.
````

## Das Schaltkreis-LGS aufstellen und lösen

Unser RLC-Netzwerk besteht aus drei Maschen mit den Strömen $I_1$, $I_2$,
$I_3$. $I_3$ ist der gesuchte Laststrom. Kirchhoffs Maschenregel liefert
drei Gleichungen. Wir bringen alle Unbekannten nach links, wie in Kapitel
3.2:

$$Z_{C1}\, I_1 + (R_1 + Z_L)\, I_2 = U_0 \qquad (1)$$

$$-(R_1 + Z_L)\, I_2 + (Z_{C2} + R_L)\, I_3 = 0 \qquad (2)$$

$$I_1 - I_2 - I_3 = 0 \qquad (3)$$

Der Lösungsvektor ist $\vec{x} = (I_1,\, I_2,\, I_3)^T$.

```{code-cell} python
def getLaststrom(omega):
    """Berechnet den Laststrom I3 bei Kreisfrequenz omega."""

    # --- Impedanzoperatoren ---
    ZL  = 1j * omega * L
    ZC1 = 1. / (1j * omega * C1)
    ZC2 = 1. / (1j * omega * C2)

    # --- Koeffizientenmatrix ---
    # Unbekannte: x = [I1, I2, I3]
    #
    # Gl. (1): ZC1*I1 + (R1+ZL)*I2 + 0*I3      = U0
    #   => Zeile 0: [ZC1,  R1+ZL,      0]   b[0] = U0
    #
    # Gl. (2): 0*I1 - (R1+ZL)*I2 + (ZC2+RL)*I3 = 0
    #   => Zeile 1: [  0, -(R1+ZL), ZC2+RL]  b[1] = 0
    #
    # Gl. (3): I1 - I2 - I3 = 0
    #   => Zeile 2: [  1,       -1,     -1]  b[2] = 0
    A = np.array([
        [ZC1,    R1 + ZL,        0.   ],
        [0.,  -(R1 + ZL),   ZC2 + RL  ],
        [1.,       -1.,        -1.    ],
    ], dtype=complex)

    b = np.array([U0, 0., 0.], dtype=complex)

    # --- Lösen: identischer Aufruf wie in Kapitel 3 ---
    x = np.linalg.solve(A, b)
    I1, I2, I3 = x
    return I3


# Test bei f = 50 Hz
I3 = getLaststrom(omega=2. * np.pi * 50.)
print(f"Laststrom bei f = 50 Hz:")
print(f"  Betrag |I3| = {np.abs(I3):.4f} A")
print(f"  Phase phi   = {np.angle(I3, deg=True):.2f}°")
```

`np.linalg.solve` akzeptiert komplexe Koeffizientenmatrizen ohne jede
Änderung am Aufruf. Das Ergebnis $I_3$ ist eine komplexe Zahl: der Betrag
ist die Stromamplitude, die Phase die Verschiebung gegenüber $U_0$.

````{admonition} Mini-Übung
:class: tip
1. Was passiert bei `getLaststrom(omega=0.01)`? Führen Sie den Code aus.
   Was sagt das Ergebnis über das Verhalten eines Kondensators bei sehr
   niedrigen Frequenzen aus? Begründen Sie mithilfe der Formel
   $Z_C = 1/(j\omega C)$.
2. Verdoppeln Sie den Lastwiderstand auf `RL = 200` Ohm (ändern Sie die
   globale Variable vor dem Aufruf) und rufen Sie `getLaststrom` für
   $\omega = 2\pi \cdot 50$ auf. Wird $|I_3|$ größer oder kleiner?
   Begründen Sie zuerst ohne Code.
````

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# Frage 1: sehr kleine omega -> ZC1, ZC2 sehr groß -> Matrix fast singulär
# -> |I3| wird sehr klein: Kondensatoren sperren bei niedrigen Frequenzen
print(f"|I3| bei omega=0.01: {np.abs(getLaststrom(0.01)):.6f} A")
# Ergebnis nahe 0: der Kondensator C1 im Eingangskreis blockiert
# den Stromfluss fast vollständig (ZC1 = 1/(j*0.01*C1) sehr groß)

# Frage 2: größerer RL -> größerer Gesamtwiderstand im Lastkreis
# -> Kirchhoff: mehr Widerstand bei gleicher Spannung -> weniger Strom
RL = 200.
I3_neu = getLaststrom(omega=2. * np.pi * 50.)
print(f"|I3| bei RL=200 Ohm: {np.abs(I3_neu):.4f} A")
RL = 100.   # zurücksetzen
```

Bei sehr kleinem $\omega$ wird $|Z_C| = 1/(\omega C)$ sehr groß: der
Kondensator sperrt den Stromfluss fast vollständig. Bei doppeltem
Lastwiderstand fällt $|I_3|$ ab, weil der Gesamtwiderstand im Lastkreis
steigt.
````

## Frequenzgang und Resonanz

Wir rufen `getLaststrom` jetzt für ein ganzes Array von Kreisfrequenzen auf
und erhalten den vollständigen Frequenzgang.

```{code-cell} python
import matplotlib.pyplot as plt

# Kreisfrequenzen von 1 bis 600 rad/s
omega_arr = np.arange(start=1, stop=601, step=1)

# Laststrom für alle Kreisfrequenzen (List Comprehension)
I3_arr = np.array([getLaststrom(omega) for omega in omega_arr])

# --- Betrag und Phase ---
I3_betrag = np.abs(I3_arr)
I3_phase  = np.mod(np.angle(I3_arr, deg=True), 360.)   # Phase in (0°, 360°)

# --- Resonanzfrequenz: Index des Maximums ---
idx_res   = np.argmax(I3_betrag)
omega_res = omega_arr[idx_res]
f_res     = omega_res / (2. * np.pi)

print(f"Resonanz bei omega = {omega_res:.1f} rad/s  (f = {f_res:.2f} Hz)")
print(f"  Laststrom |I3|_max = {I3_betrag[idx_res]:.4f} A")
print(f"  Phase              = {I3_phase[idx_res]:.2f}°")

# --- Plot ---
fig, ax = plt.subplots(nrows=2, figsize=(7, 6))

ax[0].plot(omega_arr, I3_betrag, color='tab:blue', label=r'$|I_3|$ in A')
ax[0].axvline(omega_res, color='tab:red', linestyle='--',
              label=fr'Resonanz $\omega$ = {omega_res:.0f} rad/s')
ax[0].set_xlabel(r'$\omega$ in rad/s')
ax[0].set_ylabel(r'$|I_3|$ in A')
ax[0].legend()
ax[0].grid(True)

ax[1].plot(omega_arr, I3_phase, color='tab:blue', label=r'Phase $\varphi$ in °')
ax[1].axvline(omega_res, color='tab:red', linestyle='--')
ax[1].set_xlabel(r'$\omega$ in rad/s')
ax[1].set_ylabel(r'$\varphi$ in °')
ax[1].legend()
ax[1].grid(True)

plt.tight_layout()
plt.show()
```

````{admonition} Mini-Übung
:class: tip
1. Lesen Sie die Resonanzfrequenz $f_\text{res}$ aus dem Plot ab und
   vergleichen Sie sie mit dem analytischen Wert $f_0 = 1/(2\pi\sqrt{LC_1})$
   aus der ersten Mini-Übung. Stimmen die Werte überein?
2. Setzen Sie `C1 = C2 = 100e-6` F und berechnen Sie den Frequenzgang neu.
   Verschiebt sich die Resonanzfrequenz nach oben oder nach unten?
   Begründen Sie zuerst mit der Formel $\omega_0 = 1/\sqrt{LC_1}$.
3. Bei der Resonanzfrequenz beträgt die Phase im Beispiel annähernd 0°.
   Was bedeutet das für das Verhältnis von Spannung und Strom am Lastwiderstand?
````

```{code-cell} python
# Hier Ihren Code eingeben
```

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np
import matplotlib.pyplot as plt

# Frage 1: analytisch vs. numerisch
f0_analytisch  = 1. / (2. * np.pi * np.sqrt(L * C1))
print(f"Analytisch:  f0    = {f0_analytisch:.2f} Hz")
print(f"Numerisch:   f_res = {f_res:.2f} Hz")
# Beide stimmen auf ~0.5 Hz überein (Diskretisierungsfehler des omega-Arrays)

# Frage 2: kleinere Kapazität -> höhere Resonanzfrequenz
C1 = 100.e-6
C2 = 100.e-6
f0_neu = 1. / (2. * np.pi * np.sqrt(L * C1))
print(f"Neue Resonanzfrequenz: f0 = {f0_neu:.2f} Hz")

omega_arr_neu = np.arange(1, 1001, 1)
I3_neu = np.abs(np.array([getLaststrom(w) for w in omega_arr_neu]))
print(f"Numerisch (neu): f_res = {omega_arr_neu[np.argmax(I3_neu)] / (2.*np.pi):.2f} Hz")

C1 = 220.e-6   # zurücksetzen
C2 = 220.e-6
```

Kleinere Kapazität vergrößert $\omega_0 = 1/\sqrt{LC_1}$, die Resonanz
verschiebt sich zu höheren Frequenzen. Phase $\approx 0°$ bei der Resonanz
bedeutet: Spannung und Laststrom sind in Phase, der Schwingkreis verhält sich
wie ein rein ohmscher Widerstand.
````

## Zusammenfassung und Ausblick

Wechselstromnetzwerke führen auf LGS mit komplexen Koeffizienten. Die
Impedanzoperatoren $Z_R = R$, $Z_L = j\omega L$ und $Z_C = 1/(j\omega C)$
ersetzen die reellen Widerstände. Kirchhoffs Regeln liefern das LGS,
`np.linalg.solve` löst es ohne jede Änderung am Aufruf. Betrag und Phase
der komplexen Lösung lesen wir mit `np.abs` und `np.angle` aus.

Im nächsten Kapitel erweitern wir das Prinzip auf ein Netzwerk mit fünf
Maschen und analysieren einen **Bandpassfilter**, der nur einen bestimmten
Frequenzbereich durchlässt. Das Ergebnis stellen wir als Bode-Diagramm dar.
