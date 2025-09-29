# Ollama - Lokaler LLM-Runner

## Was ist Ollama?

Ollama ist eine Software, die es ermöglicht, Large Language Models (LLMs) lokal auf dem eigenen Computer auszuführen. Es funktioniert ähnlich wie Docker für Container, aber speziell für KI-Modelle: Modelle können mit einem einfachen Befehl heruntergeladen, verwaltet und gestartet werden.

## Wofür wird es im Projekt verwendet?

Ollama ist das **Herzstück** aller drei Agenten in der KI-LAN-Party:

- **Lokale Modell-Bereitstellung**: Stellt Sprachmodelle wie `llama3.1:8b` und `qwen3:8b` bereit
- **API-Schnittstelle**: Bietet OpenAI-kompatible API unter `http://localhost:11434/v1`
- **Keine Cloud-Abhängigkeit**: Modelle laufen vollständig offline auf der eigenen Hardware
- **Für alle Agents**: Agent 1, 2 und 3 nutzen Ollama-Modelle

## Kernkonzepte

### 1. Modell-Tags und Varianten

Ollama-Modelle haben Tags, die Version und Größe angeben:
- **`llama3.1:8b`**: Llama 3.1 mit 8 Milliarden Parametern (~4.7 GB Download)
- **`qwen3:8b`**: Qwen 3 mit 8 Milliarden Parametern (~4.7 GB)
- **`nomic-embed-text`**: Embedding-Modell für RAG (~274 MB)

Größere Modelle (z.B. `:70b`) sind leistungsfähiger, benötigen aber deutlich mehr RAM und sind langsamer.

### 2. Pull, Run, List - Grundbefehle

```bash
# Modell herunterladen
ollama pull llama3.1:8b

# Verfügbare Modelle anzeigen
ollama list

# Interaktiver Chat im Terminal (zum Testen)
ollama run llama3.1:8b

# Modell löschen
ollama rm llama3.1:8b
```

### 3. API-Modus

Ollama startet automatisch einen API-Server im Hintergrund:
- **Endpoint**: `http://localhost:11434`
- **OpenAI-kompatibel**: `/v1/chat/completions` Endpunkt
- **Keine Authentifizierung**: Lokal offen, keine API-Keys nötig

Diese API wird von Dify und Open WebUI genutzt, um mit den Modellen zu kommunizieren.

### 4. Docker-Integration: `host.docker.internal`

Da Dify in einem Docker-Container läuft, kann es nicht direkt `localhost:11434` erreichen. Stattdessen:
- **macOS/Windows**: `http://host.docker.internal:11434/v1`
- **Linux**: `http://172.17.0.1:11434/v1` (Docker Bridge IP)
- **WSL2**: WSL-IP mit `ipconfig` ermitteln und verwenden

`host.docker.internal` ist eine spezielle Docker-Adresse, die auf den Host-Rechner verweist.

### 5. Modelfile - Eigene Modelle konfigurieren

Ollama unterstützt benutzerdefinierte Modellkonfigurationen über Modelfiles:
```
FROM llama3.1:8b
PARAMETER temperature 0.7
PARAMETER top_p 0.9
SYSTEM "Du bist ein hilfreicher Assistent."
```

Im Projekt wird dies nicht verwendet, da die Prompts in Dify/Open WebUI konfiguriert werden.

## Systemanforderungen

### Minimale Anforderungen (8B-Modelle)
- **RAM**: 8 GB (besser: 16 GB)
- **Speicher**: ~10 GB pro Modell
- **CPU**: Moderne CPU mit AVX2-Unterstützung
- **GPU**: Optional (beschleunigt Inferenz erheblich)

### GPU-Beschleunigung
- **NVIDIA**: Automatisch mit CUDA (empfohlen)
- **AMD**: ROCm-Unterstützung (experimentell)
- **Apple Silicon (M1/M2/M3)**: Native Metal-Beschleunigung

Ohne GPU sind Antworten langsamer (5-20 Sekunden statt 1-3 Sekunden).

## Typische Workflows

### Modell installieren und testen
```bash
# 1. Modell herunterladen
ollama pull llama3.1:8b

# 2. Verfügbarkeit prüfen
ollama list

# 3. Im Terminal testen
ollama run llama3.1:8b
>>> Hallo, wie geht es dir?

# 4. API testen
curl http://localhost:11434/api/tags
```

### Modell in Dify einbinden
1. Ollama muss laufen (Icon in Systemleiste)
2. Modell mit `ollama pull` herunterladen
3. In Dify: **Settings** → **Model Providers** → **OpenAI-API-compatible**
4. Konfiguration:
   - **Model Name**: Exakt wie in `ollama list` (z.B. `llama3.1:8b`)
   - **API Endpoint**: `http://host.docker.internal:11434/v1`
   - **API Key**: Beliebig (z.B. `ollama`)

### Problemlösung: Verbindung fehlgeschlagen
```bash
# 1. Ollama-Status prüfen
ollama list

# 2. API erreichbar?
curl http://localhost:11434/api/tags

# 3. Dify kann Host erreichen? (im Docker-Container testen)
docker exec -it docker-api-1 curl http://host.docker.internal:11434/api/tags

# 4. WSL2: Umgebungsvariable setzen
export OLLAMA_HOST=0.0.0.0:11434
```

## Vorteile

- **Privacy**: Keine Daten verlassen den Computer
- **Kostenlos**: Keine API-Gebühren
- **Offline**: Funktioniert ohne Internetverbindung
- **Einfach**: Installation und Nutzung sehr unkompliziert
- **Open Source**: Transparenz und Community-getrieben

## Limitierungen

- **Performance**: Langsamer als Cloud-APIs (abhängig von Hardware)
- **Modellgröße**: Große Modelle (70B+) benötigen viel RAM
- **Qualität**: 8B-Modelle erreichen nicht die Qualität von GPT-4 oder Claude
- **Kontextlänge**: Begrenzt (meist 4k-32k Tokens)
- **Multimodalität**: Wenige Modelle unterstützen Bilder (z.B. `llava`)

## Häufige Fehler

### "Model not found"
- **Ursache**: Modell-Tag in Dify stimmt nicht mit Ollama überein
- **Lösung**: Mit `ollama list` exakten Namen prüfen (inkl. `:tag`)

### "Connection refused"
- **Ursache**: Ollama läuft nicht oder Docker kann Host nicht erreichen
- **Lösung**: Ollama-Icon in Taskleiste prüfen, `host.docker.internal` verwenden

### Langsame Antworten
- **Ursache**: Keine GPU, zu wenig RAM, großes Modell
- **Lösung**: Kleineres Modell (`8b` statt `70b`), GPU-Treiber installieren

## Weiterführende Links

- **Offizielle Website**: [https://ollama.com](https://ollama.com)
- **Dokumentation**: [https://docs.ollama.com](https://docs.ollama.com)
- **Modell-Bibliothek**: [https://ollama.com/library](https://ollama.com/library)
- **GitHub**: [https://github.com/ollama/ollama](https://github.com/ollama/ollama)

---

**Quellen**: Eigene Zusammenstellung basierend auf Ollama-Dokumentation und Projekt-Setup