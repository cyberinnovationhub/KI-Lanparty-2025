# Open WebUI - Alternative UI für LLMs mit Sprachfunktionen

## Was ist Open WebUI?

Open WebUI ist eine benutzerfreundliche Web-Oberfläche für die Interaktion mit Large Language Models. Es bietet eine ChatGPT-ähnliche Benutzeroberfläche und unterstützt sowohl lokale Modelle (via Ollama) als auch Cloud-APIs (OpenAI, Anthropic). Ein besonderes Feature sind die integrierten **Sprachfunktionen** (Text-to-Speech und Speech-to-Text).

## Wofür wird es im Projekt verwendet?

Open WebUI wird für **Agent 3** (Interview-Assistant) verwendet:

- **Sprachchat**: Natürliche Gesprächsführung mit TTS/STT für realistische Interview-Simulationen
- **Alternative zu Dify**: Einfachere, ChatGPT-ähnliche UI
- **Direkter Ollama-Zugriff**: Nutzt lokale Modelle ohne zusätzliche Konfiguration
- **System Prompts**: Definiert Interview-Struktur und Verhalten des Assistenten

**Unterschied zu Dify**:
- **Dify**: Low-Code-Plattform mit visuellen Workflows, RAG-Integration, Multi-App-Verwaltung
- **Open WebUI**: Fokus auf Chat-Interface, Sprachfunktionen, einfachere Konfiguration

## Kernkonzepte

### 1. Sprachfunktionen (TTS/STT)

**Text-to-Speech (TTS)**:
- Wandelt Agent-Antworten in gesprochene Sprache um
- Nutzt Browser-native Speech Synthesis API (keine zusätzliche Installation)
- Verfügbare Stimmen abhängig von OS und Browser
- **Beste Erfahrung**: Safari auf macOS (natürlichste Stimmen)

**Speech-to-Text (STT)**:
- Wandelt gesprochene Eingaben in Text um
- Nutzt Browser-native Speech Recognition API
- Aktivierung über Sound-Wave-Button in der Chat-Leiste
- Funktioniert am besten in ruhiger Umgebung

**Konfiguration** (Settings → Audio):
- **Geschwindigkeit**: 1.0 - 1.5x (empfohlen: 1.1 - 1.25x)
- **Stimme**: Deutsche Stimmen bevorzugen (z.B. "Flo (German (Germany))")
- **Auto-Submit**: Optional automatisches Senden nach Spracheingabe

### 2. System Prompts

System Prompts definieren die Persönlichkeit und das Verhalten des Agenten:
- **Global**: Settings → General → System Prompt (gilt für alle Chats)
- **Pro Chat**: In Chat-Settings individuell überschreibbar
- **Agent 3**: Nutzt detaillierten System Prompt mit Interview-Struktur (5 Phasen)

**Tipp**: Lange, strukturierte System Prompts funktionieren gut für komplexe Rollenspiele wie Interviews.

### 3. Modell-Auswahl

Open WebUI erkennt automatisch verfügbare Ollama-Modelle:
- **Model Dropdown**: Zeigt alle mit `ollama list` verfügbaren Modelle
- **Wechsel während Chat**: Möglich (neues Modell kennt bisherigen Kontext nicht)
- **Für Agent 3**: `llama3.1:8b` oder `qwen3:8b` empfohlen

### 4. Chat-Verwaltung

- **Neue Chats**: Jeder neue Chat ist eine unabhängige Konversation
- **Chat-Historie**: Alle Chats werden lokal gespeichert (im Docker Volume)
- **Chat-Aktionen**: Umbenennen, Löschen, Teilen, Exportieren
- **Memory-Funktionen**: Optional Cross-Chat-Memory (experimentell)

### 5. Docker-Setup

Open WebUI läuft als einzelner Docker-Container:

```bash
# Standard-Setup
docker run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```

**Wichtige Parameter**:
- **`-p 3000:8080`**: Host-Port 3000 → Container-Port 8080
- **`-v open-webui:/app/backend/data`**: Persistente Datenspeicherung
- **`--name open-webui`**: Container-Name für einfaches Management

**GPU-Version**:
```bash
docker run -d -p 3000:8080 --gpus all \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:cuda
```

### 6. Port-Konflikt mit Dify

Beide nutzen Host-Port **3000**:
- **Dify starten**: Open WebUI muss gestoppt sein
- **Open WebUI starten**: Dify muss gestoppt sein

**Container stoppen/starten**:
```bash
# Dify stoppen
cd dify/docker && docker compose down

# Open WebUI starten
docker start open-webui

# Open WebUI stoppen
docker stop open-webui

# Dify starten
cd dify/docker && docker compose up -d
```

## Typische Workflows

### Ersten Account erstellen
1. Container starten und `http://localhost:3000` öffnen
2. **Sign Up** klicken
3. E-Mail, Name und Passwort eingeben (nur lokal gespeichert)
4. **Erstes Konto = Admin** mit vollen Rechten

