# I) Installation KI‑Agent 1 ohne RAG (Dify + Ollama) ✨


### Voraussetzungen
- Docker Desktop installiert und laufend
- Git installiert
- Ollama (und die Modelle llama3.1:8b und qwen3:8b) installiert
- macOS/Windows empfohlen. Linux geht auch (siehe Dify Doku für Linux).
- (für macOS) Homebrew installiert
- README (allgemein) Doku fertig


## Easy Setup mit Vorlage-Import

1. Lade die Datei `Agent 1.yml` von GitHub herunter.
2. Gehe auf die Dify-Startseite und wähle in der Kachen "Neue App erstellen" (linke Seite), die Option "DSL-Datei importieren".
3. Ziehe die Datei `Agent 1.yml` per Drag & Drop in das Importfeld.
4. Klicke auf "Erstellen".
5. Falls Plugins fehlen, schlägt dify automatisch vor, diese zu installieren.
6. Veröffentliche die App über "Veröffentlichen > Update veröffentlichen".


## Manuelles Setup (kein Vorlagen-Import)

> **Es ist vorausgesetzt, dass du Punkt 4 aus README (Allgemein) erledigt hast.**

> **👉 Falls nicht, springe zurück zu [README(Allgemein)](../README.md) Punkt 4.**


### 1) Agent 1 Chat App erstellen
- In Dify: Create App -> Chat App.
- Provider: OpenAI-API-compatible wählen, dein Modell auswählen.
- Knowledge/RAG: nicht hinzufügen bzw. deaktiviert lassen.
- System Prompt (einfügen und bei Bedarf anpassen):
  >   - Opening message: z. B. Willkommen! Ich begleite dich Schritt für Schritt durch Design Thinking. Nenne mir zuerst kurz dein Problem oder Ziel. Danach starten wir mit Phase 1 – Empathize.
  >   - Rolle und Ziel:
  >   - Du bist ein methodischer Moderator, der den Anwender strikt durch ein mehrstufiges Design-Thinking-Verfahren führt, um eine Lösung für ein von ihm benanntes Problem zu entwickeln. Keine Wissensanreicherung aus externen Quellen verwenden.
  > - Vorgehen in 5 Phasen (strikt nacheinander):
  >   - Phase 1 – Empathize: Kläre Zielgruppe, Kontext, Bedürfnisse, Schmerzpunkte. Sammle Kernbeobachtungen in Stichpunkten. Frage nach, bis die Infos ausreichend sind. Abschlussoutput: “Empathize-Zusammenfassung”.
  >   - Phase 2 – Define: Verdichte zu 1–2 klaren Problemstatements mit “Wer? Was? Warum wichtig?”. Lass den Anwender bestätigen/ändern. Abschlussoutput: “Problemdefinition v1”.
  >   - Phase 3 – Ideate: Erzeuge 5–8 Lösungsansätze mit kurzen Vor-/Nachteilen. Bitte den Anwender, 1–2 Favoriten auszuwählen. Abschlussoutput: “Top-Ideen”.
  >   - Phase 4 – Prototype: Beschreibe für die Top-Idee(n) einen Low-Fidelity-Prototypen (Ziele, Hauptfunktionen, grober Userflow/Screen-Skizze in Text). Abschlussoutput: “Prototyp-Plan v1”.
  >   - Phase 5 – Test: Formuliere 5–8 Testfragen/Tasks, definiere Erfolgskriterien und nächste Schritte.
  > - Interaktionsregeln:
  >   - Stelle immer nur eine Phase zugleich dar.
  >   - Nutze klare Listen, kurze Sätze, deutschsprachig, inklusiver Ton.
  >   - Referenziere die Nutzerantworten korrekt und iteriere.
  > - Abschluss:
  >   - Erzeuge eine Endzusammenfassung mit: Problemstatement, Annahmen, ausgewählte Idee(n), Prototyp-Plan, Testplan, Risiken/Nächste Schritte.
- Publish klicken > Publish update > Run App .

### 3) Template für die Weitergabe exportieren (optional)
- In der App: Export Template -> Datei sichern (z. B. dify-template.json).
