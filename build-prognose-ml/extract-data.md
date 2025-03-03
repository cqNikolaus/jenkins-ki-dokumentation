# Bestandteile
## Dockerfile
➡ [Link zum Dockerfile](https://github.com/cqNikolaus/JenkinsML/blob/main/data_extraction/Dockerfile)  
### Ablauf
Es wird ein Standard-Python-Image verwendet. Anschließend werden die Python-Abhängigkeiten `requests` (für API-Anfragen) und `pandas` (für die Datenaufbereitung) installiert.  

## Jenkinsfile
➡ [Link zum Jenkinsfile](https://github.com/cqNikolaus/JenkinsML/blob/main/data_extraction/Jenkinsfile) 
### Job-Ablauf
**1. Der Job nimmt zwei Pflichtparameter entgegen:**
- **TARGET_JOB_NAME**: Name des Jobs, dessen Build-Daten extrahiert werden sollen.
- **FIELDS_TO_INCLUDE**: Die Daten (Features), die extrahiert werden sollen. Der Default-Wert listet bereits alle verfügbaren Features auf. Nicht benötigte können einfach entfernt werden. Das Feld `result_bin` ist dabei nicht gelistet, es wird automatisch immer extrahiert. Dabei handelt es sich um das "Label", das als primärer Schlüssel für das Modelltraining dient. (1 = Build war erfolgreich, 0 = Build ist fehlgeschlagen). Alle anderen Features sind optional. Sollen die exportierten Daten anschließend zum Modelltraining mit dem zugehörigen Job verwendet werden, müssen mindestens die Felder `duration_sec`, `error_count` und `commits_count`
- [Liste aller Extrahierbaren Features](feature-list.md)

**2. Das Docker-Image wird gebaut**

**3. Der Container wird gestartet. Beim Start wird folgendes als Umgebungsvariablen in den Container übergeben:**

- Das Jenkins-API-Token aus den Credentials der Jenkins Instanz.
- Die Parameter `TARGET_JOB_NAME` und `FIELDS_TO_INCLUDE` 

**4. Die Standardausgabe des Skripts wird in die Datei `{TARGET_JOB_NAME}_build_data.csv` geschrieben und im Jenkins Workspace gespeichert.**
