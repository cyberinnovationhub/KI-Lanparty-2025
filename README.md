# KI-LAN-Party Demo-Umgebung

Willkommen zur KI-LAN-Party Arbeitsumgebung! 👋

Dieses Repository führt Sie in die Einrichtung einer sofort einsatzbereiten Demo-Umgebung für eine KI-LAN-Party mit drei Prototypen: Zwei KI-Agenten basierend auf einem Low-Code-Framework und einen Sprachassistenten.

Die Software läuft lokal auf den Teilnehmerrechnern bzw. bereitgestellter Hardware. Das Setup liefert klare Templates und Anleitungen, damit Teilnehmer selbst Experimente durchführen können.

## Übersicht der Agenten

- **Agent 1**: Design Thinking Assistant (ohne RAG) → [Setup-Anleitung](./Agent_1/README.md)
- **Agent 2**: Dokumentations-Experte (mit RAG) → [Setup-Anleitung](./Agent_2/README.md)
- **Agent 3**: Interview-Assistant mit Sprachsteuerung → [Setup-Anleitung](./Agent_3/README.md)

## Verwendete Software

- **[Docker](https://docs.docker.com/)**: Container-Plattform, die Anwendungen inklusive ihrer Abhängigkeiten in isolierten Containern verpackt
- **[Dify](https://docs.dify.ai/en/introduction)**: Low-Code-Framework zur Erstellung KI-gesteuerter Prototypen
- **[Open WebUI](https://docs.openwebui.com/)**: Grafische Oberfläche zur Interaktion mit lokal laufenden Modellen
- **[Ollama](https://docs.ollama.com/)**: Lokaler Runner zur Bereitstellung von Modellen (z.B. `llama3.1:8b`, `qwen3:8b`)

## Grundlegendes Setup

Das Setup erfolgt in drei wesentlichen Schritten:

1. **Ollama** - Ein lokaler KI-Model Runner (für Agents 1-3)
2. **Docker Desktop** - Die Container-Platform (wird für Dify und Open WebUI)
3. **Dify** - Ein Low-Code AI-Framework (für Agents 1-2)

### Ollama installieren und Modelle vorbereiten

**Installation je nach Betriebssystem:**
- **macOS**: Ollama-Installer von [ollama.com](https://ollama.com) oder via Homebrew: `brew install ollama`
- **Windows**: Ollama-Installer von [ollama.com](https://ollama.com)
- **Linux**: curl-Installation gemäß [ollama.com](https://ollama.com)

**Sprachmodelle herunterladen:**

Option 1 - Terminal:
```bash
ollama pull llama3.1:8b
ollama pull qwen3:8b
# Testen:
ollama list
```

Option 2 - Ollama UI:
1. Starte Ollama
2. Suche die gewünschten Modelle (`llama3.1:8b` und `qwen3:8b`)
  
   ![Ollama Modell suchen](https://github.com/user-attachments/assets/a4fcb0c1-0ffa-49a9-845c-a9024a7b1e02)

3. Sende eine Nachricht im Chat, um den Download zu starten

   ![Ollama Download starten](https://github.com/user-attachments/assets/153270f0-f50a-4bdb-81db-0fca22239f6a)

4. Lade beide Modelle herunter (`llama3.1:8b` und `qwen3:8b`)

### Docker Desktop installieren

1. Lade Docker Desktop von [docker.com](https://docker.com) herunter
2. Installiere und starte Docker Desktop

> **macOS-Hinweis**: Bei MacBooks kann eine Fehlermeldung bezüglich Rosetta erscheinen. Klicke auf das Fenster → Installieren → bei Fehlermeldungen auf 'Retry' klicken. Wenn sich Docker schließt, wurde Rosetta erfolgreich installiert. Docker anschließend manuell neu starten.
  
### Dify lokal starten

1. Dify wird als Repository bereitgestellt. Nutze das Terminal, um das Repository zu klonen und wechsle danach in das Docker-Verzeichnis:
   ```bash
   git clone https://github.com/langgenius/dify
   cd dify/docker
   ```

2. dify benötigt eine Umgebungsdatei `.env`, in der die Konfigurationen vorgenommen wird. In unserem Fall können wir die Beispiel-Datei kopieren:
   - Kopiere `.env.example` zu `.env`: 
   ```bash
   cp .env.example .env
   ```
   - Falls du die Datei per Finder/Explorer kopieren möchtest, musst du gegebenenfalls die **versteckten Dateien anzeigen**:
     - **macOS**: `Cmd + Shift + .`
     - **Windows**: Datei-Explorer → Ansicht → Versteckte Elemente anzeigen

3. Dify starten:
   ```bash
   docker compose up -d
   ```

4. UI öffnen: [http://localhost:3000](http://localhost:3000)

5. Beim ersten Start muss ein lokales Admin-Konto angelegt werden. Dieses wird nur lokal gespeichert.

> **Container-Verwaltung**: In Docker Desktop unter **Containers** den `docker`-Container stoppen/starten über die **Actions**-Buttons. Alternative kann der Befehl `docker compose down` genutzt werden, um alle Container zu stoppen. Im Terminal kann mit `docker compose ps` der Status geprüft werden, und mit `docker compose logs -f` die Logs live verfolgt werden.

### Dify mit Ollama verbinden

1. In Dify UI: **Settings** aufrufen

   ![Dify Settings](https://github.com/user-attachments/assets/520be460-85f1-4ed1-aedc-40ec4ae99a15)

2. **Model Providers** → Suche nach "OpenAI-API-compatible" → **Add model**

   ![OpenAI Compatible Provider](https://github.com/user-attachments/assets/7d57814e-bad2-4b93-8587-86903e9ffa37)

3. Felder ausfüllen (Beispiel für `llama3.1:8b`):
   - **Model Type**: `LLM`
   - **Model Name**: `llama3.1:8b`
   - **Authorization Name**: `Authorization`
   - **Model display name**: `Llama 3.1 8B (Ollama)` (frei wählbar)
   - **API Key**: `ollama` (beliebiger String, wird nicht geprüft)
   - **API endpoint URL**:
     - **macOS/Windows**: `http://host.docker.internal:11434/v1`
     - **Linux**: `http://172.17.0.1:11434/v1`
   - **Model name for API endpoint**: `llama3.1:8b` (muss exakt zum Ollama-Tag passen)

   ![Modell Konfiguration](https://github.com/user-attachments/assets/81869b51-e41e-4480-8793-747bfae8fbd3)

**Windows/WSL-Troubleshooting**: Bei Verbindungsproblemen zwischen Dify und Ollama:
- Umgebungsvariable setzen: `OLLAMA_HOST=0.0.0.0:11434`
- WSL-IP finden mit `ipconfig` in PowerShell
- URL verwenden: `http://<WSL-IP>:11434/v1`
- Ollama neu starten

---

## ✅ Setup abgeschlossen!

**Schnellstart**: Gehe direkt zu den Agent-Anleitungen:
- [Agent 1: Design Thinking Assistant](./Agent_1/README.md)
- [Agent 2: Dokumentations-Experte](./Agent_2/README.md)
- [Agent 3: Interview-Assistant](./Agent_3/README.md)

---


## Nützliche Tipps & Tricks

### Grundlagen der Agenten-Nutzung

**Chat-Format**: Jeder neue Chat ist eine neue Unterhaltung. Du kannst Memory-Funktionen aktivieren, damit sich Agenten an vorherige Chats erinnern.

**Mehrere Agenten**: Erstelle verschiedene Agenten für verschiedene Anwendungsfälle über die Dify-Hauptseite.

### Prompt-Engineering

**Grundprinzip**: Je detaillierter der Kontext, desto besser die Antworten. Stelle dir vor, du erklärst alles einer entfernten Person, die Expertin in vielen Bereichen ist.

**Variablen verwenden**:
- Im Prompt: `{{variablenname}}`
- Beispiel: `Nenne mich bei meinem Namen {{name}}`

**Template-Frameworks**: Nutze etablierte Frameworks wie AUTOMAT für strukturierte Prompts.

### Dokumente hochladen (RAG)

1. **Knowledge** → **Create Knowledge**

   ![Knowledge Tab](https://github.com/user-attachments/assets/ca4d8b25-b9fe-431a-8604-dc58b6d084b8)

2. **Import aus Textdatei** → Drag & Drop → **High Quality** → **Save & Process**

   ![Dokument Upload](https://github.com/user-attachments/assets/de9406e6-5bd1-4fe8-b46b-cf6c1921e591)

3. In der App: **Context** → Dokumente hinzufügen

   ![Context hinzufügen](https://github.com/user-attachments/assets/8b1b30a8-4068-4c05-a297-b9797178ae48)

### Wichtige Einschränkungen

- **Kontextverlust**: Bei langen Chats oder großen Dokumenten kann der Agent frühere Informationen vergessen
- **Halluzinationen**: KI-Agenten können falsche Informationen generieren - immer Antworten überprüfen


## FAQ & Troubleshooting

### Verbindungsprobleme

**Warum `host.docker.internal:11434/v1`?**
- Dify läuft im Docker-Container und braucht eine Brücke zum Host-System
- `/v1` ist für OpenAI-kompatible API erforderlich
- **Linux**: Nutze statt dem Host die Gateway-IP: `docker network inspect bridge`
- **Windows/WSL**: Nutze statt dem Host die WSL-IP, die Du mit `ipconfig` ermitteln kannst.

**Verbindung fehlgeschlagen:**
```bash
# Ollama-Status prüfen
ollama list
# API testen
curl http://localhost:11434/api/tags
```

**"Model not found":**
- Model Name muss exakt dem Ollama-Tag entsprechen
- Prüfe verfügbare Modelle: `ollama list`

### Häufige Fragen

**API Key bei Ollama?** Beliebiger String (z.B. `ollama`) - wird nicht geprüft

**Chat vs. LLM Model Type?** In Dify heißt es "LLM" (nicht "Chat")

**Langsame Antworten?**
- Kleinere Modelle verwenden
- Max-Token in Dify reduzieren
- RAM/CPU-Ressourcen prüfen

**Template Import/Export:**
- **Export**: App → Export Template
- **Import**: Create App → Import DSL File

### Port-Konflikte

Standardports prüfen:
- Dify UI: Port `3000`
- Ollama: Port `11434`

Bei Konflikten: Ports in `docker-compose.yaml` anpassen
