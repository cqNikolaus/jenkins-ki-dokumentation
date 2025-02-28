# Build-Prognose in Jenkins mit Machine-Learning-Modellen

Dieses Projekt nutzt drei separate Jenkins-Jobs, um Daten aus Builds zu extrahieren (1), damit ein ML-Modell zu trainieren (2) und mit dem trainierten Modell die Erfolgswahrscheinlichkeit des nächsten Builds eines beliebigen Jobs vorherzusagen. 

## Inhaltsverzeichnis
- [Projektübersicht](#projektübersicht)
- [Architektur & Komponenten](#architektur--komponenten)
- [Jobs](#jobs)
  - [Job 1: Datenextraktion](#job-1-datenextraktion)
  - [Job 2: Modelltraining](#job-2-modelltraining)
  - [Job 3: Build-Prognose](#job-3-build-prognose)
- [Hinweise](#hinweise)


# Projektübersicht
