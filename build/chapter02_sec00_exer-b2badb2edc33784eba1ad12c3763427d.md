---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# Aufgabe: Prüfstand für eine Windkraftanlage

````{admonition} Aufgabe: Prüfstand für eine Windkraftanlage
:class: tip
Ein Testlauf misst die Windgeschwindigkeit an 8 Zeitpunkten während einer
70-sekündigen Anlaufphase (in m/s):

```
3.2, 5.1, 6.8, 7.5, 6.2, 4.9, 5.5, 6.0
```

**(a)** Legen Sie die Werte als Array `windgeschwindigkeit` an. Legen Sie
außerdem eine Zeitachse `zeit` mit 8 gleichmäßig verteilten Werten zwischen
0 und 70 s an. Geben Sie Form und Datentyp beider Arrays aus.

**(b)** Die Rotorleistung lässt sich vereinfacht mit $P = k \cdot v^3$
berechnen, mit $k = 1.2$. Berechnen Sie `rotorleistung` in Watt aus
`windgeschwindigkeit`.

**(c)** Der Generator braucht eine Anlaufzeit, um seinen vollen
Wirkungsgrad zu erreichen. Der Wirkungsgrad zum Zeitpunkt $t$ folgt
näherungsweise

$$\eta(t) = \eta_{max} \cdot \left(1 - e^{-t/\tau}\right)$$

mit $\eta_{max} = 0.95$ und $\tau = 20\,\text{s}$. Berechnen Sie
`wirkungsgrad` für die Zeitpunkte aus `zeit`.

**(d)** Berechnen Sie die tatsächlich abgegebene elektrische Leistung
`elektrische_leistung` aus `rotorleistung` und `wirkungsgrad`.

**(e)** Charakterisieren Sie den Testlauf: mittlere, minimale und maximale
abgegebene Leistung sowie die Streuung um den Mittelwert.

**(f)** Ein zweiter Testlauf an einem windreicheren Standort liefert an
denselben Zeitpunkten folgende Windgeschwindigkeiten (m/s):

```
7.5, 8.1, 6.9, 9.2, 8.8, 7.6, 8.4, 9.0
```

Führen Sie die komplette Berechnung aus (a) bis (e) für diesen zweiten
Standort eigenständig durch. Welcher Standort liefert im Mittel mehr
Leistung? Welcher liefert die gleichmäßigere Leistung?
````

````{code-cell} python
# Code-Zelle
````

````{admonition} Lösung
:class: tip
:class: dropdown
```python
import numpy as np

# --- (a) Windgeschwindigkeit und Zeitachse ---
windgeschwindigkeit = np.array([3.2, 5.1, 6.8, 7.5, 6.2, 4.9, 5.5, 6.0])
zeit = np.linspace(0, 70, 8)

print(windgeschwindigkeit.shape, windgeschwindigkeit.dtype)
print(zeit.shape, zeit.dtype)

# --- (b) Rotorleistung ---
k = 1.2
rotorleistung = k * windgeschwindigkeit**3
print(rotorleistung)

# --- (c) Wirkungsgrad ---
eta_max = 0.95
tau = 20.0
wirkungsgrad = eta_max * (1 - np.exp(-zeit / tau))
print(wirkungsgrad)

# --- (d) Elektrische Leistung ---
elektrische_leistung = rotorleistung * wirkungsgrad
print(elektrische_leistung)

# --- (e) Kenngrößen ---
mittlere_leistung = np.mean(elektrische_leistung)
min_leistung = np.min(elektrische_leistung)
max_leistung = np.max(elektrische_leistung)
streuung = np.std(elektrische_leistung)

print(f"Mittelwert: {mittlere_leistung:.1f} W")
print(f"Minimum:    {min_leistung:.1f} W")
print(f"Maximum:    {max_leistung:.1f} W")
print(f"Streuung:   {streuung:.1f} W")

# --- (f) Zweiter Standort ---
windgeschwindigkeit_2 = np.array([7.5, 8.1, 6.9, 9.2, 8.8, 7.6, 8.4, 9.0])

rotorleistung_2 = k * windgeschwindigkeit_2**3
wirkungsgrad_2 = eta_max * (1 - np.exp(-zeit / tau))
elektrische_leistung_2 = rotorleistung_2 * wirkungsgrad_2

mittlere_leistung_2 = np.mean(elektrische_leistung_2)
streuung_2 = np.std(elektrische_leistung_2)

print(f"Standort 2, Mittelwert: {mittlere_leistung_2:.1f} W")
print(f"Standort 2, Streuung:   {streuung_2:.1f} W")

print(f"Standort 1 liefert im Mittel: {mittlere_leistung:.1f} W")
print(f"Standort 2 liefert im Mittel: {mittlere_leistung_2:.1f} W")
```

Standort 2 liefert eine deutlich höhere mittlere Leistung, weil die
Leistung mit der dritten Potenz der Windgeschwindigkeit wächst, schon
moderat höhere Windgeschwindigkeiten schlagen also überproportional durch.
Welcher Standort die gleichmäßigere Leistung liefert, hängt vom Vergleich
der beiden Streuungswerte ab, prüfen Sie das anhand der berechneten Zahlen.
````
