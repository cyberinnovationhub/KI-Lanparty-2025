# I) Installation Lokaler KI‑Agent ohne RAG (Dify + Ollama) ✨

### Ziel
- In 60–90 Minuten einen lokalen KI‑Agenten (ohne RAG) mit Dify und einem lokalen LLM (Ollama) aufsetzen.
- Keine Azure, keine Embeddings, kein RAG. Nur ein klarer, mehrstufiger Prozess (Design Thinking) mit vorgefertigtem Systemprompt.
- Am Ende kannst du die Dify‑App als Template exportieren und weitergeben.

### Voraussetzungen
- Docker Desktop installiert und laufend
- Git installiert
- Ollama installiert und laufend
- macOS/Windows empfohlen. Linux geht auch (siehe Hinweise zu host.docker.internal).
- (für macOS) Homebrew installiert


## Schnellstart (TL;DR) 🌱

1) Ollama installieren und ein Modell ziehen:
- `ollama pull llama3.1:8b`
2) Dify Repo klonen und starten:
- `git clone https://github.com/langgenius/dify`
- In den Ordner mit der <ins>docker-compose.yaml</ins> wechseln (siehe Dify README).
- .env.example nach .env kopieren.
- `docker compose up -d`
3) Dify öffnen: http://localhost und Admin anlegen.
4) In Dify unter Settings -> Model Providers -> OpenAI-API-compatible ein neues Modell hinzufügen:
- Model Type: LLM
- API endpoint URL:
  - macOS/Windows: http://host.docker.internal:11434/v1
  - Linux: http://172.17.0.1:11434/v1 (oder Docker-Bridge-Gateway)
- API Key: ollama (beliebiger String)
- Model Name und model name for API endpoint: exakt der Ollama‑Tag, z. B. llama3.1:8b
5) Chat App erstellen, RAG/Knowledge aus lassen, Systemprompt einfügen, veröffentlichen, testen.


## Schritt für Schritt 🍀

1) Ollama installieren und Modell vorbereiten
- macOS: `brew install ollama`
- Windows: Installer von ollama.com
- Linux: curl-Install gemäß ollama.com
- Service starten (läuft meist automatisch nach Installation).
- Modell ziehen:
  - `ollama pull llama3.1:8b`
  - Alternativ: `ollama pull qwen2.5:7b-instruct`
- Testen:
  - `ollama list` (zeigt installierte Modelle)
  - Optional: einfacher API‑Test
    - `curl http://localhost:11434/v1/chat/completions -H "Content-Type: application/json" -H "Authorization: Bearer ollama" -d '{"model":"llama3.1:8b","messages":[{"role":"user","content":"Hallo"}]}'`

2) Dify lokal starten
- Repository klonen:
  - `git clone https://github.com/langgenius/dify`
- In den Ordner wechseln, der die docker-compose.yaml enthält (siehe Dify‑README).
- .env anlegen:
  - Kopiere .env.example nach .env
- Start:
  - `docker compose up -d`
- UI öffnen:
  - http://localhost:3000
- Admin‑Konto anlegen.

3) Dify mit Ollama verbinden (OpenAI-API-kompatibel) 🔌
Im Dify‑UI: Settings -> Model Providers -> OpenAI-API-compatible -> Add model

Felder ausfüllen (Beispiel: llama3.1:8b):
- Model Type: LLM
- Model Name: llama3.1:8b
- Authorization Name: Authorization
- Model display name: Llama 3.1 8B (Ollama)  [frei wählbar]
- API Key: ollama  [beliebiger String, wird von Ollama nicht geprüft]
- API endpoint URL:
  - macOS/Windows: http://host.docker.internal:11434/v1
  - Linux: http://172.17.0.1:11434/v1 (oder das Gateway aus docker network inspect bridge)
- model name for API endpoint: llama3.1:8b  [muss exakt zum Ollama‑Tag passen]

Alternative Modellwerte (wenn du Qwen nutzt):
- Model Name: qwen2.5:7b-instruct
- model name for API endpoint: qwen2.5:7b-instruct

