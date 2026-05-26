---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 9.2 Übungen zur Trapezregel

````{admonition} Übung 9.1 (✩)
:class: tip
Gegeben ist der folgende Code, der den Bremsweg mit einer Variante der
Trapezregel berechnet.

```python
import numpy as np

def trapez_paarweise(t, v):
    n   = len(v)
    dt  = t[1] - t[0]
    summe = 0.0
    for i in range(0, n - 1, 2):
        summe += 0.5 * (v[i] + v[i + 1]) * dt
    return summe

t_a = np.array([0.0, 1.0, 2.0, 3.0, 4.0])
v_a = np.array([20.0, 16.0, 12.0, 8.0, 4.0])

t_b = np.array([0.0, 1.0, 2.0, 3.0, 4.0, 5.0])
v_b = np.array([20.0, 16.0, 12.0, 8.0, 4.0, 0.0])

print(f"Ergebnis a: {trapez_paarweise(t_a, v_a):.1f} m")
print(f"Ergebnis b: {trapez_paarweise(t_b, v_b):.1f} m")
```

1. Über welche Indizes läuft `range(0, n-1, 2)` für `n=5`?
   Welche Intervalle werden verwendet, welches wird übersprungen?
   Beantworten Sie die Frage ohne Code.
2. Berechnen Sie `Ergebnis a` von Hand. Zeigen Sie jede Trapezfläche einzeln.
3. Für `n=6` läuft `range(0, n-1, 2)` über andere Indizes. Welche?
   Was ändert sich gegenüber `n=5`, und welchen Wert erwarten Sie für
   `Ergebnis b`?
4. Vergleichen Sie beide Ergebnisse mit `np.trapezoid`. Was stellen Sie
   fest? Was schlussfolgern Sie über `trapez_paarweise` als
   Integrationsmethode?
5. Führen Sie den Code aus und überprüfen Sie Ihre Vorhersagen.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip, dropdown
```python
import numpy as np

def trapez_paarweise(t, v):
    n     = len(v)
    dt    = t[1] - t[0]
    summe = 0.0
    for i in range(0, n - 1, 2):
        summe += 0.5 * (v[i] + v[i + 1]) * dt
    return summe

t_a = np.array([0.0, 1.0, 2.0, 3.0, 4.0])
v_a = np.array([20.0, 16.0, 12.0, 8.0, 4.0])

t_b = np.array([0.0, 1.0, 2.0, 3.0, 4.0, 5.0])
v_b = np.array([20.0, 16.0, 12.0, 8.0, 4.0, 0.0])

print(f"Ergebnis a:     {trapez_paarweise(t_a, v_a):.1f} m")
print(f"Ergebnis b:     {trapez_paarweise(t_b, v_b):.1f} m")
print()
print(f"np.trapezoid a: {np.trapezoid(v_a, t_a):.1f} m")
print(f"np.trapezoid b: {np.trapezoid(v_b, t_b):.1f} m")
```

Erwartete Ausgabe:
```
Ergebnis a:     28.0 m
Ergebnis b:     30.0 m

np.trapezoid a: 48.0 m
np.trapezoid b: 50.0 m
```

Zu Frage 1: Für `n=5` gilt `n-1=4`, also `range(0, 4, 2)` = [0, 2].
Die Schleife bildet zwei Trapeze: Intervall [0, 1] und Intervall [2, 3].
Das letzte Intervall [3, 4] wird übersprungen, weil Index 3 nicht in
`range(0, 4, 2)` liegt.

Zu Frage 2: Handrechnung für `t_a`, `v_a`:
- Trapez [0, 1]: $(20 + 16) / 2 \cdot 1{,}0 = 18{,}0$ m
- Trapez [2, 3]: $(12 +  8) / 2 \cdot 1{,}0 = 10{,}0$ m
- Gesamtergebnis: $18{,}0 + 10{,}0 = 28{,}0$ m

Zu Frage 3: Für `n=6` gilt `n-1=5`, also `range(0, 5, 2)` = [0, 2, 4].
Es entstehen drei Trapeze: [0,1], [2,3] und [4,5]. Das letzte Intervall
wird jetzt einbezogen:
- Trapez [4, 5]: $(4 + 0) / 2 \cdot 1{,}0 = 2{,}0$ m
- Gesamtergebnis: $18{,}0 + 10{,}0 + 2{,}0 = 30{,}0$ m

Zu Frage 4: `np.trapezoid` liefert 48.0 m und 50.0 m, also deutlich mehr.
`trapez_paarweise` überspringt bei ungerader Punktanzahl das letzte
Intervall und bei gerader Punktanzahl alle Intervalle mit ungeradem
Startindex. Das Ergebnis hängt damit nicht nur von den Daten, sondern auch
von der Parität von $n$ ab. `trapez_paarweise` ist keine korrekte
Implementierung der Trapezregel.
````

