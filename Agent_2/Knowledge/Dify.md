# Dify - Low-Code AI-Framework

## Was ist Dify?

Dify ist eine Open-Source-Plattform, die es ermöglicht, KI-Anwendungen mit einem visuellen Low-Code-Ansatz zu erstellen. Statt Code zu schreiben, können Nutzer KI-Agenten, Chatbots und Workflows über eine grafische Benutzeroberfläche zusammenklicken.

## Wofür wird es im Projekt verwendet?

In der KI-LAN-Party wird Dify für **Agent 1** (Design Thinking Assistant) und **Agent 2** (Dokumentations-Experte mit RAG) verwendet:

- **Agent 1**: Strukturierter Chatbot ohne externe Wissensbasis
- **Agent 2**: RAG-basierter Agent mit Zugriff auf hochgeladene Dokumentationen
- **Visuelle Konfiguration**: System-Prompts, Modellauswahl und RAG-Integration erfolgen über die Web-UI
- **Lokale Ausführung**: Läuft als Docker-Container und verbindet sich mit lokal laufenden Ollama-Modellen

## Kernkonzepte

### 1. Apps und App-Typen
Dify bietet verschiedene App-Typen:
- **Chat App**: Konversationsbasierte Agenten mit Memory-Funktion
- **Agent**: Erweiterte Agenten mit Tool-Nutzung (z.B. Web-Suche, API-Calls)
- **Chatflow/Workflow**: Visuelle Node-basierte Prozesse

Im Projekt werden hauptsächlich **Chat Apps** verwendet.

### 2. Model Providers
Dify kann verschiedene LLM-Anbieter integrieren:
- **OpenAI-API-compatible**: Für Ollama (lokale Modelle)
- **Azure OpenAI**: Für Cloud-basierte GPT-Modelle
- **Weitere**: Anthropic Claude, Google Gemini, Hugging Face, etc.

Die Konfiguration erfolgt unter **Settings → Model Providers**.

### 3. Knowledge Base (RAG)
Dify bietet integrierte RAG-Funktionalität:
- **Document Upload**: Markdown, PDF, TXT, DOCX hochladen
- **Chunking & Embedding**: Automatische Verarbeitung der Dokumente
- **Index Modes**:
  - **Economical**: Einfache Keyword-Suche (kein Embedding-Modell nötig)
  - **High Quality**: Semantische Suche mit Embeddings (benötigt Embedding-Modell)
- **Retrieval**: Relevante Dokumentenausschnitte werden automatisch in den Prompt eingefügt

### 4. System Prompts
Der System Prompt definiert die Rolle, Persönlichkeit und Verhaltensweise des Agenten:
- Wird bei jeder Anfrage an das LLM gesendet
- Definiert Kontext, Aufgaben und Einschränkungen
- Kann Variablen enthalten (z.B. `{{name}}` für dynamische Werte)

### 5. Context und Memory
- **Context**: Zusätzliches Wissen, das bei jeder Anfrage bereitgestellt wird (z.B. aus Knowledge Base)
- **Memory**: Erinnerungsfunktion über mehrere Chat-Sessions hinweg

### 6. DSL (Domain Specific Language) Templates
Dify-Apps können als YAML-Dateien (.yml) exportiert und importiert werden:
- **Export**: Sichert Konfiguration, Prompts und Workflows
- **Import**: Ermöglicht schnelles Setup vorkonfigurierter Agenten
- Im Projekt: `Agent 1.yml` und `Agent 2.yml` Templates

## Vorteile für Lern-Projekte

- **Schnelles Prototyping**: Keine Programmierung erforderlich
- **Visuelle Konfiguration**: Leicht verständliche UI
- **Flexibel**: Unterstützt lokale und Cloud-Modelle
- **RAG-Integration**: Out-of-the-box Dokumenten-Suche
- **Open Source**: Kostenlos, transparent, erweiterbar

## Typische Workflows

### Agent erstellen
1. **Create App** → **Chat App**
2. **Model Provider** auswählen
3. **System Prompt** konfigurieren
4. (Optional) **Knowledge Base** hinzufügen
5. **Publish** → Testen

### Knowledge Base hinzufügen
1. **Knowledge** → **Create Knowledge**
2. Dokumente hochladen (Drag & Drop)
3. **Index Mode** wählen (Economical/High Quality)
4. **Save & Process**
5. In App: **Context** → Knowledge hinzufügen

### Modell mit Ollama verbinden
1. **Settings** → **Model Providers** → **OpenAI-API-compatible**
2. Konfiguration:
   - **Model Name**: `llama3.1:8b` (oder anderes Ollama-Modell)
   - **API Endpoint**: `http://host.docker.internal:11434/v1`
   - **API Key**: Beliebiger String (z.B. `ollama`)
3. **Save** → Modell in App auswählen

## Limitierungen

- **Kontextlänge**: Abhängig vom verwendeten Modell (meist 4k-128k Tokens)
- **RAG-Qualität**: High Quality Mode benötigt gutes Embedding-Modell
- **Performance**: Lokale Modelle langsamer als Cloud-APIs
- **Komplexität**: Sehr komplexe Workflows können unübersichtlich werden

## Weiterführende Links

- **Offizielle Dokumentation**: [https://docs.dify.ai](https://docs.dify.ai)
- **GitHub Repository**: [https://github.com/langgenius/dify](https://github.com/langgenius/dify)
- **Community Discord**: Für Fragen und Support

---

**Quellen**: Eigene Zusammenstellung basierend auf Dify-Dokumentation und Projekt-Setup