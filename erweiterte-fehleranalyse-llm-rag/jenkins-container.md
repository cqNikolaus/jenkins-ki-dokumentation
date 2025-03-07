# Jenkins-Container: Erweiterte Fehleranalyse mit LLM & RAG 

Dieser Container beinhaltet die Jenkins-Instanz sowie relevante Jobs, die in unserem isolierten Setup die KI-basierte Fehleranalyse demonstrieren.

---

## Übersicht

Der **Jenkins-Container** erfüllt zwei zentrale Funktionen:

1. **Absichtlicher Fehler-Job (`error-job`)**  
   Dieser Job schlägt bewusst fehl, um den Analyseworkflow auszulösen.

2. **Fehleranalyse-Job (`llama2-log-analyzer`)**  
   Startet nach Fehlschlag automatisch und führt eine detaillierte Fehleranalyse mittels lokalem LLM und RAG (ChromaDB) durch.

---

## Jobs & Ablauf

### **error-job**

Dieser Job erzeugt absichtlich einen Fehler, um das System zu testen:

- **Was macht er genau?**
  - Versucht, eine Datei in ein Verzeichnis zu kopieren, das nicht existiert:
  ```bash
  cp target/app.jar /opt/myapp/
  ```
- **Ergebnis:** 
  ```
  Error: No such file or directory: /opt/myapp
  ```
- **On-Failure Aktion:**  
  Startet automatisch den Job `llama2-log-analyzer`.

---

## **llama2-log-analyzer Job**

Dieser Job startet die automatische KI-basierte Log-Analyse des fehlgeschlagenen Jobs.

### Ablauf:
1. Parameter erhalten vom `error-job`:
   - `FAILED_JOB_NAME`
   - `FAILED_BUILD_NUMBER`

2. Lädt das lokal verfügbare Docker-Image `analyze-log-image`.

3. Startet das Docker-Image mit den Umgebungsvariablen des error-jobs:

4. **Ergebnis:** Die vom LLM generierte KI-Analyse (inklusive Lösungsvorschläge) wird im Workspace als `analysis_report.txt` abgelegt.

---

## **Credentials & Sicherheit**

- Das Jenkins-eigene API-Token (`JENKINS_API_TOKEN`) wird sicher per `withCredentials` eingebunden:
```groovy
withCredentials([
    string(credentialsId: 'jenkins-api-token', variable: 'JENKINS_API_TOKEN')
])
```

---

## **Isolation vom Internet**

Der Container und alle Jenkins-Jobs laufen in einem isolierten Docker-Netzwerk (`llm-net`).  

- **Kein externer Traffic** → Simulation einer Offline-Umgebung  
- **Lokales Docker-Image** wird vorab erstellt, als `.tar` gespeichert und manuell in den Jenkins-Container geladen.

---

## **Jenkins Jobs (Übersicht)**

| Job | Zweck | Anmerkung |
|------|--------|------------|
| error-job | Erzeugt absichtlich einen Fehler, der zur Analyse führt | Demo des Workflows |
| llama2-log-analyzer | Führt KI-Analyse mit lokalem LLM aus | Automatisch ausgelöst durch Fehler |

Andere vorhandene Jobs (z.B. zur Jenkins-Instanzerstellung) sind nicht relevant für diese Demo.
