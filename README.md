# REM2030-Simulation
# README

Dieses Repository enthält eine Simulationsumgebung, die auf den REM2030-Daten basiert und untersucht, unter welchen Umständen eine Umstellung auf Elektrofahrzeuge pro Unternehmen möglich ist und welche Ladeinfrastruktur (Anzahl und Leistung von Ladepunkten) dafür benötigt wird.

---

## Übersicht

1. **Ziel**  
   - Das Modell simuliert minutengenau Fahr- und Parkverhalten von Firmenfahrzeugen und ermittelt daraus den Ladebedarf.  
   - Es wird davon ausgegangen, dass Fahrzeuge nur am Unternehmensstandort laden können, sofern sie sich dort gerade im Parkzustand befinden.

2. **Fahrzeugmodelle**  
   - Für jede Fahrzeuggröße (genaue Beschreibung im **REM2030Codebook**) sind zwei Fahrzeugkonfigurationen hinterlegt:  
     - **Basisvariante**: niedrigere Reichweite  
     - **Maximalvariante**: größere Reichweite  
   - Konkrete Beispiele:
     - **Klein**: VW ID.3  
     - **Mittel**: VW ID.5  
     - **Groß**: Mercedes EQS  
     - **Transporter**: VW e-Transporter  
     - **Sonstige (Lkw)**: Mercedes eActros 600  

3. **Aufbau der Dateien**  
   - **REM2030_v2015** und **REM2030_v2015_car_info**: Ursprungsdatensätze (Rohdaten) des REM-Projekts.  
   - **CleaningREM2030 (Code 1)** und **CleaningREM2030car (Code 2)**: Skripte zur Datenbereinigung.  
   - **REM2030_cleaned** und **REM2030_car_cleaned**: Bereinigte Datensätze, extrahieren nur die relevanten Informationen zu Fahrzeugen, Firmen und Strecken.  
   - **REM2030 Basisplots**: Erste visuelle Exploration der originalen REM2030-Daten.  
   - **ModellierungFahrdaten (Code 3)**: Erstes relevanten Notebook für die Simulation; erzeugt eine detaillierte, minutengenaue Zusammenfassung pro Fahrzeug.  
   - **Simulationsvorbereitung (Code 4)**: Optionales Notebook zum Zusammenfassen einzelner Firmen nach Wirtschaftszweigen.  
   - **Simulation (Code 5)**: Kern-Simulation des Ladebedarfs auf Basis der modellierten Fahrdaten.  
   - **Ergebnisplots final**: Erzeugt diverse Diagramme und Auswertungen aus den Simulationsergebnissen.

---

## Nutzung

1. **Download & Verzeichnisstruktur**  
   - Laden Sie alle Dateien (Notebooks und CSVs) herunter und legen Sie sie in **einem** Ordner ab.  
   - Achten Sie darauf, dass:
     - Die CSV-Dateien (**REM2030_v2015**, **REM2030_v2015_car_info**, **REM2030_cleaned**, **REM2030_car_cleaned** etc.) sich im **selben Ordner** wie die Notebooks befinden, damit Pfade korrekt funktionieren.

2. **Ablauf der Simulation**  
   1. **(Optional)** Führen Sie die Notebooks *CleaningREM2030* und *CleaningREM2030car* aus, wenn Sie die Bereinigungsschritte nachvollziehen oder anpassen wollen. (Die bereinigten Daten sind bereits vorhanden, daher **nicht zwingend nötig**.)  
   2. **ModellierungFahrdaten (Code 3)**  
      - Einstellen gewünschter Konfigurationsparameter (siehe unten).  
      - Ausführen, um eine CSV-Datei `fahrzeug_zusammenfassung` zu erzeugen (minutengenaue Fahrzeugstatusdaten).  
   3. **(Optional) Simulationsvorbereitung (Code 4)**  
      - Fasst mehrere Unternehmen pro Wirtschaftszweig zusammen und belegt eine neue Firmen-ID.  
      - Nur relevant, wenn Wirtschaftszweige und nicht die Originalfirmen miteinander verglichen werden sollen.  
      - Ergebnis ist eine Datei `fahrzeug_zusammenfassung2`.  
      - Wenn Sie **nicht** mit diesem Schritt arbeiten, müssen Sie **fahrzeug_zusammenfassung**  selbst in `fahrzeug_zusammenfassung2` umbenennen, damit der nachfolgende Code darauf zugreifen kann.  
   4. **Simulation (Code 5)**  
      - Hier findet die eigentliche Simulation des Ladebedarfs statt.  
      - Erzeugt drei Ergebnisdateien:  
        - `fahrzeuge_ergebnisse.csv`  
        - `firmen_ergebnisse.csv`  
        - `ladesaeulen_ergebnisse.csv`  
   5. **Ergebnisplots final**  
      - Erstellt umfassende Diagramme und Kennzahlen aus den Ergebnisdateien.

---

## Wichtige Einstellungen in den Notebooks

### 1. Notebook **ModellierungFahrdaten**

