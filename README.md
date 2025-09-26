# README (Allgemein) - KI-Lanparty
Willkommen zur KI-Lanparty Laborumgebung! 👋 

Dieses Repository führt Sie in die Einrichtung einer sofort einsatzbereiten Demo-Umgebung für eine KI-Lanparty mit drei Prototypen: Zwei KI-Agenten basierend auf einem Low-Code-Framework und einen On-Edge-Sprachassistenten. 

Die Software läuft lokal auf den Teilnehmerrechnern bzw. bereitgestellter Hardware. Das Setup liefert klare Templates und Anleitunen, damit Teilnehmer selbst Experimente durchführen können.

### Kurzbeschreibung der verwendeten Software

- Docker: Docker ist eine Container-Plattform, die Anwendungen inklusive ihrer Abhängigkeiten in isolierten Containern verpackt.

- Dify: Low-Code-Framework zur Erstellung KI-gesteuerter Prototypen; liefert Vorlagen, Systemprompts und einfache Integrationen für KI-Agenten.
  
- Open WebUI: Grafische Oberfläche zur Interaktion mit lokal laufenden Modellen; ermöglicht einfachen Modellwechsel und -anpassungen.

- Ollama: Lokaler Runner zur Bereitstellung von Modellen (z. B. llama3:8b, qwen3:8b); ermöglicht die direkte Ausführung von KI-Modellen ohne Cloud-Anbindung.

## Setup

### 1) Ollama installieren und Modell vorbereiten
- macOS: `brew install ollama` / Installer von ollama.com
- Windows: Installer von ollama.com
- Linux: curl-Install gemäß ollama.com

#### 1.1) Sprachmodelle über Terminal ziehen
  - via Ollama App UI: folge Step 1.2
  - via Terminal Befehl: `ollama pull llama3.1:8b` und danach`ollama pull qwen3:8b`
    - Testen: `ollama list` (zeigt installierte Modelle)

#### 1.2) Sprachmodelle über Ollama UI ziehen

- Starte Ollama
- Suche die gewünschten Modelle (`llama3:8b` und `qwen3:8b`):
  
  <img width="470" height="351" alt="image" src="https://github.com/user-attachments/assets/a4fcb0c1-0ffa-49a9-845c-a9024a7b1e02" />

- Tippe eine Nachricht in den Chat, um den Download zu starten

  <img width="454" height="336" alt="image" src="https://github.com/user-attachments/assets/153270f0-f50a-4bdb-81db-0fca22239f6a" />

- Lade beide Modelle runter (`llama3.1:8b` und `qwen3:8b`)
  
### 3) Dify lokal starten
- Repository klonen:
  - `git clone https://github.com/langgenius/dify`
- In den Ordner wechseln, der die docker-compose.yaml enthält (siehe Dify‑README).
- neue .env Datei anlegen:
  - Kopiere .env.example Inhalt nach .env
- Start:
  - `docker compose up -d`
- UI öffnen:
  - http://localhost:3000 oder `localhost:3000` als URL eingeben und Enter drücken
- Admin‑Konto anlegen.

> ### Stop here if you want to work with templates (easier and faster) and go to Agent 1 README.
> ### If you want to learn how to do it yourself, follow the next steps below (more complicated and takes longer)

### 4) Dify mit Ollama verbinden (OpenAI-API-kompatibel) 🔌
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


> ### Good job! You finished the initial setup.
> ### You can move on to the Agent_1 README now. 
> Or, if you're still a bit unsure, you can read the Usage / FAQ Sections below.


## Usage and Configuration

### Chat Format
- The AI Agent works as a Chat format, meaning every new chat is like a new conversation. You can activate the 'memory' features in the settings so that the Agent remembers things from other chats and conversations, but this uses more resources - and sometimes you might just want to start fresh every time. 

### Multiple Agents
- Dify gives you the opportunity to create multiple Agents - meaning that, if you have different use cases, you can create a dedicated Agent for each use case. You can create Agents from the main Dify page (click on the Dify logo).

### How to write prompts (Prompt engineering)
- You can change the prompt however you like. The current prompt is just an example. Your AI Agent could be a lawyer, teacher, strategist, or anything else. Just keep in mind that, the more detailed context you give it, the better and more accurate it will be. Imagine this is a person living far away that doesn't know anything about you or what you're trying to do. And you're chatting online with this person (who is an expert on many things), so you need to give them a detailed background of the problem.

### Placeholder template prompt
- If you need a placeholder template as a starter prompt, you can use the AUTOMAT framework, but there are many you can find online. Here you will find some basic ones: https://medium.com/the-generator/the-perfect-prompt-prompt-engineering-cheat-sheet-d0b9c62a2bba

### Variables
- You can add variables for the prompt. Inside the prompt text, just add two curly brackets with a variable name inbetween (`{{name}}`) and then add it as a variable in the Dify menu below the prompt text. Depending on how you included the variable inside your prompt, the Agent will use that variable accordingly. A simple example you can test is telling it to call you by your name. (`Call me by my name {{name}}`).

### Uploading documents
- It is possible to upload documents that the AI Agent will later use as an information source. This would be useful, for example, if you had a large PDF file with lots of information about a certain topic. You could upload it and ask the AI specific questions about the topic. Or summarize the whole document. Or improve the spelling and grammar. There are many use cases.
- To do this, you have to go to the Main Menu Knowledge Tab (top bar) > Create Knowledge > Import from file > Select/Drag&Drop your file > Next > High Quality > Save & Process. After you upload it there, you can go to your chat in studio view, Knowledge Dropdown on the left side (not top bar), and just select the document you uploaded earlier. Now, the AI Agent will know everything inside that document.

### Other features
- There are many other possible features that take a bit more time to set up (e.g. Vision feature to analyse uploaded Images). If you have some experience and have the extra time, you can try setting them up. 

### Limitations
- There are limitations when it comes to the amount of information the AI Agent can process. If you chat to the Agent for long enough, it will start to forget what you talked about in the beginning, and might get confused halfway through. Also, if you upload documents for the AI to use as a source of information, and the documents are too large, the AI Agent might also forget things or lose context. You can learn the exact limitations via trial and error.
- The AI Agents are not perfect and they often hallucinate. So you definitely need to double check the information you get from them, since they will confidently give you wrong answers.


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
