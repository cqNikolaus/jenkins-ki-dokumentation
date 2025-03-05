# Einfache Fehleranalyse in Jenkins mit der OpenAI-API
Dieses Projekt extrahiert die Logzeilen eines fehlgeschlagenen Jenkins-Builds, sendet sie zur Analyse an die OpenAI-API und gibt die Antwort als potenzielle Fehlerursache mit einer empfohlenen Lösung aus.

## Inhaltsverzeichnis
- [Projektübersicht](#projektübersicht)
- [Architektur & Komponenten](#architektur--komponenten)
- [Job: Log-Analyse](#job-1-log-analyse)

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
