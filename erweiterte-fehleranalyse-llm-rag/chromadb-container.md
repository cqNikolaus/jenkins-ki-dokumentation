# ChromaDB-Container

Diese Seite beschreibt den **ChromaDB-Container**, seine Aufgaben im Gesamtworkflow und die Besonderheiten bei der Nutzung als semantische Wissensdatenbank.

---

## Zweck des Containers

Der ChromaDB-Container stellt die zentrale Wissensdatenbank für den LLM-Analyseprozess bereit. Er speichert vorverarbeitete, embeddings-basierte Textinhalte (Domänenwissen), um bei neuen Fehlerfällen schnell relevante historische Kontextinformationen liefern zu können.

Die ChromaDB wird genutzt, um durch semantische Ähnlichkeitssuche (Retrieval-Augmented Generation, kurz: **RAG**) relevante Lösungshinweise und Kontextdaten zu finden.

---

## Warum ChromaDB?

ChromaDB wurde bewusst gewählt, da sie speziell für effiziente semantische Suchen mit Embeddings ausgelegt ist. Für unseren Einsatzzweck (automatische Fehleranalyse mit KI) ist diese Funktion optimal geeignet, da so gezielt relevante historische Informationen gefunden und als Kontext für das LLM bereitgestellt werden können.

Weitere Gründe für ChromaDB in diesem Projekt:

- Schnelle semantische Suche nach Embeddings
- Leichtgewichtige Datenbank, einfacher Betrieb
- Sehr gute Integration mit Python und gängigen Embedding-Modellen

---

## Inhalt und Funktionsweise

Der Container enthält eine **einzelne Collection**, die Embeddings aus einem vorbereiteten Support-Chat enthält. Diese Embeddings wurden zuvor aus Texten generiert und in die Datenbank geladen.

### Erzeugung & Befüllung der Embeddings-Datenbank

Die Embeddings wurden lokal generiert und dann mittels eines Skripts (`ingestion.py`) innerhalb des Containers direkt in die Datenbank geladen.

- Skript zur Erstellung von Embeddings:  
  [`text_to_embeddings.py`](skripte-und-dateien.md#text_to_embeddingspy) (lokale Ausführung)
- Ingestion-Skript zum Befüllen der Datenbank:  
  [`ingestion.py`](skripte-und-dateien.md#ingestionpy)

### Ablauf der Befüllung (kurz):

- Der Support-Chat-Text wird in sinnvolle Chunks zerlegt.
- Für jeden Chunk wird mithilfe eines Embedding-Modells ein Embedding-Vektor generiert.
- Die Embeddings werden als JSON (`embeddings.json`) gespeichert.
- Anschließend werden diese Embeddings über das Skript `ingestion.py` direkt in die ChromaDB geladen.

---

## Nutzung im laufenden Betrieb

Wenn der API-Endpunkt im LLM-API-Container Logs von Jenkins empfängt, sucht er in der ChromaDB nach passenden historischen Kontexten:

- Die Logs werden in Chunks aufgeteilt.
- Jeder Chunk wird einzeln über eine semantische Suche abgefragt.
- ChromaDB gibt die relevantesten Ergebnisse zurück.
- Der gefundene Kontext fließt in den Prompt für das lokale Llama-2-Modell ein.

---

## Skripte & Dateien (zur Erzeugung der Embeddings & Befüllung)

- [`text_to_embeddings.py`](skripte-und-dateien.md#text_to_embeddingspy):  
  Erzeugt lokal aus Rohtexten Embeddings.
  
- [`ingestion.py`](skripte-und-dateien.md#ingestionpy):  
  Lädt Embeddings in die ChromaDB-Collection.

---

## Weiterführende Links

- [Zurück zur Projektübersicht](erweiterte-fehleranalyse-llm-rag.md)
- [Jenkins-Container](jenkins-container.md)
- [LLM-API-Container](llm-api-container.md)
