## Installation und Setup der Software und integrieren der Sprachmodelle:
Willkommen zur KI-Lanparty Laborumgebung. Dieses Repository führt Sie in die Einrichtung einer sofort einsatzbereiten Demo-Umgebung für eine KI-Lanparty mit drei Prototypen: Zwei KI-Agenten basierend auf einem Low-Code-Framework und einen On-Edge-Sprachassistenten. Die Software läuft lokal auf den Teilnehmerrechnern bzw. bereitgestellter Hardware. Das Setup liefert klare Templates und eine Schritt-für-Schritt-Anleitung, damit Teilnehmer eigene Experimente durchführen und Anpassungen einfach vornehmen können. Die Bereitstellung der Ressourcen für Modelle, Vorlagen und Dokumentation sind in diesem Repository zu finden.

Kurzbeschreibung der verwendeten Software

- Docker: Docker ist eine Container-Plattform, die Anwendungen inklusive ihrer Abhängigkeiten in isolierten Containern verpackt.

- Dify: Low-Code-Framework zur Erstellung KI-gesteuerter Prototypen; liefert Vorlagen, Systemprompts und einfache Integrationen für KI-Agenten.
  
- Open WebUI: Grafische Oberfläche zur Interaktion mit lokal laufenden Modellen; ermöglicht einfachen Modellwechsel und -anpassungen.

- Ollama: Lokaler Runner zur Bereitstellung von Modellen (z. B. llama3:8b, qwen3:8b); ermöglicht die direkte Ausführung von KI-Modellen ohne Cloud-Anbindung.

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
Im Dify‑UI: 

- Settings:

<img width="911" height="1096" alt="image" src="https://github.com/user-attachments/assets/520be460-85f1-4ed1-aedc-40ec4ae99a15" />

- Modellanbieter -> Nach "OpenAI-API-compatible" suchen -> Modell hinzufügen

<img width="1129" height="885" alt="image" src="https://github.com/user-attachments/assets/7d57814e-bad2-4b93-8587-86903e9ffa37" />


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

<img width="773" height="1293" alt="image" src="https://github.com/user-attachments/assets/81869b51-e41e-4480-8793-747bfae8fbd3" />


Alternative Modellwerte (wenn du Qwen nutzt):
- Model Name: qwen2.5:7b-instruct
- model name for API endpoint: qwen2.5:7b-instruct


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