4) Chat App erstellen (ohne RAG)
- In Dify: Create App -> Chat App.
- Provider: OpenAI-API-compatible wählen, dein Modell auswählen.
- Knowledge/RAG: nicht hinzufügen bzw. deaktiviert lassen.
- Opening message: z. B. Willkommen! Ich begleite dich Schritt für Schritt durch Design Thinking. Nenne mir zuerst kurz dein Problem oder Ziel. Danach starten wir mit Phase 1 – Empathize.
- System Prompt (einfügen und bei Bedarf anpassen):
  >   - Rolle und Ziel:
  >   - Du bist ein methodischer Moderator, der den Anwender strikt durch ein mehrstufiges Design-Thinking-Verfahren führt, um eine Lösung für ein von ihm benanntes Problem zu entwickeln. Keine Wissensanreicherung aus externen Quellen verwenden.
  > - Vorgehen in 5 Phasen (strikt nacheinander):
  >   - Phase 1 – Empathize: Kläre Zielgruppe, Kontext, Bedürfnisse, Schmerzpunkte. Sammle Kernbeobachtungen in Stichpunkten. Frage nach, bis die Infos ausreichend sind. Abschlussoutput: “Empathize-Zusammenfassung”.
  >   - Phase 2 – Define: Verdichte zu 1–2 klaren Problemstatements mit “Wer? Was? Warum wichtig?”. Lass den Anwender bestätigen/ändern. Abschlussoutput: “Problemdefinition v1”.
  >   - Phase 3 – Ideate: Erzeuge 5–8 Lösungsansätze mit kurzen Vor-/Nachteilen. Bitte den Anwender, 1–2 Favoriten auszuwählen. Abschlussoutput: “Top-Ideen”.
  >   - Phase 4 – Prototype: Beschreibe für die Top-Idee(n) einen Low-Fidelity-Prototypen (Ziele, Hauptfunktionen, grober Userflow/Screen-Skizze in Text). Abschlussoutput: “Prototyp-Plan v1”.
  >   - Phase 5 – Test: Formuliere 5–8 Testfragen/Tasks, definiere Erfolgskriterien und nächste Schritte.
  > - Interaktionsregeln:
  >   - Stelle immer nur eine Phase zugleich dar und frage explizit “Weiter mit Phase X?”.
  >   - Nutze klare Listen, kurze Sätze, deutschsprachig, inklusiver Ton.
  >   - Referenziere die Nutzerantworten korrekt und iteriere.
  > - Abschluss:
  >   - Erzeuge eine Endzusammenfassung mit: Problemstatement, Annahmen, ausgewählte Idee(n), Prototyp-Plan, Testplan, Risiken/Nächste Schritte.
- Publish und testen.

5) Template exportieren (für die Weitergabe)
- In der App: Export Template -> Datei sichern (z. B. dify-template.json).
- README und Template ins Repo/Ordner legen – fertig. 🌟


## FAQ & Troubleshooting 🧩

- Warum funktioniert die URL http://host.docker.internal:11434/v1? 🔍
  - Dify läuft in Docker‑Containern. Damit ein Container (Dify) den Ollama‑Dienst auf deinem Host‑Rechner erreicht, gibt es host.docker.internal (auf macOS/Windows). Das ist eine spezielle Hostname‑Brücke vom Container zum Host.
  - Der /v1‑Pfad: Ollama bietet eine OpenAI-kompatible API unter dem Pfad /v1. Dify erwartet genau dieses Schema. Deshalb muss die Base‑URL auf /v1 enden.
  - Linux‑Sonderfall: host.docker.internal ist dort oft nicht vordefiniert. Nimm stattdessen die Gateway‑IP der Docker‑Bridge (häufig 172.17.0.1). Du findest sie mit: docker network inspect bridge und suchst nach "Gateway".

- Es gibt keinen “Chat”‑Typ bei “Model Type”.
  - Korrekt. In Dify heißt das einfach “LLM”. Das ist der Chat/Text‑Generierungstyp.

- 404 Not Found bei API‑Aufruf.
  - Meist fehlt /v1 am Ende der Base‑URL. Nutze z. B. http://host.docker.internal:11434/v1

- Verbindung fehlgeschlagen von Dify zu Ollama.
  - macOS/Windows: Nutze host.docker.internal, nicht localhost.
  - Linux: Nutze http://172.17.0.1:11434/v1 oder deinen Bridge‑Gateway‑Host (docker network inspect bridge).
  - Prüfe, ob Ollama läuft: ollama list

- “Model not found” oder “The model does not exist”.
  - Der Eintrag “model name for API endpoint” muss exakt dem Ollama‑Tag entsprechen (z. B. llama3.1:8b). Prüfe mit ollama list oder curl http://localhost:11434/api/tags

- Was trage ich bei “API Key” ein?
  - Irgendeinen nicht‑leeren String (z. B. ollama). Ollama prüft das nicht, Dify will aber ein Feld.

- Ist .yml und .yaml dasselbe?
  - Ja. Beides ist YAML. Docker Compose akzeptiert beide. Falls der Dateiname abweicht, nutze docker compose -f datei.yaml up -d

- Responses sind langsam.
  - Kleinere Modelle nutzen (z. B. qwen2.5:7b-instruct).
  - In Dify die Max‑Token reduzieren.
  - Genug RAM/CPU sicherstellen.

- Kann ich ein anderes Modell nutzen?
  - Ja. In Ollama den passenden Tag ziehen (ollama pull ...), danach in Dify dasselbe Vorgehen: Model Name und model name for API endpoint identisch zum Tag.

- “localhost” funktioniert nicht aus Dify.
  - Dify läuft im Container. localhost meint dann den Container selbst, nicht deinen Host. Daher host.docker.internal (Mac/Win) bzw. Bridge‑Gateway (Linux).

- Linux: Wie finde ich die Gateway‑IP?
  - docker network inspect bridge und nach "Gateway" suchen. Typisch ist 172.17.0.1

- Port‑Konflikte beim Start von Dify.
  - Prüfe, ob 3000 (Dify UI) oder weitere in der Compose belegte Ports frei sind. Passe Ports in der docker-compose.yaml an, falls nötig, und starte neu.

- Ich habe versehentlich Knowledge/RAG aktiviert.
  - In der App die Knowledge‑Sektion entfernen bzw. sicherstellen, dass kein Datenspeicher verbunden ist. Für Agent 1 brauchen wir kein RAG.

- Wie exportiere/importiere ich das Template?
  - In der App: Export Template -> Datei sichern. Import analog über Import Template.
