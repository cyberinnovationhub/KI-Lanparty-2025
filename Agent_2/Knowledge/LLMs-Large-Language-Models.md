# LLMs - Large Language Models

## Was sind LLMs?

Large Language Models (LLMs) sind KI-Modelle, die darauf trainiert wurden, menschliche Sprache zu verstehen und zu generieren. Sie basieren auf neuronalen Netzwerken mit Milliarden von Parametern und wurden mit enormen Textmengen aus dem Internet trainiert. LLMs können Texte vervollständigen, Fragen beantworten, übersetzen, programmieren und vieles mehr.

**Bekannte Beispiele**: GPT-4 (OpenAI), Claude (Anthropic), Llama (Meta), Gemini (Google)

## Wofür werden sie im Projekt verwendet?

LLMs sind die **Kernkomponente** aller drei Agenten:

- **Agent 1**: Llama 3.1 (8B) führt Design Thinking Interviews durch
- **Agent 2**: Llama/Qwen oder GPT-4o beantwortet Fragen mit RAG-Unterstützung
- **Agent 3**: Llama 3.1 (8B) simuliert technische Bewerbungsinterviews

Die Modelle werden lokal über Ollama oder remote über Azure OpenAI bereitgestellt.

## Kernkonzepte

### 1. Parameter und Modellgröße

Die Anzahl der Parameter bestimmt die Fähigkeiten und den Ressourcenbedarf:

- **7-8B** (z.B. Llama 3.1 8B):
  - ~5 GB RAM
  - Gut für einfache Aufgaben, Konversationen
  - Schnell auf Consumer-Hardware

- **70B** (z.B. Llama 3.1 70B):
  - ~40 GB RAM
  - Deutlich bessere Qualität
  - Benötigt High-End-Hardware oder Server

- **400B+** (z.B. GPT-4):
  - Nur als Cloud-API verfügbar
  - Beste Qualität, aber kostenpflichtig

**Faustregel**: Mehr Parameter = bessere Qualität, aber langsamer und ressourcenhungriger.

### 2. Tokens

LLMs verarbeiten Text als "Tokens" – kleine Texteinheiten (ca. 4 Zeichen oder 0.75 Wörter):

```
"Hallo, wie geht es dir?" → ["Hallo", ",", " wie", " geht", " es", " dir", "?"]
≈ 7 Tokens
```

**Kontextlänge** = maximale Anzahl Tokens, die das Modell gleichzeitig "sehen" kann:
- **Llama 3.1 8B**: 128k Tokens (~96.000 Wörter)
- **GPT-4o**: 128k Tokens
- **Claude 3.5 Sonnet**: 200k Tokens

Wenn die Konversation zu lang wird, vergisst das Modell frühere Nachrichten.

### 3. Prompts

Ein **Prompt** ist die Eingabe an das LLM. Es besteht typischerweise aus:

```
[System Prompt: "Du bist ein hilfreicher Assistent..."]
[User: "Erkläre mir Docker"]
[Assistant: "Docker ist eine Plattform..."]
[User: "Und was sind Container?"]
```

**System Prompt**: Definiert Rolle und Verhalten (wird bei jeder Anfrage mitgesendet)
**User/Assistant**: Konversationsverlauf

### 4. Temperature und Sampling

Diese Parameter steuern die Kreativität der Antworten:

- **Temperature** (0.0 - 2.0):
  - **0.0**: Deterministisch, immer die wahrscheinlichste Antwort (gut für Fakten)
  - **0.7**: Ausgewogen (Standard)
  - **1.5+**: Kreativ, aber unvorhersehbar (gut für Storytelling)

- **Top-p** (0.0 - 1.0):
  - Betrachtet nur die wahrscheinlichsten Token (Nucleus Sampling)
  - **0.9**: Standard-Wert (berücksichtigt 90% der Wahrscheinlichkeitsmasse)

**Im Projekt**: Meist Default-Werte (Temperature ~0.7), da strukturierte Antworten gewünscht sind.

### 5. Inference (Inferenz)

**Inference** = Der Prozess, bei dem das Modell eine Antwort generiert:

1. **Input**: Prompt wird in Tokens umgewandelt
2. **Verarbeitung**: Neuronales Netzwerk berechnet Wahrscheinlichkeiten
3. **Sampling**: Nächstes Token wird basierend auf Temperature ausgewählt
4. **Iteration**: Schritt 2-3 wiederholen bis Antwort vollständig
5. **Output**: Tokens werden zurück in Text umgewandelt

**Geschwindigkeit**: Gemessen in Tokens/Sekunde:
- **Lokale 8B-Modelle** (CPU): 5-20 tok/s
- **Lokale 8B-Modelle** (GPU): 30-100 tok/s
- **Cloud-APIs** (GPT-4o): 50-150 tok/s

### 6. Halluzinationen

LLMs können **Halluzinationen** erzeugen – überzeugende, aber falsche Informationen:

**Beispiel**:
```
User: "Wann wurde Docker erfunden?"
LLM: "Docker wurde 2011 von Solomon Hykes entwickelt."
```
(Korrekt: 2013)