- **N_Weeks**: Anzahl zu simulierender Wochen (Standard: 3).  
- **VEHICLES_PER_SECTOR**: Legt fest, wie viele Fahrzeuge je Wirtschaftszweig betrachtet werden (für den Zufallszugriff bei der Fahrzeugauswahl).  
- **HEATMAP_VEHICLE_ID**: Fahrzeug-ID, zu der am Ende des Codes eine Heatmap (Fahrend, Parkend am Firmenstandort, Parkend Remote) erstellt wird.

Nach dem Ausführen entsteht die CSV `fahrzeug_zusammenfassung`, in der jedes Fahrzeug minutengenau für den betrachteten Zeitraum aufgeführt ist.

### 2. Notebook **Simulation**

- **BATTERY_VARIANT_INDEX** und **CONSUMPTION_VARIANT_INDEX** (jeweils 0 oder 1):  
  - Bestimmen, ob die Basisvariante (0) oder die Maximalvariante (1) der Fahrzeuge eingesetzt wird.  
- **battery_variants** / **consumption_variants**: Enthalten Beispielwerte für die entsprechenden Fahrzeugmodelle. Können bei Bedarf angepasst werden.  
- **WAITING_THRESHOLD** (in %):  
  - Darunter **müssen** die Fahrzeuge sofort laden (wenn sie am Firmenstandort sind. Sie warten dann aber ggf. in der Warteschlange).  
- **LOADING_THRESHOLD** (in %):  
  - Ab diesem Wert **dürfen** Fahrzeuge laden, wenn eine Ladesäule **frei** ist (und sie am Firmenstandort sind).  
  - Dient zur besseren Auslastung der Ladesäulen.  
- **MIN_SOC_ALLOWED** (in %):  
  - Untergrenze, ab der ein Fahrzeug als „nicht elektrifizierbar“ gilt, wenn auch bei Maximalkonfiguration (default: 50 Ladesäulen à 50 kW) der SoC nicht über diesen Wert gehalten werden kann. (Standard: 0)  
- **charger_powers**: Liste möglicher Ladesäulenleistungen (Standardwerte: 3.7, 11, 22, 50 kW).  
- **ENABLE_WAITING (bool)** (Standard: True):  
  - Ob Fahrzeuge bei blockierten Ladesäulen warten, bis eine Säule frei wird.  
- **TARGET_COMPANIES**:  
  - Bestimmt, welche Firmen(-IDs) am Ende des Codes genauer visualisiert werden. (Standard: IDs der zusammengefassten Wirtschaftszweige.)

Der Code erzeugt im Anschluss die drei oben genannten Ergebnisdateien.

---

## Dateibeschreibungen

- **REM2030_v2015.csv** und **REM2030_v2015_car_info.csv**:  
  - Rohdaten aus dem REM-Projekt mit Fahrzeug-, Firmen- und Fahrteninformationen.  
  - Können auch direkt über die REM2030-Webseite heruntergeladen werden (siehe Codebook-Link dort).  

- **REM2030Codebook** (nicht direkt im Repo, Link in den Originalunterlagen):  
  - Enthält detaillierte Beschreibung aller Variablen in den Rohdaten.

- **CleaningREM2030** / **CleaningREM2030car**:  
  - Bereinigen die originalen CSV-Dateien und extrahieren die wichtigsten Informationen.  
  - Bereits erledigt, aber nützlich, falls Anpassungen gewünscht sind (z.B. mehr Daten).  

- **REM2030_cleaned.csv** und **REM2030_car_cleaned.csv**:  
  - Bereinigte Datensätze mit nur den relevanten Variablen aus den Rohdaten.  

- **REM2030 Basisplots**:  
  - Enthält grundlegende Diagramme und Visualisierungen auf Basis der Ursprungsdaten.  

- **ModellierungFahrdaten (Code 3)**:  
  - Erstes Notebook, das tatsächlich für die Simulation benötigt wird.  
  - Enthält Konfigurationsmöglichkeiten, wählt pro Firma Fahrzeuge gemäß einer prozentualen Verteilung aus und generiert `fahrzeug_zusammenfassung`.  

- **Simulationsvorbereitung (Code 4)**:  
  - Optional: Fasst Firmen nach Wirtschaftszweigen zusammen.  
  - Ergebnisdatei: `fahrzeug_zusammenfassung2`.  

- **Simulation (Code 5)**:  
  - Führt die eigentliche Simulation durch und berechnet den Ladebedarf der Fahrzeuge.  
  - Erstellt:
    - `fahrzeuge_ergebnisse.csv`  
    - `firmen_ergebnisse.csv`  
    - `ladesaeulen_ergebnisse.csv`  

- **Ergebnisplots final**:  
  - Erstellt diverse Plots und Auswertungen zu den Simulationsergebnissen.  

---

## Empfehlungen

- Verwenden Sie beispielsweise **Visual Studio Code** (VSC) mit den passenden Erweiterungen für Jupyter Notebooks bzw. Python, um eine bequeme Entwicklungs- und Ausführungsumgebung zu haben.  
- Die einzelnen Code-Zellen sind übersichtlich gehalten und ausführlich kommentiert. Sie bauen *nicht* strikt aufeinander auf, sodass Anpassungen unkompliziert möglich sind.  
- Viel Spaß mit dem Modell!  

Bei Fragen oder Problemen gerne ein Issue in diesem Repository erstellen.
