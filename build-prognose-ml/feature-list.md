
### 1. Extrahierte Rohdaten aus der Jenkins API
*Extrahiert mit dem Skript [data_extraction.py](https://github.com/cqNikolaus/jenkins-ki-dokumentation/blob/main/build-prognose-ml/extract-data.md)*

- **build\_number**  
*Beschreibung:* Die fortlaufende ID des Builds (z. B. 1, 2, 3 …).  
*Aussage:* Dient zur Identifikation des Builds; in der Regel nur für Debugging/Referenz.
- **result**  
*Beschreibung:* Das Build-Ergebnis als Text (z. B. "SUCCESS", "FAILURE").  
*Aussage:* Zeigt den Ausgang des Builds an.
- **result\_bin**  
*Beschreibung:* Binäre Kennzeichnung des Ergebnisses (1 = FAILURE, 0 = SUCCESS).  
*Aussage:* Dient als Zielvariable für ML-Modelle.
- **duration\_sec**  
*Beschreibung:* Die tatsächliche Dauer des Builds in Sekunden.  
*Aussage:* Längere oder ungewöhnlich kurze Build-Dauern können auf Probleme hindeuten.
- **estimated\_duration\_sec**  
*Beschreibung:* Von Jenkins geschätzte Dauer des Builds in Sekunden.  
*Aussage:* Wird als Vergleichsmaßstab zur tatsächlichen Dauer genutzt.
- **commits\_count**  
*Beschreibung:* Anzahl der in den Build einfließenden Commits.  
*Aussage:* Mehr Commits können auf umfangreichere Änderungen und damit potenziell höhere Fehleranfälligkeit hindeuten.
- **built\_on**  
*Beschreibung:* Der Node/Agent, auf dem der Build ausgeführt wurde (Standard: "built\_in" bei keinem separaten Agenten).  
*Aussage:* Kann Unterschiede in der Build-Umgebung aufzeigen.
- **build\_url**  
*Beschreibung:* URL zum Build in Jenkins.  
*Aussage:* Nützlich zur direkten Nachverfolgung, aber für ML nicht relevant.
- **parameters**  
*Beschreibung:* Übergebene Parameter als JSON-String.  
*Aussage:* Kann Einfluss auf den Build haben, falls unterschiedliche Parameter zu abweichenden Ergebnissen führen.
- **commit\_authors\_count**  
*Beschreibung:* Anzahl der unterschiedlichen Entwickler, die Commits beigetragen haben.  
*Aussage:* Mehr Autoren können auf höhere Komplexität und Koordinationsaufwand hinweisen.
- **total\_commit\_msg\_length**  
*Beschreibung:* Summe der Zeichenlängen aller Commit-Messages.  
*Aussage:* Längere Commit-Messages könnten auf umfangreichere oder detailliertere Änderungen hindeuten.
- **change\_set\_kind**  
*Beschreibung:* Gibt an, aus welchem Versionskontrollsystem (z. B. Git) die Änderungen stammen.  
*Aussage:* Kann helfen, unterschiedliche Verhaltensmuster zwischen VCS-Typen zu erkennen.
- **culprits\_count**  
*Beschreibung:* Anzahl der von Jenkins identifizierten „culprits“ (mögliche Fehlerverursacher).  
*Aussage:* Zeigt, ob Jenkins bestimmte Änderungen als problematisch einstuft.
- **executor\_name**  
*Beschreibung:* Der Name des Executors, der den Build ausgeführt hat.  
*Aussage:* Kann Unterschiede in der Build-Performance zwischen verschiedenen Executors offenbaren.
- **trigger\_types**  
*Beschreibung:* Textuelle Information darüber, was den Build ausgelöst hat (z. B. manueller Start, SCM-Trigger, Timer).  
*Aussage:* Unterschiedliche Trigger können Einfluss auf die Build-Qualität haben.
- **error\_count**  
*Beschreibung:* Anzahl der Vorkommen von Fehler-Keywords (wie ERROR, Exception) im Konsolenlog.  
*Aussage:* Ein direkter Indikator für Fehler im Build-Log.

---

### 2. Abgeleitete (zusätzliche) Features in der CSV

- **duration\_diff**  
*Ableitung:* Differenz zwischen der tatsächlichen Build-Dauer (*duration\_sec*) und der geschätzten Dauer (*estimated\_duration\_sec*).  
*Aussage:* Zeigt an, ob der Build länger oder kürzer als erwartet dauert.
- **duration\_ratio**  
*Ableitung:* Verhältnis der tatsächlichen Dauer zur geschätzten Dauer (duration\_sec / estimated\_duration\_sec).  
*Aussage:* Gibt einen normierten Vergleich der Build-Dauer; Werte >1 deuten auf längere Builds als erwartet hin.
- **culprit\_ratio**  
*Ableitung:* Verhältnis von *culprits\_count* zu *commit\_authors\_count* (mit +1 im Nenner, um Division durch 0 zu vermeiden).  
*Aussage:* Ein hoher Wert kann darauf hindeuten, dass ein großer Anteil der Autoren als potenzielle Fehlerverursacher identifiziert wurde.
- **build\_date**  
*Ableitung:* Extrahiert aus dem Zeitstempel (timestamp\_ms) im Format "YYYY-MM-DD".  
*Aussage:* Gibt das Datum des Builds an; Grundlage für weitere zeitliche Analysen.
- **build\_time**  
*Ableitung:* Extrahiert aus dem Zeitstempel im Format "HH:MM:SS".  
*Aussage:* Zeigt die genaue Uhrzeit des Builds; nützlich, um zeitliche Muster (z. B. Tageszeiten) zu erkennen.
- **build\_weekday**  
*Ableitung:* Abgeleitet aus dem Datum (timestamp\_ms) – Wochentag (0 = Montag, 6 = Sonntag).  
*Aussage:* Hilfreich, um zu prüfen, ob Builds an bestimmten Wochentagen häufiger fehlschlagen.
- **build\_hour**  
*Ableitung:* Extrahiert die Stunde (0–23) aus dem Zeitstempel.  
*Aussage:* Ermöglicht die Analyse, ob zu bestimmten Tageszeiten Build-Probleme auftreten.
- **build\_month**  
*Ableitung:* Extrahiert den Monat aus dem Zeitstempel.  
*Aussage:* Kann saisonale Trends oder monatliche Schwankungen im Build-Verhalten aufzeigen.
- **build\_year**  
*Ableitung:* Extrahiert das Jahr aus dem Zeitstempel.  
*Aussage:* Dient dazu, langfristige Trends oder Veränderungen im Build-Verhalten über die Jahre zu analysieren.
