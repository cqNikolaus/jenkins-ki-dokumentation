# Build-Prognose in Jenkins mit Machine-Learning-Modellen

Dieses Projekt nutzt drei separate Jenkins-Jobs, um Daten aus Builds zu extrahieren, ein ML-Modell zu trainieren und mit einem trainierten Modell die Erfolgswahrscheinlichkeit des nächsten Builds eines beliebigen Jobs vorherzusagen. 

## Inhaltsverzeichnis
- [Projektübersicht](#projektübersicht)
- [Architektur & Komponenten](#architektur--komponenten)
- [Jobs](#jobs)
  - [Job 1: Datenextraktion](#job-1-datenextraktion)
  - [Job 2: Modelltraining](#job-2-modelltraining)
  - [Job 3: Build-Prognose](#job-3-build-prognose)
- [Hinweise](#hinweise)


# Projektübersicht

Das Projekt zeigt, wie Machine Learning in Jenkins eingesetzt werden kann, um die Erfolgswahrscheinlichkeit des nächsten Durchlaufs eines Jenkins-Jobs vorherzusagen. Ziel ist es, Entwicklern zu helfen, Probleme frühzeitig zu erkennen und Build-Prozesse zu verbessern. Dafür gibt es drei separate Jenkins-Jobs: 

- Der erste Job sammelt historische Build-Daten (z. B. Dauer, Erfolg etc.) eines Jobs und speichert sie in einer CSV-Datei.
- Der zweite Job trainiert ein Machine-Learning-Modell mit den Daten aus einer CSV-Datei und gibt das fertige Modell als .pkl-Datei aus.
- Der dritte Job nutzt ein trainiertes Modell und eine passende CSV-Datei, um die Erfolgswahrscheinlichkeit des nächsten Builds eines beliebigen Jobs vorherzusagen.

Die Jobs sind unabhängig nutzbar, können aber zusammen einen Ablauf bilden – etwa indem man Daten extrahiert, ein Modell trainiert und dann eine Vorhersage trifft. So wird klar, wie Machine Learning Jenkins smarter machen kann.


# Architektur & Komponenten

![Flowchart](https://github.com/cqNikolaus/jenkins-ki-dokumentation/blob/main/build-prognose-ml/jenkins-ml-diagramm.png)
