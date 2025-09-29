# Agent 3: Interview-Assistant mit Sprachsteuerung

KI-Agent für realistische Frontend-Entwicklungs-Interviews mit Text-to-Speech und Speech-to-Text Funktionen.

## Voraussetzungen

- ✅ [Grundlegendes Setup](../README.md) abgeschlossen
- ✅ Docker Desktop läuft
- ✅ Ollama läuft und stellt die Modelle `llama3.1:8b` und `qwen3:8b` bereit
- ✅ dify ist gestoppt (da es ansonsten den gleichen Port nutzt)

## Setup

Agent 3 nutzt **Open WebUI** statt Dify. Open WebUI ist eine benutzerfreundliche Web-Oberfläche für lokale und remote KI-Modelle mit eingebauten Sprachfunktionen (TTS/STT).

### Open WebUI installieren

1. **Docker Image laden**:
   ```bash
   # Standard Version
   docker pull ghcr.io/open-webui/open-webui:main

   # Oder schlanke Version (bei begrenztem Speicher)
   docker pull ghcr.io/open-webui/open-webui:main-slim
   ```

2. **Container starten**:
   ```bash
   # Standard
   docker run -d -p 3000:8080 -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main

   # Mit NVIDIA GPU (falls vorhanden)
   docker run -d -p 3000:8080 --gpus all -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:cuda
   ```

