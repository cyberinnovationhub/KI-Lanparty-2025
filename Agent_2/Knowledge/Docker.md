# Docker - Container-Plattform

## Was ist Docker?

Docker ist eine Plattform zur Containerisierung von Anwendungen. Ein Container ist wie eine isolierte Mini-Umgebung, die eine Anwendung inklusive aller Abhängigkeiten (Betriebssystem, Bibliotheken, Konfiguration) enthält. Docker ermöglicht es, Software konsistent auf verschiedenen Systemen zu betreiben, ohne dass Abhängigkeiten manuell installiert werden müssen.

**Metapher**: Ein Docker-Container ist wie ein Schiffscontainer – standardisiert, portabel und isoliert vom Rest des Systems.

## Wofür wird es im Projekt verwendet?

Docker wird genutzt, um **Dify** und **Open WebUI** zu betreiben:

- **Dify**: Läuft in mehreren Docker-Containern (API, Worker, Datenbank, Redis)
- **Open WebUI**: Läuft in einem einzelnen Container für Agent 3
- **Isolation**: Beide Anwendungen laufen unabhängig vom Host-System
- **Einfaches Setup**: `docker compose up` startet alle benötigten Services automatisch
- **Konsistenz**: Funktioniert identisch auf Windows, macOS und Linux

## Kernkonzepte

### 1. Images vs. Container

- **Image**: Unveränderbare Vorlage (wie ein Rezept oder Blaupause)
  - Beispiel: `ghcr.io/open-webui/open-webui:main`
  - Wird einmal heruntergeladen und kann mehrfach verwendet werden

- **Container**: Laufende Instanz eines Images (wie ein gestartetes Programm)
  - Beispiel: Der gestartete `open-webui` Container
  - Kann gestartet, gestoppt und gelöscht werden

**Workflow**: Image pullen → Container erstellen → Container starten

### 2. Docker Compose

Docker Compose verwaltet Multi-Container-Anwendungen über eine YAML-Konfigurationsdatei (`docker-compose.yaml`):

```yaml
services:
  api:
    image: langgenius/dify-api:latest
    ports:
      - "5001:5001"
    environment:
      - DATABASE_URL=...

  web:
    image: langgenius/dify-web:latest
    ports:
      - "3000:3000"
```

**Befehle**:
```bash
# Alle Container starten
docker compose up -d

# Status anzeigen
docker compose ps

# Logs anzeigen
docker compose logs -f

# Container stoppen
docker compose down
```

Im Projekt wird Compose für **Dify** genutzt (mehrere Container: API, Worker, DB, Redis, nginx).

### 3. Volumes - Persistente Daten

Container sind flüchtig – beim Löschen gehen Daten verloren. Volumes speichern Daten außerhalb des Containers:

```bash
# Volume wird automatisch erstellt und gemountet
docker run -v open-webui:/app/backend/data ...
```

Im Projekt:
- **Dify**: Datenbank-Daten in Volumes (`postgres`, `redis`)
- **Open WebUI**: Chats und Settings in `open-webui` Volume

### 4. Port-Mapping

Container haben eigene Netzwerke. Port-Mapping verbindet Container-Ports mit Host-Ports:

```bash
# Format: -p HOST_PORT:CONTAINER_PORT
docker run -p 3000:8080 ...
```

- **Links**: Host-Port `3000`
- **Rechts**: Container-Port `8080`
- **Zugriff**: `http://localhost:3000` erreicht Container-Port `8080`

Im Projekt:
- **Dify Web**: `3000:3000` (Host 3000 → Container 3000)
- **Open WebUI**: `3000:8080` (Host 3000 → Container 8080)

**Konflikt**: Dify und Open WebUI können nicht gleichzeitig laufen (beide nutzen Host-Port 3000).

### 5. `host.docker.internal` - Host-Zugriff

Container laufen in isolierten Netzwerken. `host.docker.internal` ist eine spezielle Adresse, über die Container den Host-Rechner erreichen:

```bash
# Im Container: Zugriff auf Ollama (läuft auf dem Host)
http://host.docker.internal:11434
```

