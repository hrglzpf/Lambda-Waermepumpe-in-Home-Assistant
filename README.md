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
<img width="600" alt="{792CCD2A-4638-439A-9518-3EA266133796}" src="https://github.com/user-attachments/assets/e4e583c8-24bc-4957-85ec-1d5847d91a8d" />  

Vollständige Konfiguration der Modbus Integration siehe https://github.com/hrglzpf/Lambda-Waermepumpe-in-Home-Assistant/blob/main/modbus.yaml    

### Beispiel für die kombinierte Abfrage des "General" Teils der Modbus Register

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
<img width="600" alt="{D8720FFB-4C35-41FB-9645-09948FB4441F}" src="https://github.com/user-attachments/assets/57c1c8d6-f5c5-4150-b205-fd1722514b1f" />

## Helfer zum Erzeugen "verwendbarer" Entitäten 
Es werden nicht alle vorhandenen Register in Entitäten abgebildet, sondern nur diejenigen die ich aktuell verwende!  

Das Array im State des "kombinierten" Sensors wird im Template mit der Split Methode zerlegt.  
Beispiel `sensor.wp_allgemein_betriebszustand`:  
```yaml
{% if (states('sensor.wp_allgemein_kombiniert').split(',')[1] | int(0)) == 0  %}
Aus 
{% elif states('sensor.wp_allgemein_kombiniert').split(',')[1] | int(0) == 1  %}
Automatik
{% elif states('sensor.wp_allgemein_kombiniert').split(',')[1] | int(0) == 2  %}
Hand
{% elif states('sensor.wp_allgemein_kombiniert').split(',')[1] | int(0) == 3  %}
Störung
{% endif %}
```
Konfigurationsdialog für o.a. Helfer:  
<img width="400" alt="{B8678F93-B57D-4505-868E-04FDA04BA336}" src="https://github.com/user-attachments/assets/94b34a67-5dde-4ed1-888f-c83474e2fd4c" />

Die Konfigurationen der Template Helfer sind als Einzeldatein in diesem Repository zu finden.  

## Werte, die nicht von der Wärmepumpe zur Verfügung gestellt werden
Die Werte für die Vorlauf-Soll Temperatur `sensor.wp_vorlauf_heizkreis_berechnet` und für die Puffer-Soll Temperatur `sensor.wp_puffer_soll_berechnet` werden nicht per Modbus zur Verfügung gestellt.  
Deshalb mit je einem Template Helfer berechnet.  
Für die Berechnung der Vorlauf-Soll Temperatur wird einfach die Heizkurve nachgebildet.  
Hier bitte mit probieren die richtigen Werte ermitteln.  
```yaml
{{36 + ((25 - (float(states('sensor.wp_allgemein_aussentemperatur_berechnet_fur_regelung'))) ) *0.40) | round(1) }}
```
Für die Puffer-Soll Temperatur wird die Vorlauf-Soll Temperatur plus der Vorlauf-Offset plus 1 °C verwendet.  
```yaml
{{
states('sensor.wp_vorlauf_heizkreis_berechnet') | float() + 
states('sensor.wp_heizkreis_offset_vorlauftemperatur_soll') | float() + 1
}}
```
## Sollwert und Steuer Register beschreiben