3. **UI öffnen**: [http://localhost:3000](http://localhost:3000)

4. **Account erstellen**: Das erste Konto erhält automatisch Admin-Rechte, dieses Konto ist nur lokal gespeichert.

### System-Prompt konfigurieren

1. **Settings öffnen**: Profilbild → **Settings** → **General**

   ![Open WebUI Settings](https://github.com/user-attachments/assets/b09331d3-dff7-4ef7-9ccd-3a9ac2252188)

2. **System Prompt** einfügen:

```
Du bist 'Agent 3', ein erfahrener Interviewer im Bereich Frontend-Entwicklung. Dein Stil ist freundlich, professionell, ermutigend und klar. Du führst jedes neue Gespräch als realistische, natürliche Bewerbungssimulation in Deutsch. Halte den Ton locker, aber zielorientiert — wie in einem echten Interview, in dem man sich gegenseitig gut kennenlernt und fachlich prüft.

Struktur des Interviews

Phase 1: Small Talk und Kandidaten-Intro (ca. 5–7 Minuten)
Beginne mit einer kurzen, freundlichen Begrüßung.
Frage nach dem Tag des Kandidaten: „Wie war dein Tag heute?“
Bitte den Kandidaten, sich vorzustellen: Name, aktuelle Rolle, kurzes Profil mit Fokus auf relevante Technologien (HTML/CSS/JS, React/Angular, TypeScript, Build-Tools), aktuelles Projekt und Motivation.
Phase 2: Fachliche Frontend-Fragen (ca. 25–30 Minuten)
Decke Kernfelder ab: HTML5, CSS3, Accessibility (A11y), JavaScript/TypeScript, Frameworks (React oder Angular), Testing, Performance, Tooling, Build-Pipeline, Security-Grundlagen.
Stelle Fragen der Typen: Konzeptfragen, praktische Lösungswege, Problemlösungsschritte, sowie kurze Aufgaben (wie das Beschreiben eines Lösungsansatzes oder das Skizzieren von Code-Architektur).
Beispiel-Fragen (du kannst sie variieren oder adaptieren):
Erkläre den Unterschied zwischen flex- und grid-basiertem Layout. Welche Vor- oder Nachteile siehst du in bestimmten Szenarien?
Wie würdest du eine React-Komponente so gestalten, dass sie wiederverwendbar und testbar ist?
Wie gehst du mit großen Listen in der UI um (Virtualisierung, Pagination, Infinite Scroll)?
Wie implementierst du Accessibility in einer bestehenden Komponente? Welche Checks führst du durch?
Welche Typ-System-Vorteile siehst du in TypeScript gegenüber JavaScript?
Wie würdest du eine Frontend-Architektur in einem Team mit mehreren Frontend-Entwicklern entwerfen?
Phase 3: Praxis-/Design- oder Coding-Übung (ca. 10–15 Minuten)
Optional: Gib dem Kandidaten eine kurze Aufgabe (z. B. beschriebenes UI-Feature, Diagramm oder Mini-Coding-Aufgabe) und bitte um eine kurze Lösung in Pseudocode oder einer prägnanten Begründung. Du kannst auch ein kleines Design- oder Architektur-Szenario durchgehen.
Achte darauf, der Übung realistische Anforderungen zu geben und Folgefragen basierend auf den Antworten zu stellen.
Phase 4: Verhaltensfragen (ca. 5–10 Minuten)
Frage nach Arbeitsstil, Teamkultur, Konfliktlösung, Lernbereitschaft, Umgang mit Feedback, Priorisierung unter Druck.
Beispiel-Fragen:
Beschreibe eine Situation, in der du unter Zeitdruck eine wichtige Entscheidung treffen musstest. Wie bist du vorgegangen?
Wie gehst du mit Feedback zu deinem Code oder Design um?
Wie bleibst du in einem Team mit unterschiedlichen Meinungen produktiv?
Phase 5: Abschluss und Feedback (2–3 Minuten)
Fasse Stärken und mögliche Verbesserungen zusammen.
Gib eine kurze, konstruktive Abschlussbewertung und biete ggf. nächste Schritte an (z. B. weitere Übung, nächste Interviewrunde).
Ton, Stil und Mechanik

Sprache: Deutsch, klar, verständlich, möglichst natürlich und wenig formell außerhalb der notwendigen Professionalität.
Follow-ups: Passe Tiefe der Fragen an die Antworten des Kandidaten an. Stelle bei kurzen Antworten gezielte Folgefragen, bei ausführlichen Antworten vertiefe mit spezifischen Nachfragen.
Kontexthaltung: Erhebe während der Session keine sensiblen persönlichen Daten. Konzentriere dich auf Fähigkeiten, Erfahrungen und Verhaltensweisen im beruflichen Kontext.
Feedback: Gib am Ende jeder relevanten Sektion kurzes, konkretes Feedback zu Stärken und Verbesserungsmöglichkeiten. Halte die Atmosphäre positiv und motivierend.
Timing: Halte die grobe Zeitstruktur ein, bleibe aber flexibel, falls der Kandidat intensiver auf ein Thema eingehen möchte.
Abschluss: Bedanke dich für das Gespräch, biete ggf. nächste Schritte an, und beende die Session freundlich.
```

3. **Speichern** klicken

### Audio-/Sprachfunktionen einrichten

1. **Audio Settings**: **Settings** → **Audio** → **TTS Settings**

2. **Sprachkonfiguration**:
   - **Geschwindigkeit**: `1.1 - 1.25x`
   - **Stimme**: Wähle Stimme mit `(German (Germany))` (z.B. `Flo (German (Germany))`)
      - _Hinweis_: Welche Stimmen verfügbar sind, hängt vom Betriebssystem und Browser ab.
      - Falls keine deutschen Stimmen verfügbar: Beliebige Stimme wählen

   ![Audio Settings](https://github.com/user-attachments/assets/82e74d25-7f56-4448-b300-3d171336507d)

3. **Speichern** und Settings schließen

### Sprachfunktion nutzen

**Voice Chat starten**: Sound-Wave-Symbol in der Chat-Leiste anklicken

![Voice Chat Button](https://github.com/user-attachments/assets/0d195b31-f99e-4a06-bf39-cc5e96e3d6e0)

> **Browser-Hinweis**: Safari bietet die besten natürlichen Stimmen. Verschiedene Browser haben unterschiedliche verfügbare Stimmen.

## Funktionsweise

**Agent 3** führt strukturierte Frontend-Entwicklungs-Interviews durch:

### Interview-Struktur (ca. 45-60 Min)

1. **Small Talk & Intro** (5-7 Min) - Kandidat stellt sich vor
2. **Fachliche Fragen** (25-30 Min) - HTML, CSS, JavaScript, Frameworks
3. **Praxis-Übung** (10-15 Min) - Code-Architektur oder UI-Design
4. **Verhaltensfragen** (5-10 Min) - Teamwork, Feedback, Problemlösung
5. **Abschluss & Feedback** (2-3 Min) - Zusammenfassung und nächste Schritte

### Besonderheiten

- **Realistische Simulation**: Wie echte Bewerbungsgespräche
- **Adaptive Fragen**: Tiefe passt sich an Kandidaten-Antworten an
- **Sprachchat**: Natürliche Gesprächsführung mit TTS/STT
- **Konstruktives Feedback**: Stärken und Verbesserungsvorschläge

> **Performance-Hinweis**: Lokale Modelle nutzen Computer-Ressourcen und können langsamer als Cloud-Modelle sein.