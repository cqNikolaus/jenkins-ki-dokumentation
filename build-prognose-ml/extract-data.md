# Bestandteile
## Dockerfile
➡ [Link zum Dockerfile](https://github.com/cqNikolaus/JenkinsML/blob/main/data_extraction/Dockerfile)  
### Ablauf
Es wird ein Standard-Python-Image verwendet. Anschließend werden die Python-Abhängigkeiten `requests` (für API-Anfragen) und `pandas` (für die Datenaufbereitung) installiert.  

## Jenkinsfile
➡ [Link zum Jenkinsfile](https://github.com/cqNikolaus/JenkinsML/blob/main/data_extraction/Jenkinsfile) 
### Ablauf
Der Job nimmt zwei Pflichtparameter entgegen:
- **TARGET_JOB_NAME**: Name des Jobs, dessen Build-Daten extrahiert werden sollen.
- **FIELDS_TO_INCLUDE**: Die Daten (Features), die extrahiert werden sollen. Der Default-Wert listet bereits alle verfügbaren Features auf. Nicht benötigte können einfach entfernt werden. Das Feld `result_bin` ist dabei nicht gelistet, es wird automatisch immer extrahiert. Dabei handelt es sich um das "Label", das als primärer Schlüssel für das Modelltraining dient. (1 = Build war erfolgreich, 0 = Build ist fehlgeschlagen). Alle anderen Features sind optional. Sollen die exportierten Daten anschließend zum Modelltraining mit dem zugehörigen Job verwendet werden, müssen mindestens die Felder `duration_sec`, `error_count` und `commits_count`
- ➡ [Liste aller Extrahierbaren Features](https://github.com/cqNikolaus/JenkinsML/blob/main/data_extraction/Jenkinsfile)