- **macOS/Windows**: Funktioniert automatisch
- **Linux**: Nutze stattdessen `172.17.0.1` (Docker Bridge Gateway)

### 6. Docker Desktop vs. Docker Engine

- **Docker Desktop**: GUI-Anwendung für Windows/macOS (enthält Docker Engine + GUI)
- **Docker Engine**: Nur CLI, typisch auf Linux-Servern

Im Projekt wird **Docker Desktop** verwendet (einfachere Verwaltung für Anfänger).

## Typische Workflows

### Dify starten/stoppen

```bash
# Starten (im dify/docker Verzeichnis)
cd dify/docker
docker compose up -d

# Status prüfen
docker compose ps

# Logs ansehen
docker compose logs -f api

# Stoppen
docker compose down

# Stoppen und Volumes löschen (ACHTUNG: Datenverlust!)
docker compose down -v
```

### Open WebUI starten/stoppen

```bash
# Image pullen
docker pull ghcr.io/open-webui/open-webui:main

# Container starten
docker run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main

# Container stoppen
docker stop open-webui

# Container starten (wenn bereits erstellt)
docker start open-webui

# Container löschen
docker rm open-webui
```

### Problemlösung

```bash
# Alle laufenden Container anzeigen
docker ps

# Alle Container (auch gestoppte)
docker ps -a

# Container-Logs ansehen
docker logs <container-name>

# In Container einsteigen (Debugging)
docker exec -it <container-name> /bin/bash

# Volumes anzeigen
docker volume ls

# Netzwerke anzeigen
docker network ls
```

## Vorteile

- **Konsistenz**: "Works on my machine" Probleme gelöst
- **Isolation**: Anwendungen beeinflussen sich nicht gegenseitig
- **Portabilität**: Gleicher Container läuft überall
- **Einfaches Setup**: `docker compose up` statt komplexer Installation
- **Versionierung**: Verschiedene Versionen parallel betreiben

## Limitierungen

- **Ressourcen**: Container teilen sich Host-Ressourcen (CPU, RAM)
- **Performance**: Leichter Overhead (meist vernachlässigbar)
- **Komplexität**: Networking und Volumes können anfangs verwirrend sein
- **Speicher**: Images und Volumes können viel Platz belegen

## Häufige Fehler

### Port bereits belegt
```
Error: bind: address already in use
```
**Lösung**: Anderen Container auf dem Port stoppen oder Port in `docker-compose.yaml` ändern

### Volume-Berechtigungen (Linux)
```
Error: Permission denied
```
**Lösung**: Docker mit entsprechenden User-Rechten starten oder User zur `docker` Gruppe hinzufügen

### Dify kann Ollama nicht erreichen
```
Connection refused: http://localhost:11434
```
**Lösung**: `host.docker.internal:11434` statt `localhost:11434` verwenden

### Container startet nicht
```bash
# Logs prüfen
docker logs <container-name>

# Detaillierte Infos
docker inspect <container-name>
```

## Docker Desktop Tipps

### Container-Verwaltung in der GUI
1. **Containers** Tab öffnen
2. Container-Gruppe (`docker`) auswählen
3. **Actions**-Buttons: Start/Stop/Restart/Delete

### Ressourcen-Limits einstellen
**Settings** → **Resources**:
- **CPUs**: Anzahl der CPU-Kerne für Docker
- **Memory**: RAM-Limit für alle Container
- **Disk**: Speicherplatz für Images/Volumes

### Automatischer Start
**Settings** → **General**:
- ☑ **Start Docker Desktop when you log in**
- ☐ **Open Docker Dashboard at startup** (optional)

## Weiterführende Links

- **Offizielle Dokumentation**: [https://docs.docker.com](https://docs.docker.com)
- **Docker Compose Docs**: [https://docs.docker.com/compose/](https://docs.docker.com/compose/)
- **Docker Hub**: [https://hub.docker.com](https://hub.docker.com) (Image-Registry)
- **Docker Desktop**: [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

---

**Quellen**: Eigene Zusammenstellung basierend auf Docker-Dokumentation und Projekt-Setup