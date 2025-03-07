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

3. **Semantische Kontextsuche via ChromaDB**
   - Extrahiert relevante Fehlerzeilen aus den Logs.
   - Führt eine semantische Suche auf der ChromaDB aus, um passende historische Kontexte zu ermitteln.

3. **Prompt-Erstellung**
   - Nutzt die Logs und die ChromaDB-Treffer, um einen verständlichen Prompt zu generieren.
   
4. **Aufruf des Llama-2-Modells**
   - Das Prompt wird an das lokale Llama-2-Modell übergeben.
   - Das LLM erstellt daraus eine detaillierte Fehleranalyse und gibt Lösungsvorschläge zurück.

5. **Zurücksenden der Analyse**
   - Die erstellte Analyse wird als HTTP-Response an den aufrufenden Jenkins-Analyzer-Job zurückgegeben.

---

## Technische Umsetzung

Die komplette Logik (API & LLM-Nutzung) wird durch ein einzelnes Python-Skript (`app.py`) bereitgestellt:

- Das Python-Skript startet eine REST-API mittels `FastAPI` und stellt den Endpunkt `/analyze_log` bereit.
- Das Llama-2-Modell wird durch das Python-Modul `llama_cpp` geladen und innerhalb dieses Skriptes verwendet.
- Für die semantische Suche nach Embeddings greift das Skript via `chromadb` auf den ChromaDB-Container zu.
- Die gesamte Logik (Log-Abruf, Prompt-Erstellung, LLM-Analyse, Kontextabruf) erfolgt ausschließlich in diesem Skript.

Die genaue Beschreibung des Skripts findest du hier:  
→ [app.py (detaillierte Erklärung)](scripts-und-dateien.md#apppy)

---

## Semantische Suche mit ChromaDB

Das API-Skript nutzt intern eine Verbindung zur ChromaDB, um mithilfe einer embeddingsbasierten semantischen Suche relevante historische Daten abzurufen. Diese Daten werden dem Prompt hinzugefügt, um eine präzise Fehleranalyse mit Kontextbezug zu ermöglichen.

Mehr zu ChromaDB und deren Einrichtung findest du hier:  
→ [ChromaDB-Container](chromadb-container.md)

---

## Verwendung des Llama-2-Modells

Im selben Container ist das lokal installierte LLM-Modell (Llama-2) verfügbar. Dieses Modell empfängt den vom API-Skript erstellten Prompt und generiert anschließend die KI-Analyse inklusive Lösungsanweisungen.  

Gründe für die Nutzung von Llama-2:  

- Hohe Genauigkeit bei technischen Logs  
- Gute Performance auch in Umgebungen mit begrenztem RAM  
- Breite Unterstützung in der Community  
- Effiziente Nutzung lokaler Ressourcen  

---

## Zusammenfassung des Workflows im LLM-API-Container

Der Ablauf im Container zusammengefasst:

1. API-Endpunkt erhält Anfrage vom Analyzer-Job (Jenkins)
2. API ruft Logs aus Jenkins ab (intern über HTTP)
3. API führt semantische Suche in der ChromaDB durch
4. API generiert Prompt (Logs + ChromaDB-Kontext)
5. API übergibt Prompt an Llama-2-Modell
6. LLM erstellt Fehleranalyse & Lösungsvorschläge
5. API sendet Analyseergebnis an Analyzer-Job zurück

---

## Besonderheiten und Designentscheidungen

- **Vollständig offline**
  - Container hat keinen Zugriff auf das Internet
  - Modell und alle benötigten Ressourcen sind lokal verfügbar
  
- **Effiziente Nutzung der Ressourcen**
  - Llama-2 wurde gewählt, weil es effizient auf einer 16GB-RAM VM betrieben werden kann, weit verbreitet ist und zuverlässig bei technischer Log-Analyse arbeitet.

---

## Zugehörige Dateien & Skripte (Verweise)

- [`app.py`](scripts-und-dateien.md#apppy)
- Dockerfile zur Erstellung des Containers  

[→ Übersicht aller Skripte und Dateien](scripts-und-dateien.md)

---

## Weiterführende Links

- [Zurück zur Projektübersicht](erweiterte-fehleranalyse-llm-rag.md)
- [Jenkins-Container](jenkins-container.md)
- [ChromaDB-Container](chromadb-container.md)  

---

Mit dieser Struktur sind die Abläufe eindeutig und verständlich geordnet, und dein Leser weiß genau, was in welchem Schritt passiert.  
Soll ich noch etwas weiter anpassen, oder passt dir diese Version jetzt so?
