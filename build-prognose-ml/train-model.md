# Modelltraining
**Grundfunktion des Jobs:**   

Das Skript lädt eine CSV-Datei mit Jenkins-Build-Daten, bereinigt und transformiert die Features, trainiert ein Machine-Learning-Modell (z. B. Random Forest, XGBoost) zur Build-Prognose und speichert das trainierte Modell als .pkl-Datei.

**Hintergrund:**
- ML-Modelle **benötigen numerische Daten** – kategorische oder textbasierte Werte müssen vor dem Training sinnvoll in Zahlen kodiert werden.  
- Daten müssen **klaren Typen zugeordnet werden** (numerisch, kategorisch, zeitbasiert etc.), sonst ist das Training fehlerhaft oder unbrauchbar.  
- **Automatische Identifikation von Spaltentypen ist fehleranfällig**, deshalb gibt man dem Modell normalerweise exakt vor, welche Daten es erwartet.  
- Ein vollständig flexibles Skript wäre unrealistisch und fehleranfällig, ein zu striktes Modell wäre unflexibel.  
- **Lösung: Hybrid-Ansatz**  
  - Das Skript kennt und verarbeitet **die üblichen Jenkins-Metriken zuverlässig**.  
  - **Unbekannte Spalten werden analysiert:** Falls sie nicht sicher zugeordnet werden können, werden sie ignoriert.  
- **Wahl der Python-ML-Bibliothek:** Scikit-Learn passt besser als PyTorch, da es Spaltentypen analysieren und automatisch transformieren kann, während PyTorch fertige kodierte Daten erwartet.

**Bestandteile:**
- `Dockerfile`
- `Jenkinsfile`
- `requirements.txt`
- `extract_data.py`


## Dockerfile
➡ [Link zum Dockerfile](https://github.com/cqNikolaus/JenkinsML/blob/main/training/Dockerfile)  
### Ablauf
Es wird ein Standard-Python-Image verwendet. Anschließend werden die Python-Abhängigkeiten aus der `requirements.txt` installiert.  

## Jenkinsfile
➡ [Link zum Jenkinsfile](https://github.com/cqNikolaus/JenkinsML/blob/main/training/Jenkinsfile) 
### Ablauf
**1. Der Job nimmt zwei Pflichtparameter entgegen:**
- **CSV_FILE**: Die CSV-Datei mit der das Modell trainiert werden soll. Diese muss nicht zwingend mit dem [Data-Extraction-Job](https://github.com/cqNikolaus/jenkins-ki-dokumentation/blob/main/build-prognose-ml/extract-data.md) erzeugt worden sein, es werden allerdings mindestens folgende 4 Pflichtspalten benötigt: `result_bin`, `duration_sec`, `error_count` und `commits_count` 
- **MODEL_NAME**: Es kann eines von 4 ML-Modellen gewählt werden: _random_forest, gradient_boosting, logistic_regression, xgboost._

**2. Das Docker-Image wird gebaut**

**3. Die hochgeladene CSV-Datei wird im Workspace gespeichert.**

**4. Der Container wird erstellt und die CSV-Datei hineinkopiert. Anschließend wird das Skript mit der CSV-Datei und dem gewählten Modell gestartet. Nach Abschluss wird das trainierte Modell (model.pkl) in den Jenkins-Workspace kopiert.**

## requirements.txt
Beinhaltet die zu installierenden Abhängigkeiten:  

**numpy** → Numerische Berechnungen, z. B. zyklische Zeittransformation.   
**pandas** → Laden und Verarbeiten der CSV-Daten.  
**scikit-learn** → ML-Modelle, Training und Feature-Transformationen.  
**joblib** → Speichern und Laden des trainierten Modells.  
**argparse** → Einlesen von Kommandozeilenparametern.  



## train_model.py
➡ [Link zum Skript](https://github.com/cqNikolaus/JenkinsML/blob/main/training/train_model.py)

### Ablauf

#### 1. Pflichtspalten prüfen:  
Es wird geprüft, ob die vier Pflichtspalten `result_bin`, `duration_sec`, `error_count` und `commits_count` vorhanden sind. Andernfalls bricht das Skript mit einer Fehlermeldung ab.

---
#### 2.Bekannte Features:  
Anhand vordefinierter Listen werden numerische, kategoriale und zeitbasierte (Time‑Based) Features erkannt.  

– _Numerisch:_ z. B. `duration_sec`, `commits_count`, `error_count` usw.  
– _Kategorial:_ z. B. `built_on`, `change_set_kind`, `executor_name`, …  
– _Zeitbasiert:_ z. B. `build_date`, `build_time`, `build_hour`  

Zusätzlich werden unwichtige Spalten (wie `build_number`, `build_url`, `parameters`) entfernt.

---
#### 3. Unbekannte Spalten analysieren:  
Alle übrigen Spalten werden geprüft:

- **Numerische Spalten:** Es wird die Pearson-Korrelation zur Zielvariablen `result_bin` berechnet. Nur wenn der Betrag der Korrelation mindestens 0.1 beträgt, wird die Spalte in die Analyse übernommen.
 - **Kategoriale Spalten:** Enthält die Spalte wenige unterschiedliche Werte (hier ≤ 30), wird mittels Chi‑Quadrat-Test (p‑Wert < 0.05) geprüft, ob ein signifikanter Zusammenhang zur Zielvariablen besteht. Ist dies der Fall, wird die Spalte aufgenommen.  

Spalten, die als Freitext (viele unterschiedliche Werte) erkannt werden oder die den Kriterien nicht genügen, werden ignoriert.

---
#### 4. Feature‑Engineering für Zeitangaben:  
Ein eigener Transformer (`TimeFeaturesExtractor`) zerlegt:

`build_date`: In Jahr, Monat, Tag und Wochentag
`build_time` und `build_hour`: Es erfolgt eine zyklische Transformation (sin/cos‑Kodierung, basierend auf 24 Stunden).

---
#### 5. Pipeline-Aufbau:  
Zunächst wird der Zeit‑Transformer als Vorverarbeitungsschritt angewendet, sodass die neuen (numerischen) Zeitfeatures zur Verfügung stehen. Anschließend kommen ein numerischer Pipeline-Teil (Imputation + Skalierung) und ein kategorialer Pipeline-Teil (Imputation + One-Hot-Encoding) zum Einsatz.  

Das gewünschte Modell (z. B. RandomForest, Gradient Boosting, Logistic Regression oder XGBoost) wird als finaler Klassifikator in die Pipeline eingebunden.

---
#### 6.Training & Speicherung:  

Das Modell wird mit den gefilterten Features trainiert und mittels `joblib.dump` in der angegebenen Zieldatei (z. B. `model.pkl`) gespeichert.


