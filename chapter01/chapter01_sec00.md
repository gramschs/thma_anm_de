---
kernelspec:
  name: python3
  display_name: 'Python 3'
---

# Exkurs: Mit Jupyter Notebooks arbeiten

In den folgenden Kapiteln schreiben und testen wir Python-Code in **Jupyter
Notebooks**. Diese Seite zeigt in wenigen Minuten, wie man ein Notebook
bedient. Wer schon mit Notebooks gearbeitet hat, kann direkt zu Kapitel 1.1
springen.

## Was ist ein Jupyter Notebook?

Ein Jupyter Notebook ist ein interaktives Dokument, das Text, Python-Code und
Ausgaben in einer Datei vereint. Notebook-Dateien erkennen wir an der Endung
`.ipynb`. Ein Notebook besteht aus einzelnen **Zellen**. Es gibt zwei Arten:

* **Code-Zellen** enthalten Python-Code. Wenn wir eine Code-Zelle ausführen,
  rechnet Python den Code und zeigt das Ergebnis direkt darunter an.
* **Textzellen** enthalten erklärenden Text. Sie werden in der Sprache
  **Markdown** geschrieben und beim Ausführen als formatierter Text angezeigt.

Dieses Skript ist selbst aus Notebooks aufgebaut: Der Fließtext steht in
Textzellen, die grau hinterlegten Codeblöcke sind ausführbare Code-Zellen.

## Eine Zelle ausführen

Wir klicken die Zelle an und drücken dann **Shift+Enter**. Alternativ gibt es
in der Werkzeugleiste einen Knopf mit einem Dreieck (`Run`). Das Ergebnis
erscheint unmittelbar unter der Zelle.

```{code-cell} python
3 * (7 - 10) + 5
```

Steht die letzte Zeile einer Code-Zelle für einen Wert, zeigt das Notebook
diesen Wert automatisch an. Möchten wir mehrere Werte oder Zwischenergebnisse
sehen, verwenden wir die Funktion `print()`.

```{code-cell} python
print(3 / 4)
print(2 ** 10)
```

Links neben einer ausgeführten Code-Zelle steht eine Zahl in eckigen Klammern,
zum Beispiel `[1]`. Das ist der **Ausführungszähler**. Er zeigt an, als
wievielte Zelle diese Zelle ausgeführt wurde.

Shift+Enter führt die Zelle aus und springt zur nächsten. Wollen wir nach dem
Ausführen in derselben Zelle bleiben, drücken wir **Strg+Enter**.

Mit dem `+`-Knopf in der Werkzeugleiste fügen wir eine neue Zelle ein. Eine
markierte Zelle löschen wir mit dem Papierkorb-Symbol oder, wenn die Zelle
blau umrandet ist, durch zweimaliges Drücken der Taste `d`.

## Die Reihenfolge zählt

Ein Notebook führt Zellen **nicht** von selbst von oben nach unten aus. Es
führt sie in der Reihenfolge aus, in der wir sie anklicken und starten. Der
Ausführungszähler `[1]`, `[2]`, `[3]` zeigt diese Reihenfolge.

Das ist wichtig, sobald eine Zelle auf einer anderen aufbaut. Im folgenden
Beispiel legt die erste Zelle die Variable `masse` an, die zweite rechnet
damit weiter.

```{code-cell} python
masse = 1200
```

```{code-cell} python
print(masse * 2)
```

Führen wir die zweite Zelle aus, ohne die erste vorher ausgeführt zu haben,
kennt Python die Variable `masse` noch nicht und meldet einen `NameError`.

Wenn der Zustand des Notebooks durcheinandergeraten ist, hilft ein Neustart:
Über das Menü `Kernel` wählen wir `Restart Kernel and Run All Cells`. Damit
vergisst Python alle Variablen und führt das ganze Notebook frisch von oben
nach unten aus.

## Wenn eine Fehlermeldung erscheint

Fehlermeldungen sind normal und gehören zum Programmieren. Die **letzte Zeile**
der Meldung nennt den Fehlertyp und eine kurze Beschreibung, zum Beispiel:

```code
NameError: name 'masse' is not defined
```

Diese Meldung sagt uns: Python kennt die Variable `masse` nicht. Meistens
liegt es an einem Tippfehler im Namen oder daran, dass die Zelle mit der
Zuweisung noch nicht ausgeführt wurde.

Damit kennen wir alles, was wir für den Einstieg brauchen. Weiter geht es mit
Kapitel 1.1.
