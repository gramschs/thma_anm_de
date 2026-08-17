---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# 2.1 NumPy-Grundlagen

In der Messtechnik fallen schnell tausende von Messwerten an. Ein
Beschleunigungssensor, der eine vibrierende Maschine überwacht, liefert
beispielsweise 10.000 Messwerte pro Sekunde. Wollen wir diese Daten mit
Python-Listen verarbeiten, brauchen wir Schleifen über tausende von Elementen:
mühsam zu schreiben und langsam in der Ausführung. In diesem Kapitel lernen wir
NumPy kennen, eine Bibliothek, die genau für solche Aufgaben gebaut wurde. Ihr
zentraler Datentyp, das **Array**, erlaubt es, mathematische Operationen und
statistische Kenngrößen direkt auf ganze Zahlenreihen anzuwenden, ohne eine
einzige Schleife zu schreiben.

Als roten Faden verwenden wir das Schwingungssignal einer Maschine, das wir in
diesem Kapitel erzeugen und statistisch beschreiben. In Kapitel 2.2
visualisieren wir dasselbe Signal dann mit Plotly Express.
