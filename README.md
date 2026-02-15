# Lambda Wämepumpe in Home Assistant
### Hinweis
Sämtliche Inhalte stehen zur freien Verfügung.
Sie widerspiegeln den funktionierenden Stand in meiner eigenen Installation.
Ich erhebe keinen Anspruch auf Vollständigkeit und Richtigkeit und übernehme somit auch keinerlei Verantwortung für Funktion und / oder Fehlerfreiheit.  
**Verwendung auf eigenes Risiko!**  

Ein gewisses Grundverständnis für die Funktion von Home Assistant wird vorausgesetzt.  
Grundlegende Kenntnisse in der Handhabung der `configuration.yaml`,   
in der YAML Syntax https://www.home-assistant.io/docs/configuration/yaml/,   
in Templates https://www.home-assistant.io/docs/configuration/templating/ und   
Sensor Entities https://www.home-assistant.io/integrations/template/#sensor  
werden vorausgesetzt.  
Es sind dazu mehr als ausreichend Anleitungen und Hilfestellungen auf Youtube und in den diversen Foren vorhanden!

### Grundlegende Gedanken
Es werden hier mit der Modbus Integration und Template Helfern **ausschliesslich** Core Integrationen / Funktionen verwendet.  
Dadurch entsteht keine Abhängigkeit von 3<sup>rd</sup> Party Integrationen und die Wahrscheinlichkeit, dass nach einem der gefühlt unendlich vielen Updates immer noch alles funktioniert, ist relativ hoch 😎.
## Einleitung
Die Konfiguration der Modbus Integration ist ausgelagert in eine eigene Datei `modbus.yaml`.  
Damit das funktioniert, in der `configuration.yaml` die Zeile `modbus: !include modbus.yaml` eintragen.  
Danach werden die erzeugten Entitäten mithilfe von Templaten Helfern in einzelne "verwendbare" Entitäten zerlegt und sind somit in den Home Assistant Einstellungen auffindbar und suchbar, weil mit Zuordnung zu Kategorie, Label, Gerät, ....

## Werte per Modbus Integration abfragen
Aktueller Stand (2026-02-15) der Modbus Dokumentation der Fa. Lambda https://lambda-wp.at/wp-content/uploads/2025/02/Modbus-Beschreibung-und-Protokoll.pdf  

Softwarestand auf meiner Wärmepumpe:  
<img width="1722" height="1032" alt="{792CCD2A-4638-439A-9518-3EA266133796}" src="https://github.com/user-attachments/assets/e4e583c8-24bc-4957-85ec-1d5847d91a8d" />  

Vollständige Konfiguration der Modbus Integration siehe https://github.com/hrglzpf/Lambda-Waermepumpe-in-Home-Assistant/blob/main/modbus.yaml  

```yaml
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
Hinweis zur Formatierung des Parameters "structure":  [https://docs.python.org/3.2/library/struct.html#format-strings](https://docs.python.org/3.2/library/struct.html#format-strings)

Es werden jeweils ganze, zusammengehörige Blöcke abgefragt und die abgefragten Modbus Register werden als Array im State der erzeugten Entität angelegt.  
<img width="816" height="309" alt="{D8720FFB-4C35-41FB-9645-09948FB4441F}" src="https://github.com/user-attachments/assets/57c1c8d6-f5c5-4150-b205-fd1722514b1f" />

## Helfer zum Erzeugen "verwendbarer" Entitäten 
## Sollwert und Steuer Register beschreiben
