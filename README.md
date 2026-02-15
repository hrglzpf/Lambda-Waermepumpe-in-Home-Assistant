# Lambda Wämepumpe in Home Assistant
### Hinweis
Sämtliche Inhalte stehen zur freien Verfügung.
Sie widerspiegeln den funktionierenden Stand in meiner eigenen Installation.
Ich erhebe keinen Anspruch auf Vollständigkeit und Richtigkeit und übernehme somit auch keinerlei Verantwortung für Funktion Fehlerfreiheit.
Benutzung auf eigenes Risiko.

### Grundlegende Gedanken
Es werden hier mit der Modbus Integration und Template Helfern ausschliesslich Core Integrationen / Funktionen verwendet.
Dadurch entsteht keine Abhängigkeit von 3<sup>rd</sup> Party Integrationen und die Wahrscheinlichkeit, dass nach einem Update immer noch alles funktioniert, ist relativ hoch 😎.
## Einleitung
Die Konfiguration der Modbus Integration ist ausgelagert in eine eigene Datei modbus.yaml.
Damit das funktioniert, in der configuration.yaml `modbus: !include modbus.yaml` eintragen.
Danach werden die erzeugten Entitäten mithilfe von Templaten Helfern in einzelne "verwendbare" Entitäten zerlegt und sind somit in den Home Assistant Einstellungen auffindbar und suchbar, weil mit Zuordnung zu Kategorie, Label, Gerät, ....

## Werte per Modbus Integration abfragen
Es werden jeweils ganze, zusammengehörige Blöcke abgefragt und die abgefragten Modbus Register werden als Array im State der erzeugten Entität angelegt.
```
#
# Wärmepumpe allgemeiner Teil - kombiniert
#
- name: "WP_Allgemein_kombiniert"
  unique_id: "WP Allgemein kombiniert"
  slave: 1
  address: 0
  count: 5
  data_type: custom
  structure: ">5h"
```
Zur Formatierung des Parameters "structure":  [https://docs.python.org/3.2/library/struct.html#format-strings](https://docs.python.org/3.2/library/struct.html#format-strings)
## Helfer zum Erzeugen "verwendbarer" Entitäten 
## Sollwert und Steuer Register beschreiben