**Gegenmaßnahmen**:
- **Faktenprüfung**: Kritisch hinterfragen
- **RAG**: Externe Wissensquellen einbinden
- **Temperature senken**: Weniger kreativ = weniger Halluzinationen
- **Quellen verlangen**: "Nenne die Quelle für diese Information"

### 7. Lokale vs. Cloud-Modelle

| Aspekt | Lokal (Ollama) | Cloud (Azure/OpenAI) |
|--------|----------------|---------------------|
| **Kosten** | Kostenlos | Pay-per-Token |
| **Privacy** | Daten bleiben lokal | Daten gehen zu Anbieter |
| **Qualität** | 8B = Mittel | GPT-4 = Sehr hoch |
| **Geschwindigkeit** | CPU: Langsam, GPU: Schnell | Meist sehr schnell |
| **Verfügbarkeit** | Offline möglich | Internet erforderlich |
| **Setup** | Ollama installieren | API-Key holen |

**Im Projekt**:
- **Agents 1 & 3**: Nur lokal (Llama/Qwen)
- **Agent 2**: Hybrid (lokal + Azure GPT-4o für bessere Qualität)

## Typische Anwendungsfälle

### 1. Chatbots und Assistenten
- **Beispiel**: Agent 1 (Design Thinking), Agent 3 (Interview)
- **LLM-Rolle**: Führt strukturierte Konversationen durch

### 2. Frage-Antwort-Systeme (Q&A)
- **Beispiel**: Agent 2 (Dokumentations-Experte)
- **LLM-Rolle**: Beantwortet Fragen basierend auf Kontext (RAG)

### 3. Code-Generierung
- **Beispiel**: GitHub Copilot, Cursor
- **LLM-Rolle**: Vervollständigt Code, erklärt Funktionen

### 4. Übersetzung und Zusammenfassung
- **Beispiel**: DeepL, Notion AI
- **LLM-Rolle**: Transformiert Text in andere Form

### 5. Content-Generierung
- **Beispiel**: Blog-Posts, Marketing-Texte
- **LLM-Rolle**: Erstellt kreative Texte nach Vorgaben

## Limitierungen von LLMs

### 1. Keine echte Intelligenz
LLMs "verstehen" nicht wirklich – sie erkennen statistische Muster in Trainingsdaten.

### 2. Trainingsdaten-Cutoff
Modelle wissen nichts über Ereignisse nach ihrem Training:
- **Llama 3.1**: Training bis ~2023
- **GPT-4**: Bis 2023
- Keine automatischen Updates

### 3. Kontextlänge
Auch bei 128k Tokens kann das Modell bei sehr langen Konversationen Details vergessen.

### 4. Rechenintensiv
Große Modelle benötigen viel Energie und Rechenleistung.

### 5. Bias (Voreingenommenheit)
Modelle reproduzieren Vorurteile aus Trainingsdaten (Geschlecht, Kultur, Politik).

## Best Practices für die Arbeit mit LLMs

### 1. Klare, spezifische Prompts
❌ **Schlecht**: "Erkläre Docker"
✅ **Gut**: "Erkläre Docker in 3 Sätzen für Anfänger, die noch nie mit Containern gearbeitet haben"

### 2. System Prompts nutzen
Definiere Rolle, Tonfall und Einschränkungen im System Prompt statt in jeder Nachricht.

### 3. Iteratives Prompting
Verfeinere Prompts schrittweise basierend auf Antworten:
```
1. "Erkläre RAG"
2. "Gib ein praktisches Beispiel mit Code"
3. "Wie unterscheidet sich das von klassischer Suche?"
```

### 4. Faktenchecking
Verifiziere wichtige Informationen, besonders bei:
- Technischen Details
- Historischen Fakten
- Sicherheitskritischen Empfehlungen

### 5. Kontext bereitstellen
Je mehr relevanter Kontext, desto besser die Antwort:
- **Zielgruppe**: "Erkläre für Anfänger/Experten"
- **Format**: "Als Stichpunkte/Tutorial/Code"
- **Constraints**: "In maximal 100 Wörtern"

## Zukunft von LLMs

### Trends
- **Multimodalität**: Bilder, Audio, Video verstehen (z.B. GPT-4 Vision)
- **Reasoning**: Bessere logische Schlussfolgerungen (z.B. OpenAI o1)
- **Effizienz**: Kleinere Modelle mit gleicher Leistung
- **Agents**: LLMs, die Tools nutzen und komplexe Aufgaben lösen

### Herausforderungen
- **Energieverbrauch**: Training und Inferenz sind ressourcenintensiv
- **Regulierung**: EU AI Act und ähnliche Gesetze
- **Bias-Reduktion**: Fairere, ausgewogenere Modelle

## Weiterführende Links

- **Hugging Face Model Hub**: [https://huggingface.co/models](https://huggingface.co/models) (Durchsuchbare Modell-Datenbank)
- **Ollama Model Library**: [https://ollama.com/library](https://ollama.com/library) (Lokale Modelle)
- **LLM Leaderboards**: [https://chat.lmsys.org](https://chat.lmsys.org) (Modell-Vergleiche)
- **Prompt Engineering Guide**: [https://www.promptingguide.ai](https://www.promptingguide.ai)

---

**Quellen**: Eigene Zusammenstellung basierend auf aktueller LLM-Forschung und Projekt-Anforderungen