### System Prompt konfigurieren
1. **Profilbild** → **Settings** → **General**
2. **System Prompt** Textfeld: Gewünschten Prompt einfügen
3. **Save** klicken
4. Neuen Chat starten (gilt nicht für bestehende Chats)

### Sprachchat nutzen
1. **Settings** → **Audio** → **TTS Settings**:
   - Geschwindigkeit: `1.1 - 1.25x`
   - Stimme: Deutsche Stimme wählen (falls verfügbar)
2. **Save** und Settings schließen
3. Im Chat: **Sound-Wave-Button** klicken
4. Sprechen → Automatische Transkription → Antwort wird vorgelesen

### Interview-Session mit Agent 3
1. System Prompt für Agent 3 setzen (siehe Agent_3/README.md)
2. Neuen Chat starten
3. Sprachchat aktivieren
4. Agent begrüßt und startet Interview-Phase 1
5. Auf Fragen antworten (per Sprache oder Text)
6. Agent führt durch 5 Interview-Phasen
7. Am Ende: Zusammenfassung und Feedback

## Vorteile

- **Einfach**: ChatGPT-ähnliche UI, intuitiv bedienbar
- **Sprachfunktionen**: Out-of-the-box TTS/STT ohne zusätzliche Software
- **Lokal**: Vollständig offline nutzbar (mit Ollama)
- **Flexibel**: Unterstützt Ollama, OpenAI, Anthropic, etc.
- **Open Source**: Kostenlos, transparent, erweiterbar

## Limitierungen

- **Kein visuelles Workflow-Design**: Nur Text-basierte System Prompts (keine Nodes wie in Dify)
- **Eingeschränkte RAG-Funktionen**: Dokumenten-Upload möglich, aber weniger ausgereift als Dify
- **Sprachqualität**: Abhängig von Browser und OS (Safari > Chrome > Firefox)
- **STT-Genauigkeit**: Bei Hintergrundgeräuschen oder Akzenten unzuverlässig

## Unterschied Open WebUI vs. Dify

| Feature | Open WebUI | Dify |
|---------|-----------|------|
| **Zielgruppe** | Endnutzer-Chat | App-Entwicklung |
| **UI-Paradigma** | Chat-Interface | Low-Code-Workflows |
| **RAG** | Basic | Advanced (High Quality Mode) |
| **Sprachfunktionen** | ✅ TTS/STT | ❌ Keine |
| **Multi-App** | ❌ Nur Chats | ✅ Mehrere Apps |
| **System Prompts** | Global/Pro Chat | Pro App |
| **Komplexität** | Einfach | Mittel |
| **Use Case im Projekt** | Agent 3 (Interview) | Agents 1 & 2 |

## Häufige Fehler

### Keine deutschen Stimmen verfügbar
**Ursache**: OS oder Browser unterstützt keine deutschen TTS-Stimmen
**Lösung**:
- **macOS**: Safari nutzen (beste deutsche Stimmen)
- **Windows**: Edge nutzen oder zusätzliche Stimmen installieren (Settings → Time & Language → Speech)
- Notfalls: Englische Stimme nutzen

### Sprachchat funktioniert nicht
**Ursache**: Browser-Berechtigung nicht erteilt oder HTTPS erforderlich
**Lösung**:
- Browser-Berechtigung für Mikrofon prüfen
- Bei Remote-Zugriff: HTTPS erforderlich (localhost funktioniert ohne)
- Browser-Konsole auf Fehler prüfen

### Open WebUI startet nicht (Port 3000 belegt)
**Ursache**: Dify läuft noch
**Lösung**:
```bash
cd dify/docker && docker compose down
docker start open-webui
```

### Chats verschwunden nach Container-Neustart
**Ursache**: Container ohne Volume-Mapping gestartet
**Lösung**: Immer `-v open-webui:/app/backend/data` verwenden

## Browser-Empfehlungen für Sprachfunktionen

| Browser | TTS-Qualität | STT-Qualität | Empfehlung |
|---------|--------------|--------------|------------|
| **Safari (macOS)** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Beste Wahl |
| **Chrome** | ⭐⭐⭐ | ⭐⭐⭐⭐ | Gut |
| **Edge (Windows)** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Sehr gut |
| **Firefox** | ⭐⭐ | ⭐⭐ | Eingeschränkt |

## Weiterführende Links

- **Offizielle Dokumentation**: [https://docs.openwebui.com](https://docs.openwebui.com)
- **GitHub Repository**: [https://github.com/open-webui/open-webui](https://github.com/open-webui/open-webui)
- **Discord Community**: Für Support und Fragen
- **Docker Hub**: [https://hub.docker.com/r/ghcr.io/open-webui/open-webui](https://hub.docker.com/r/ghcr.io/open-webui/open-webui)

---

**Quellen**: Eigene Zusammenstellung basierend auf Open WebUI-Dokumentation und Projekt-Setup