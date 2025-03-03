# Bestandteile
## Dockerfile
➡ [Link zum Dockerfile](https://github.com/cqNikolaus/JenkinsML/blob/main/data_extraction/Dockerfile)  
### Ablauf
Es wird ein Standard-Python-Image verwendet. Anschließend werden die Python-Abhängigkeiten `requests` (für API-Anfragen) und `pandas` (für die Datenaufbereitung) installiert.  

## Jenkinsfile
➡ [Link zum Jenkinsfile](https://github.com/cqNikolaus/JenkinsML/blob/main/data_extraction/Jenkinsfile) 
### Ablauf
**1. Der Job nimmt zwei Pflichtparameter entgegen:**
- **TARGET_JOB_NAME**: Name des Jobs, dessen Build-Daten extrahiert werden sollen.
- **FIELDS_TO_INCLUDE**: Die Daten (Features), die extrahiert werden sollen. Der Default-Wert listet bereits alle verfügbaren Features auf. Nicht benötigte können einfach entfernt werden. Das Feld `result_bin` ist dabei nicht gelistet, es wird automatisch immer extrahiert. Dabei handelt es sich um das "Label", das als primärer Schlüssel für das Modelltraining dient. (1 = Build war erfolgreich, 0 = Build ist fehlgeschlagen). Alle anderen Features sind optional. Sollen die exportierten Daten anschließend zum Modelltraining mit dem zugehörigen Job verwendet werden, müssen mindestens die Felder `duration_sec`, `error_count` und `commits_count`
- [Liste aller Extrahierbaren Features](feature-list.md)

**2. Das Docker-Image wird gebaut**

**3. Der Container wird gestartet. Beim Start wird folgendes als Umgebungsvariablen in den Container übergeben:**

- Das Jenkins-API-Token aus den Credentials der Jenkins Instanz.
- Die Parameter `TARGET_JOB_NAME` und `FIELDS_TO_INCLUDE`

**4. Das Skript [data_extraction.py](build-prognose-ml/extract-data.md) wird ausgeführt.**

**5. Die Standardausgabe des Skripts wird in die Datei `{TARGET_JOB_NAME}_build_data.csv` geschrieben und im Jenkins Workspace gespeichert.**

## extract_data.py
➡ [Link zum Skript](https://github.com/cqNikolaus/JenkinsML/blob/main/data_extraction/extract_data.py)

### Ablauf

1. **Konfiguration der Jenkins-Verbindung**  
   - Es wird eine Klasse erstellt, die eine Verbindung zu Jenkins herstellt.  
   - Die URL der Jenkins-Instanz, der Benutzername, das API-Token und der Job-Name werden eingelesen bzw. gespeichert.  
   - Optional wird eine maximale Anzahl an Builds definiert, die abgerufen werden sollen.

2. **Generierung von URLs für API-Anfragen**  
   - URLs für Build-Daten und Konsolen-Logs werden basierend auf der Build-Nummer erzeugt.

3. **Abruf von Daten aus Jenkins**  
   - Das Skript sendet HTTP-Anfragen an Jenkins und holt die Build-Daten als JSON, sowie den Log des Builds.   

4. **Verarbeitung der Build-Daten**  
   - Das Skript liest die JSON-Daten eines Builds und extrahiert relevante Informationen.
   - Zusätzliche abgeleitete Features werden hergestellt, z.B. Verhältnis von Commit-Autoren zu Fehler-Verantwortlichen. 
   - Anzahl der Fehler (`ERROR` oder `Exception`) im Log werden gezählt und als `error_count` zur Verfügung gestellt.  

5. **Abruf mehrerer Builds**  
   - Das Skript ruft nacheinander alle Builds bis zur definierten Maximalanzahl ab.  
   - Die Build-Daten werden in einer Liste gesammelt.

6. **Verarbeitung der Daten als DataFrame**  
   - Die Build-Daten werden in eine Tabelle (`pandas.DataFrame`) umgewandelt.  
   - Daten werden bereinigt, builds mit unbekanntem Ergebnis werden herausgefiltert.  
   - Falls eine Umgebungsvariable definiert, welche Spalten exportiert werden sollen, wird das hier berücksichtigt.  
   - Eine Binärvariable (`0` oder `1`) für das Build-Ergebnis (`FAILURE` oder nicht) wird hinzugefügt.

8. **Ausgabe der Daten als CSV**  
   - Die gefilterten Daten werden als CSV ausgegeben (standardmäßig in die Konsole).  
   - Falls keine Daten vorhanden sind, wird das Skript mit einer Fehlermeldung beendet.  

Am Ende hat man also eine detaillierte Tabelle mit allen relevanten Informationen über die letzten Builds aus Jenkins. 🚀
