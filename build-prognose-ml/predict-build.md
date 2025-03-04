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


## predict.py
➡ [Link zum Skript](https://github.com/cqNikolaus/JenkinsML/blob/main/prediction/predict.py)

### Ablauf

#### **1. Modell & Eingabedaten laden:**  
Das trainierte Machine-Learning-Modell (`model.pkl`) wird mit `joblib.load()` geladen. Anschließend wird die übergebene CSV-Datei mit `pandas.read_csv()` eingelesen. Falls eine der beiden Dateien fehlt oder fehlerhaft ist, bricht das Skript mit einer Fehlermeldung ab.  

---  
#### **2. Erwartete Spalten ermitteln:**  
Das Skript extrahiert aus der gespeicherten Scikit-Learn-Pipeline die erwarteten numerischen und kategorialen Features. Diese dienen als Basis für die Vorbereitung der Eingabedaten.  

---  
#### **3. Fehlende Spalten ergänzen:**  
Falls die Eingabe-CSV nicht alle erwarteten Features enthält, werden diese Spalten mit `NaN` gefüllt, damit das Modell die Daten trotzdem verarbeiten kann.  

---  
#### **4. Erfolgswahrscheinlichkeit berechnen:**  
Das Modell führt eine **Wahrscheinlichkeitsvorhersage (`predict_proba`)** durch. Der Mittelwert der Wahrscheinlichkeiten für einen erfolgreichen Build (`SUCCESS`) wird berechnet.  

---  
#### **5. Ausgabe des Ergebnisses:**  
Die berechnete Erfolgswahrscheinlichkeit wird als gerundeter Prozentwert in der Konsole ausgegeben.



