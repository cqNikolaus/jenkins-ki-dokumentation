# Einfache Fehleranalyse in Jenkins mit der OpenAI-API
Dieses Projekt extrahiert die Logzeilen eines fehlgeschlagenen Jenkins-Builds, sendet sie zur Analyse an die OpenAI-API und gibt die Antwort als potenzielle Fehlerursache mit einer empfohlenen Lösung aus.

## Inhaltsverzeichnis
- [Projektübersicht](#projektübersicht)
- [Architektur & Komponenten](#architektur--komponenten)
- [Job: Log-Analyse](#job-log-analyse)

# Projektübersicht

Dieses Projekt ermöglicht eine automatisierte Fehleranalyse in Jenkins, indem es fehlgeschlagene Builds untersucht und mit Hilfe der OpenAI-API potenzielle Fehlerursachen sowie Lösungsvorschläge liefert. Ziel ist es, Entwicklern eine schnelle, KI-gestützte Diagnose bereitzustellen, um Build-Probleme effizienter zu beheben.

Der Ablauf besteht aus zwei Hauptprozessen:

- Ein fehlgeschlagener Jenkins-Job löst automatisch eine Analyse aus.
- Der Analyse-Job holt die Logzeilen des fehlgeschlagenen Builds ab, erstellt einen Prompt und sendet diesen zur OpenAI-API.
- Die OpenAI-API verarbeitet die Anfrage und liefert eine Antwort mit möglichen Fehlerursachen und Lösungsvorschlägen zurück.
- Das Ergebnis wird als TXT-Datei im Analyse-Job ausgegeben. 

Durch diese Automatisierung entfällt die manuelle Fehlersuche, und Probleme lassen sich schneller erkennen und beheben.  


  
![Flowchart](https://github.com/cqNikolaus/jenkins-ki-dokumentation/blob/main/fehleranalyse-openai/jenkinsllm-diagramm.png)


# Architektur & Komponenten

- Die Jenkins-Instanz, wird in einem Docker Container betrieben. Dieser Container wird auf einer virtuellen Maschine (VM) in der Hetzner Cloud gehostet.
- Diese Instanz beinhaltet auch die Jobs des Projekts [Build-Prognose in Jenkins mit Machine-Learning-Modellen](https://github.com/cqNikolaus/jenkins-ki-dokumentation/blob/main/build-prognose-ml/build-prognose-ml.md), sowie Jobs zum Erstellen von Schulungsinstanzen.
  - Das Setup sollte deshalb noch einmal sauber auf einer neuen VM aufgesetzt werden.
- In der Instanz sind der Analyse Job sowie ein zugehöriger Test-Fehlerjob unter dem View "LLM Jobs" abgebildet.
- Der Analyse-Job läuft in einem separaten Docker Container auf dem Controller (built-in Node), also innerhalb des Jenkins-Containers.
- Aktuelle URL der Instanz: https://jenkins-clemens01-0.comquent.academy
- Zum Ausführen des Skripts werden ein **Jenkins-API-Token** und ein **OpenAI-API-Key** benötigt. Beide sind in den Credentials der aktuellen Jenkins Instanz hinterlegt und werden automatisch genutzt.

### Code-Repository & Struktur

Der gesamte Code des Projekts ist in einem GitHub-Repository unter https://github.com/cqNikolaus/JenkinsLLM/tree/log-analyzer organisiert.

Innerhalb dieses Repos gibt es folgende Dateien:

- Ein **Jenkinsfile**, das den Ablauf des Analysejobs beschreibt.
- Ein **Dockerfile**, das den Container für den Job definiert.
- Ein **Python-Skript**, das die eigentliche Logik implementiert.


# Job: Log-Analyse

**Grundfunktion des Jobs:**   

Das Skript verbindet sich mit einer Jenkins-Instanz, ruft die Build-Daten und Konsolen-Logs eines Jobs ab, extrahiert relevante Features (z. B. Dauer, Commits, Fehler), berechnet zusätzliche Metriken und speichert die bereinigten Daten als CSV für Machine Learning oder weitere Analysen.

**Bestandteile:**
- `Dockerfile`
- `Jenkinsfile`
- `extract_data.py`


## Dockerfile
➡ [Link zum Dockerfile](https://github.com/cqNikolaus/JenkinsLLM/blob/log-analyzer/Dockerfile)  
### Ablauf
Es wird ein Standard-Python-Image verwendet. Anschließend wird die Python-Abhängigkeit `requests` (für API-Anfragen) installiert.  

## Jenkinsfile
➡ [Link zum Jenkinsfile](https://github.com/cqNikolaus/JenkinsLLM/blob/log-analyzer/Jenkinsfile) 

### Ablauf 

**1. Ein Jobs, für den die Analyse aktiviert werden soll, wird so konfiguriert, dass er bei einem Fehlschlag automatisch den Analyse-Job startet und dabei seine Umgebungsvariablen `JOB_NAME` und `BUILD_NUMBER` als Parameter übergibt.**

**2. Der Analyse-Job nimmt zwei Pflichtparameter entgegen, die er automatisch vom fehlgeschlagenen Job erhält:**
- **FAILED_JOB_NAME**: Name des fehlgeschlagenen Jobs.
- **FAILED_BUILD_NUMBER**: Buildnummer des Fehlgeschlagenen Jobs

**3. Das Docker-Image wird gebaut**

**4. Der Container wird gestartet. Beim Start wird folgendes als Umgebungsvariablen in den Container übergeben:**

- Jenkins-API-Token und OpenAI-API-Key aus den Credentials der Jenkins Instanz.
- Die Parameter `FAILED_JOB_NAME` und `FAILED_BUILD_NUMBER`

**5. Das Skript `analyze_log.py` wird ausgeführt.**

**6. Die Standardausgabe des Skripts wird in die Datei `analysis_report.txt` geschrieben und im Jenkins Workspace des Analyse-Jobs gespeichert.**

## analyze_log.py
➡ [Link zum Skript](https://github.com/cqNikolaus/JenkinsLLM/blob/log-analyzer/analyze_log.py)

### **Ablauf**  

#### **1. Einlesen der Umgebungsvariablen & Konfiguration**  
   - Das Skript liest die benötigten **Jenkins-Verbindungsdaten** (URL, Benutzername, API-Token) sowie die **Build-Parameter** (Job-Name, Build-Nummer) aus den Umgebungsvariablen ein.  
   - Falls eine Variable fehlt, wird ein Fehler ausgegeben und das Skript abgebrochen.  

---
#### **2. Abruf des Build-Logs aus Jenkins**  
   - Die URL für das Konsolen-Log des fehlgeschlagenen Builds wird generiert.  
   - Das Log wird per **HTTP-Request mit Basic Auth** von der Jenkins-API abgerufen.  
   - Falls der Abruf fehlschlägt (z. B. falsche Authentifizierung, ungültige Build-Nummer), wird eine Fehlermeldung ausgegeben.  

---
#### **3. Extraktion relevanter Fehlermeldungen**  
   - Das abgerufene Log wird zeilenweise nach **typischen Fehlerschlagwörtern** durchsucht (`error`, `exception`, `failed`, `traceback`).  
   - Sensible Informationen wie Passwörter oder Tokens werden erkannt und durch `[REDACTED]` ersetzt.  
   - Falls keine relevanten Fehlermeldungen gefunden wurden, bricht das Skript mit einer Meldung ab.  

---
#### **4. Senden der Fehlermeldungen an die OpenAI API**  
   - Das Skript liest den **OpenAI API-Key** aus den Umgebungsvariablen.  
   - Ein **Prompt mit den extrahierten Fehlerzeilen** wird erstellt und an die OpenAI API gesendet. Der Prompt hat aktuell folgendes Muster (noch nicht optimiert):
     ```
     "Analysiere den folgenden Build-Log-Auszug.
     Identifiziere mögliche Ursachen des Fehlschlags, Fehlerquellen und mache Vorschläge zur Behebung:
     {Letzte 50 Zeilen des Jenkins Logs + Fehlerzeilen}"
   - Falls die Anfrage fehlschlägt (Netzwerkfehler, ungültiger API-Key), wird eine entsprechende Fehlermeldung ausgegeben.  

---
#### **5. Verarbeitung der API-Antwort & Ausgabe der Analyse**  
   - Die Antwort der OpenAI API wird ausgelesen und auf Fehler geprüft.  
   - Falls die API-Antwort valide ist, wird die **Fehlereinschätzung mit möglichen Lösungsvorschlägen** auf der Konsole ausgegeben.  
   - Falls die Antwort unvollständig oder fehlerhaft ist, wird eine Standardfehlermeldung ausgegeben.  

