# Skripte & Dateien

Diese Seite beschreibt alle Skripte und relevanten Dateien, die im Projekt verwendet werden. Jede Datei wird kurz, aber klar und präzise beschrieben, um ihren Zweck, Einsatzort sowie Ablauf zu verstehen. Detaillierte Codes findest du direkt im GitHub-Repository.

---

## Übersicht

| Datei | Zweck | Zugehörigkeit |
|--------|-------|----------------|
| [`text_to_embeddings.py`](#text_to_embeddingspy) | Erzeugt Embeddings aus Textdaten | Lokal (vorbereitend) |
| [`ingestion.py`](#ingestionpy) | Lädt Embeddings in ChromaDB | ChromaDB-Container |
| [`app.py`](#apppy) | API-Endpunkt & Hauptlogik (Logs, ChromaDB, LLM) | LLM-API-Container |
| [`analyze_log.py`](#analyze_logpy) | Ruft LLM-API aus Jenkins Analyzer-Job auf | Jenkins-Analyzer-Job |
| [`Dockerfile`](#dockerfile) | Baut das Docker-Image für den LLM-API-Container | Lokal (vorbereitend) |

---

## text_to_embeddings.py

- **Wo genutzt:** Lokal (nicht auf VM oder Container)
- **Zweck:** Wandelt Rohtexte (z. B. Support-Chats) in Embeddings um, die später in ChromaDB geladen werden.
- **Eingabe:** Textdatei (`.txt`)
- **Ausgabe:** `embeddings.json` (Chunks mit Embeddings)

### Was passiert hier genau?

- Skript liest eine Textdatei (z.B. Chatlogs).
- Prüft, ob Timestamps (`[HH:MM]`) vorhanden sind:
  - Falls ja, wird anhand der Timestamps aufgeteilt.
  - Falls nein, erfolgt eine satzbasierte Aufteilung.
- Wandelt Text-Chunks mithilfe des Modells `all-MiniLM-L6-v2` in Embeddings um.
- Speichert Embeddings als strukturierte JSON-Datei (`embeddings.json`) ab.

- **Output:** `embeddings.json` (Embeddings-Daten zur späteren Ingestion in ChromaDB)

- [🔗 text_to_embeddings.py auf GitHub](https://github.com/cqNikolaus/jenkins-llama2-rag-log-analyzer/blob/main/chromadb-container/text_to_embeddings.py)

---

## ingestion.py

- **Wo genutzt:** Direkt im ChromaDB-Container
- **Zweck:** Lädt vorbereitete Embeddings (`embeddings.json`) in eine ChromaDB-Collection.

**Ablauf:**
1. Verbindung zu ChromaDB herstellen (persistente Speicherung unter `/data/chroma`).
2. Falls Collection vorhanden: Diese löschen und neu erstellen.
3. JSON-Datei (`embeddings.json`) einlesen.
4. Embeddings, Texte und Metadaten strukturiert in ChromaDB speichern.

- **Ergebnis:**  
  Embeddings sind semantisch durchsuchbar für spätere Abfragen.

- [→ ingestion.py auf GitHub](https://github.com/cqNikolaus/jenkins-llama2-rag-log-analyzer/blob/main/chromadb-container/ingestion.py)

---

## app.py

- **Wo genutzt:** LLM-API-Container
- **Zweck:**  
  Zentrales Skript des Containers, bereitgestellt per FastAPI. Stellt API-Endpunkt bereit, ruft Jenkins-Logs ab, führt semantische Suche (ChromaDB) durch und interagiert mit lokalem Llama-2-Modell.

**Genauer Ablauf:**

- Empfängt HTTP POST Anfragen von Jenkins-Analyzer-Job.
- Holt mittels Jenkins-API Logs des fehlgeschlagenen Builds.
- Extrahiert automatisch relevante Fehlerzeilen und teilt sie in Chunks auf.
- Jeder Chunk wird semantisch in ChromaDB durchsucht, um relevanten historischen Kontext zu finden.
- Erstellt daraus den Prompt für das LLM-Modell.
- Übergibt Prompt an Llama-2 und empfängt die KI-basierte Fehleranalyse inklusive Lösungsschritte.
- Sendet finale Analyse zurück an Jenkins.

- [→ app.py auf GitHub](https://github.com/cqNikolaus/jenkins-llama2-rag-log-analyzer/blob/main/llm-api-container/app.py)

---

## analyze_log.py

- **Wo genutzt:** Innerhalb des Docker-Containers, gestartet durch Jenkins-Analyzer-Job.
- **Zweck:** Ruft mit den vom Jenkins-Analyzer-Job bereitgestellten Parametern (Jobname, Build-Nummer, API-Token) den API-Endpunkt (`/analyze_log`) im LLM-Container auf.

**Funktion in Kurzform:**

- Sendet HTTP POST Anfrage an LLM-API-Endpunkt.
- Empfängt die KI-Analyse als JSON-Antwort.
- Schreibt die Analyse in eine Datei (`analysis_report.txt`), welche im Jenkins Workspace gespeichert wird.

- [→ analyze_log.py auf GitHub](https://github.com/cqNikolaus/jenkins-llama2-rag-log-analyzer/blob/main/jenkins/analyze_log.py)

---

## Dockerfile

- **Wo genutzt:** Lokal zum Erstellen des Docker-Images für den LLM-API-Container
- **Zweck:** Baut den Docker-Container, der später das Skript (`app.py`) und das Llama-2-Modell bereitstellt.

**Inhalt (Kurzform):**
- Basiert auf einem Python-Base-Image
- Installiert alle nötigen Python-Abhängigkeiten (z.B. `fastapi`, `uvicorn`, `llama_cpp`)
- Kopiert Skripte (`app.py`) und Modell in den Container
- Startet den API-Server beim Start automatisch

- [→ Dockerfile auf GitHub](https://github.com/cqNikolaus/jenkins-llama2-rag-log-analyzer/blob/main/llm-api-container/Dockerfile)

---

## Weitere erwähnenswerte Dateien

### embeddings.json
- Fertig erstellte Embeddings aus dem Demo-Support-Chat (lokal generiert, manuell auf Container übertragen).
- Wird einmalig zur Befüllung der ChromaDB verwendet.

---

## Weiterführende Links

- [Jenkins-Container Dokumentation](jenkins-container.md)
- [LLM-API-Container Dokumentation](llm-api-container.md)
- [ChromaDB-Container Dokumentation](chromadb-container.md)

[← Zurück zur Projektübersicht](erweiterte-fehleranalyse-llm-rag.md)
