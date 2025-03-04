# Buildvorhersage
**Grundfunktion des Jobs:**   

Das Skript lädt ein trainiertes Machine-Learning-Modell (`model.pkl`), verarbeitet eine übergebene CSV-Datei mit Build-Daten und berechnet daraus die durchschnittliche Erfolgswahrscheinlichkeit des nächsten Pipeline-Builds. Fehlende Spalten werden ergänzt, um sicherzustellen, dass die Modell-Pipeline die Eingaben korrekt verarbeitet. Das Ergebnis wird als Prozentwert in der Konsole ausgegeben.

**Wichtig:** Das Modell muss mit der Python-Bibliothek `scikit-learn` trainiert worden sein, damit es korrekt geladen werden kann.


**Bestandteile:**
- `Dockerfile`
- `Jenkinsfile`
- `extract_data.py`


## Dockerfile
➡ [Link zum Dockerfile](https://github.com/cqNikolaus/JenkinsML/blob/main/prediction/Dockerfile)  
### Ablauf
Es wird ein Standard-Python-Image verwendet. Anschließend werden die Python-Abhängigkeiten `pandas` und `scikit-learn` installiert, um die eingelesenenen Dateien (CSV-Datei und PLK-Modell) verarbeiten zu können

## Jenkinsfile
➡ [Link zum Jenkinsfile](https://github.com/cqNikolaus/JenkinsML/blob/main/prediction/Jenkinsfile) 
### Ablauf
**1. Der Job nimmt zwei Pflichtparameter entgegen:**
- **model.plk**: Das (mit scikit-learn) trainierte Modell.
- **INPUT_DATA**: CSV-Datei mit den Build-Daten eines Jobs, für dessen nächsten Durchlauf eine Vorhersage getroffen werden soll.

**2. Das Docker-Image wird gebaut**

**3. Die hochgeladenen Dateien werden im Workspace gespeichert.**

**4. Der Container wird erstellt und die Dateien hineinkopiert. Anschließend wird das Skript mit der CSV-Datei und dem trainierten Modell gestartet. Nach Abschluss wird die prognostizierte Wahrscheinlichkeit des nächsten Builds in der Konsole angezeigt.**


## train_model.py
➡ [Link zum Skript](https://github.com/cqNikolaus/JenkinsML/blob/main/prediction/predict.py)

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



