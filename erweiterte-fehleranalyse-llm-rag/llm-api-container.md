# Jenkins LLM-API-Container

Diese Seite beschreibt den **LLM-API-Container**, seine Komponenten und die konkreten Schritte, mit denen der API-Endpunkt aufgebaut und genutzt wird.

---

## Übersicht und Zweck

Der LLM-API-Container ist die zentrale Komponente des Projekts. Er stellt einen lokalen HTTP-API-Endpunkt bereit, über den der Jenkins-Analyse-Job Jenkins-Logs automatisiert analysieren lassen kann. Die Analyse erfolgt mithilfe eines lokal ausgeführten LLM-Modells (**Llama-2**) und zusätzlichem Kontext aus einer semantischen Wissensdatenbank (**ChromaDB**).

Der Container arbeitet vollständig ohne Internetzugriff.

---

## Aufbau des Containers

Der Container beinhaltet zwei Kernkomponenten:

- **Python-Skript (app.py)**  
Dieses Skript bildet den zentralen Punkt im Container. Es stellt den API-Endpunkt (/analyze_log) bereit, lädt das KI-Modell (Llama-2), ruft die Jenkins-Logs ab, führt die semantische Kontextsuche (ChromaDB) durch und kommuniziert mit dem LLM, um daraus eine vollständige Fehleranalyse mit Lösungsvorschlägen zu generieren.
- **Lokales KI-Modell (Llama-2)**  
Wird direkt vom API-Skript geladen und angesteuert.

---

## Funktionsweise des API-Endpunkts (`app.py`)

Der API-Endpunkt wird durch das Python-Webframework **FastAPI** bereitgestellt. Das Skript enthält alle Schritte zur Fehleranalyse:

1. **Bereitstellung der HTTP-Schnittstelle**
   - API-Endpunkt: `/analyze_log`
   - Erwartet als Parameter: Jobname, Build-Nummer, Jenkins API-Token

2. **Abrufen der Jenkins-Logs**
   - Das Skript ruft über HTTP GET (Jenkins API) die Logs des fehlgeschlagenen Builds direkt von Jenkins ab.
  
3. **Fehler extrahieren**
   - Filtert relevante Fehlerzeilen aus den Logs
   
4.  **Chunks bilden**  
      - Um den Kontext optimal aus der ChromaDB abrufen zu können, wird der extrahierte Log-Text in kleinere, semantisch sinnvolle Einheiten („Chunks“) aufgeteilt. 

3. **Semantische Kontextsuche via ChromaDB**
   - Für jeden Chunk wird eine semantische Suche ausgeführt, wodurch sichergestellt wird, dass relevante historische Kontexte gezielt und treffsicher ermittelt werden.

3. **Prompt-Erstellung**
   - Nutzt die Logs und die ChromaDB-Treffer, um einen verständlichen Prompt zu generieren.
   
4. **Aufruf des Llama-2-Modells**
   - Das Prompt wird an das lokale Llama-2-Modell übergeben.
   - Das LLM erstellt daraus eine detaillierte Fehleranalyse und gibt Lösungsvorschläge zurück.

5. **Zurücksenden der Analyse**
   - Die erstellte Analyse wird als HTTP-Response an den aufrufenden Jenkins-Analyzer-Job zurückgegeben.

Die genaue Beschreibung des Skripts findest du hier:  
→ [app.py (detaillierte Erklärung)](skripte-und-dateien.md#apppy)



## Verwendung des Llama-2-Modells

Im selben Container ist das lokal installierte LLM-Modell (Llama-2) verfügbar. Dieses Modell empfängt den vom API-Skript erstellten Prompt und generiert anschließend die KI-Analyse inklusive Lösungsanweisungen.  

Gründe für die Nutzung von Llama-2:  

- Hohe Genauigkeit bei technischen Logs  
- Gute Performance auch in Umgebungen mit begrenztem RAM  
- Breite Unterstützung in der Community  
- Effiziente Nutzung lokaler Ressourcen  

---


## Zugehörige Dateien & Skripte (Verweise)

- [`app.py`](scripts-und-dateien.md#apppy)
- Dockerfile zur Erstellung des Containers  

[→ Übersicht aller Skripte und Dateien](skripte-und-dateien.md)

---

## Weiterführende Links

- [Zurück zur Projektübersicht](erweiterte-fehleranalyse-llm-rag.md)
- [Jenkins-Container](jenkins-container.md)
- [ChromaDB-Container](chromadb-container.md)  