````{admonition} Übung 9.2 (✩✩)
:class: tip
Ein Fahrzeug führt eine Notbremsung aus einer Anfangsgeschwindigkeit von
100 km/h durch. Der Fahrzeugsensor misst die Geschwindigkeit in Abständen
von 0.5 s. Das Bremsmodell folgt einem exponentiellen Abfall:

$$v(t) = v_0 \cdot e^{-k t},$$

mit $v_0 = 100\,\text{km/h}$ und $k = 0{,}35\,\text{s}^{-1}$.

Der folgende Code erzeugt die Messdaten mit realistischem Sensorrauschen:

```python
import numpy as np

np.random.seed(12)

v0 = 100 / 3.6        # Anfangsgeschwindigkeit in m/s
k  = 0.35             # Abklingkonstante in 1/s
T  = 8.0              # Messdauer in s
dt = 0.5              # Abtastrate des Sensors in s

t_mess  = np.arange(0, T + dt, dt)
v_exakt = v0 * np.exp(-k * t_mess)
rauschen = np.random.normal(0, 0.3, size=len(t_mess))  # Sensorrauschen ±0.3 m/s
v_mess   = np.maximum(v_exakt + rauschen, 0.0)         # Geschwindigkeit >= 0
```

1. Berechnen Sie den Bremsweg mit `np.trapezoid`. Geben Sie das Ergebnis
   in Metern aus.
2. Berechnen Sie den analytischen Bremsweg
   $s_\text{exakt} = \int_0^T v_0\,e^{-k t}\,\mathrm{d}t = \frac{v_0}{k}(1 - e^{-kT})$
   und vergleichen Sie mit dem numerischen Ergebnis. Wie groß ist der
   relative Fehler in Prozent?
3. Stellen Sie `v_exakt` und `v_mess` im selben Plot dar. Beschriften Sie
   Achsen mit Einheiten und fügen Sie eine Legende ein.
4. Warum ist der relative Fehler in Teilaufgabe 2 so klein, obwohl das
   Signal verrauscht ist? Formulieren Sie eine Erklärung in zwei Sätzen,
   ohne Code.
````

```{code-cell} python
# Code-Zelle
```

````{admonition} Lösung
:class: tip, dropdown
```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.style as style
style.use('seaborn-v0_8')

np.random.seed(12)

# --- Eingabe: Bremsmodell und Messdaten ---
v0 = 100 / 3.6        # Anfangsgeschwindigkeit in m/s
k  = 0.35             # Abklingkonstante in 1/s
T  = 8.0              # Messdauer in s
dt = 0.5              # Abtastrate in s

t_mess   = np.arange(0, T + dt, dt)
v_exakt  = v0 * np.exp(-k * t_mess)
rauschen = np.random.normal(0, 0.3, size=len(t_mess))
v_mess   = np.maximum(v_exakt + rauschen, 0.0)

# --- Teilaufgabe 1: numerischer Bremsweg ---
bremsweg_trapez = np.trapezoid(v_mess, t_mess)
print(f"Bremsweg (Trapezregel): {bremsweg_trapez:.2f} m")

# --- Teilaufgabe 2: analytischer Bremsweg und relativer Fehler ---
# Stammfunktion von v0*exp(-k*t): -v0/k * exp(-k*t)
# Auswertung von 0 bis T ergibt v0/k * (1 - exp(-k*T))
bremsweg_exakt  = (v0 / k) * (1 - np.exp(-k * T))
relativer_fehler = abs(bremsweg_trapez - bremsweg_exakt) / bremsweg_exakt * 100

print(f"Bremsweg (analytisch):  {bremsweg_exakt:.2f} m")
print(f"Relativer Fehler:       {relativer_fehler:.3f} %")

# --- Teilaufgabe 3: Plot ---
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(t_mess, v_exakt, color='#005A94', linewidth=2,
        label='Modell $v(t) = v_0\,e^{-kt}$')
ax.plot(t_mess, v_mess,  color='#E60000', linewidth=1, linestyle='--',
        marker='o', markersize=4, label='Sensormessung (mit Rauschen)')
ax.set_xlabel('Zeit in s')
ax.set_ylabel('Geschwindigkeit in m/s')
ax.set_title('Notbremsung: Geschwindigkeitsverlauf')
ax.legend()
ax.grid(True)
plt.tight_layout()
plt.show()
```

Erwartete Ausgabe:
```
Bremsweg (Trapezregel): 74.84 m
Bremsweg (analytisch):  74.54 m
Relativer Fehler:       0.405 %
```

Der numerische Bremsweg beträgt knapp 75 m bei einer Notbremsung aus
100 km/h. Das entspricht typischen Werten auf trockener Straße mit ABS.

Zu Frage 4: Das Sensorrauschen ist normalverteilt mit Erwartungswert null.
Über 17 Messpunkte heben sich positive und negative Abweichungen weitgehend
auf, sodass die Summe aller Trapezflächen kaum verfälscht wird. Numerische
Integration ist deshalb deutlich robuster gegenüber Messrauschen als
numerische Differentiation, bei der bereits ein einzelner Ausreißer den
lokalen Gradienten stark verfälscht.
````
