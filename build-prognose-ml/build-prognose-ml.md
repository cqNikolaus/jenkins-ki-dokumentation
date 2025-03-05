# Build-Prognose in Jenkins mit Machine-Learning-Modellen

Dieses Projekt nutzt drei separate Jenkins-Jobs, um Daten aus Builds zu extrahieren, ein ML-Modell zu trainieren und mit einem trainierten Modell die Erfolgswahrscheinlichkeit des nächsten Builds eines beliebigen Jobs vorherzusagen. 

## Inhaltsverzeichnis
- [Projektübersicht](#projektübersicht)
- [Architektur & Komponenten](#architektur--komponenten)
- [Jobs](#jobs)
  - [Job 1: Datenextraktion](#job-1-datenextraktion)
  - [Job 2: Modelltraining](#job-2-modelltraining)
  - [Job 3: Build-Prognose](#job-3-build-prognose)


# Projektübersicht

Das Projekt zeigt, wie Machine Learning in Jenkins eingesetzt werden kann, um die Erfolgswahrscheinlichkeit des nächsten Durchlaufs eines Jenkins-Jobs vorherzusagen. Ziel ist es, Entwicklern zu helfen, Probleme frühzeitig zu erkennen und Build-Prozesse zu verbessern. Dafür gibt es drei separate Jenkins-Jobs: 

- Der erste Job sammelt historische Build-Daten (z. B. Dauer, Erfolg etc.) eines Jobs und speichert sie in einer CSV-Datei.
- Der zweite Job trainiert ein Machine-Learning-Modell mit den Daten aus einer CSV-Datei und gibt das fertige Modell als .pkl-Datei aus.
- Der dritte Job nutzt ein trainiertes Modell und eine passende CSV-Datei, um die Erfolgswahrscheinlichkeit des nächsten Builds eines beliebigen Jobs vorherzusagen.

Die Jobs sind unabhängig nutzbar, können aber zusammen einen Ablauf bilden – etwa indem man Daten extrahiert, ein Modell trainiert und dann eine Vorhersage trifft.

![Flowchart](https://github.com/cqNikolaus/jenkins-ki-dokumentation/blob/main/build-prognose-ml/jenkinsml-diagramm.png)

# Architektur & Komponenten

- Die Jenkins-Instanz, wird in einem Docker Container betrieben. Dieser Container wird auf einer virtuellen Maschine (VM) in der Hetzner Cloud gehostet.
- Diese Instanz beinhaltet auch die Jobs des Projekts [Einfache Fehleranalyse mit der OpenAI-API](https://github.com/cqNikolaus/jenkins-ki-dokumentation/blob/main/fehleranalyse-openai/fehleranalyse-openai.md). Außerdem Jobs zum Erstellen von Schulungsinstanzen, sowie die 3 Jobs dieses Projekts.
  - Das Setup sollte deshalb noch einmal sauber auf einer neuen VM aufgesetzt werden.
- In der Instanz sind die drei relevanten Jobs unter dem View "Machine Learning Jobs" abgebildet.
- Alle Jobs laufen jeweils in einem separaten Docker Container auf dem Controller (built-in Node), also innerhalb des Jenkins-Containers.
- Aktuelle URL der Instanz: https://jenkins-clemens01-0.comquent.academy


### Code-Repository & Struktur

Der gesamte Code des Projekts ist in einem GitHub-Repository unter https://github.com/cqnikolaus/jenkinsML organisiert. 

Innerhalb dieses Repos gibt es für jeden der drei Jobs – Datenextraktion, Modelltraining und Vorhersage – einen eigenen Ordner. Jeder dieser Ordner enthält:

- Ein Jenkinsfile, das den Ablauf des jeweiligen Jobs beschreibt.
- Ein Dockerfile, das den Container für den jeweiligen Job definiert.
- Ein Python-Skript, das die eigentliche Logik für den jeweiligen Schritt implementiert.

# Jobs
## Job 1: Datenextraktion
Der erste Job sammelt historische Build-Daten (z. B. Dauer, Erfolg etc.) eines Jobs und speichert sie in einer CSV-Datei.  

➡️ **Ausführlicher Ablauf und Funktionsweise:** [extract-data.md](https://github.com/cqNikolaus/jenkins-ki-dokumentation/blob/main/build-prognose-ml/extract-data.md)
## Job 2: Modelltraining
Der zweite Job trainiert ein Machine-Learning-Modell mit den Daten aus einer CSV-Datei und gibt das fertige Modell als .pkl-Datei aus.  

➡️ **Ausführlicher Ablauf und Funktionsweise:** [train-model.md](https://github.com/cqNikolaus/jenkins-ki-dokumentation/blob/main/build-prognose-ml/train-model.md)
## Job 3: Build-Prognose
Der dritte Job nutzt ein trainiertes Modell und eine passende CSV-Datei, um die Erfolgswahrscheinlichkeit des nächsten Builds eines beliebigen Jobs vorherzusagen.  

➡️ **Ausführlicher Ablauf und Funktionsweise:** [predict-build.md](https://github.com/cqNikolaus/jenkins-ki-dokumentation/blob/main/build-prognose-ml/predict-build.md)


