# RAG - Retrieval-Augmented Generation

## Was ist RAG?

Retrieval-Augmented Generation (RAG) ist eine Technik, die Large Language Models (LLMs) mit externen Wissensquellen kombiniert. Anstatt sich nur auf das Trainingswissen des Modells zu verlassen, durchsucht ein RAG-System eine Wissensdatenbank und fügt relevante Informationen in den Prompt ein. Das Ergebnis: Faktenbasierte, aktuelle Antworten mit reduzierten Halluzinationen.

**Metapher**: Ein LLM ohne RAG ist wie ein Student, der nur aus dem Gedächtnis antwortet. Mit RAG darf der Student Notizen und Bücher verwenden.

## Wofür wird es im Projekt verwendet?

RAG wird bei **Agent 2** (Dokumentations-Experte) eingesetzt:

- **Wissensbasis**: Zugriff auf README-Dateien und Projekt-Dokumentationen
- **Kontextuelle Antworten**: Beantwortet Fragen basierend auf hochgeladenen Dokumenten
- **Quellenangaben**: Kann auf spezifische Dokumentenabschnitte verweisen
- **Kein Retraining**: Neue Dokumente können ohne Modell-Update hinzugefügt werden

**Ohne RAG** (Agent 1): Modell nutzt nur allgemeines Trainingswissen
**Mit RAG** (Agent 2): Modell hat Zugriff auf projektspezifische Dokumentation

## Kernkonzepte

### 1. Der RAG-Workflow (5 Schritte)

```
1. INDEXIERUNG (einmalig)
   Dokumente → Chunking → Embeddings → Vector DB

2. ANFRAGE (bei jeder User-Frage)
   User-Frage → Embedding → Suche in Vector DB

3. RETRIEVAL
   Top-K ähnlichste Dokument-Chunks abrufen

4. AUGMENTATION
   Prompt = System Prompt + Retrieved Chunks + User-Frage

5. GENERATION
   LLM generiert Antwort basierend auf erweitertem Prompt
```

**Beispiel**:
```
User: "Wie verbinde ich Dify mit Ollama?"

→ RAG sucht in Dokumentation und findet:
  - README.md: "Dify mit Ollama verbinden"
  - Agent_1/README.md: "Model Provider: OpenAI-API-compatible"

→ Prompt wird erweitert:
  [System: Du bist ein Dokumentations-Experte...]
  [Context: "Dify mit Ollama verbinden: Settings → Model Providers..."]
  [User: "Wie verbinde ich Dify mit Ollama?"]

→ LLM generiert präzise Antwort mit Schritt-für-Schritt-Anleitung
```

### 2. Chunking - Dokumente aufteilen

Dokumente werden in kleinere Einheiten ("Chunks") zerlegt:

**Strategien**:
- **Fixed-Size**: Feste Anzahl Tokens (z.B. 512)
- **Sentence-Based**: Nach Satzgrenzen
- **Semantic**: Nach semantischen Einheiten (Absätze, Kapitel)

**Im Projekt**: Dify nutzt automatisches Chunking mit ~500 Tokens pro Chunk.

**Warum Chunking?**
- Embeddings funktionieren besser mit fokussierten Textausschnitten
- Kontextlänge von LLMs ist begrenzt
- Präzisere Suche nach relevanten Informationen

### 3. Embeddings - Text in Zahlen

Embeddings sind numerische Repräsentationen von Text (Vektoren):

```
"Dify ist ein Low-Code-Framework"
→ [0.23, -0.12, 0.56, ..., 0.89]  (1024 Dimensionen)

"Dify ist eine AI-Plattform"
→ [0.21, -0.15, 0.52, ..., 0.91]  (ähnlicher Vektor!)
```

**Eigenschaften**:
- Semantisch ähnliche Texte haben ähnliche Vektoren
- Gemessen mit **Cosine Similarity** (0 = unterschiedlich, 1 = identisch)
- Modell: `nomic-embed-text` (1024 Dimensionen)

### 4. Vector Database - Wissens-Speicher

Eine Vector Database speichert Embeddings und ermöglicht schnelle Ähnlichkeitssuche:

**Im Projekt**:
- **Dify Economical Mode**: Einfache Keyword-Suche (kein Embedding)
- **Dify High Quality Mode**: pgvector (PostgreSQL-Erweiterung)
- **Alternativen**: ChromaDB, Qdrant, Pinecone, Weaviate

**Ablauf**:
1. Dokumente werden als Embeddings gespeichert
2. User-Frage wird in Embedding umgewandelt
3. Datenbank findet Top-K ähnlichste Embeddings
4. Zugehörige Text-Chunks werden zurückgegeben

### 5. Retrieval-Strategien

**Hybrid Search** (empfohlen):
- Kombiniert semantische Suche (Embeddings) mit Keyword-Suche
- **Beispiel**: "Docker Port 3000" → findet sowohl semantisch ähnliche als auch exakte Matches

**Reranking**:
- Zusätzliches Modell bewertet Relevanz der abgerufenen Chunks
- Verbessert Präzision, aber langsamer

**Metadata Filtering**:
- Filter nach Dokumenttyp, Datum, Tags
- **Beispiel**: "Nur Agent_2 Dokumentation durchsuchen"

### 6. Economical vs. High Quality Mode (Dify)

| Aspekt | Economical | High Quality |
|--------|-----------|--------------|
| **Suche** | Keyword-basiert (BM25) | Semantische Suche |
| **Embedding** | ❌ Nicht nötig | ✅ `nomic-embed-text` |
| **Qualität** | Gut für exakte Begriffe | Besser für Konzepte |
| **Setup** | Einfach | Embedding-Modell nötig |
| **Performance** | Schneller | Langsamer |

**Beispiel**:
- **Economical**: "Docker Container" findet Texte mit exakten Wörtern "Docker" und "Container"
- **High Quality**: "Wie isoliere ich Anwendungen?" findet Docker-Dokumentation (semantisch ähnlich)

## Typische RAG-Workflows

### Knowledge Base in Dify erstellen

1. **Knowledge** → **Create Knowledge**
2. Dokumente hochladen (Drag & Drop):
   - Markdown (.md)
   - PDF (.pdf)
   - Text (.txt)
   - Word (.docx)
3. **Index Mode** wählen:
   - **Economical**: Für Anfänger, kein Setup
   - **High Quality**: Für bessere Ergebnisse (benötigt Embedding-Modell)
4. **Save & Process** → Warten auf Verarbeitung

### Knowledge mit Agent verknüpfen

1. Agent öffnen → **Context**
2. **Add** → Knowledge auswählen
3. **Retrieval Settings** (optional):
   - **Top K**: Anzahl abgerufener Chunks (Standard: 3-5)
   - **Score Threshold**: Minimale Ähnlichkeit (0.0 - 1.0)
4. **Publish** → Testen

### Retrieval testen und optimieren

**Test-Frage stellen**:
```
User: "Wie starte ich Dify?"
```

**Erwartung**: Agent zitiert aus README.md → "docker compose up -d"

**Wenn Antwort ungenau**:
- **Top K erhöhen**: Mehr Kontext (aber längerer Prompt)
- **Chunking anpassen**: Kleinere/größere Chunks
- **High Quality Mode**: Semantische Suche aktivieren
- **Dokumente verbessern**: Klarere Struktur, weniger Redundanz

## 2025 Best Practices

### 1. Hybrid Search bevorzugen
Kombination aus semantischer und Keyword-Suche für beste Ergebnisse.

### 2. Query Augmentation
User-Frage umformulieren oder erweitern:
- **Original**: "Port-Problem"
- **Erweitert**: "Wie löse ich Port-Konflikte zwischen Dify und Open WebUI?"

### 3. Iterative Fine-Tuning
- Logs analysieren: Welche Fragen führen zu schlechten Antworten?
- Dokumente ergänzen/umstrukturieren
- Chunking-Strategie anpassen

### 4. Metadaten nutzen
- **Tags**: `[Agent_1]`, `[Setup]`, `[Troubleshooting]`
- **Datum**: Nur aktuelle Versionen durchsuchen
- **Typ**: Tutorials vs. API-Referenz

### 5. Quellenangaben
LLM anweisen, Quellen zu nennen:
```
"Beantworte die Frage und nenne die Quelle (Dateiname + Abschnitt)."
```

## Vorteile von RAG

- **Aktualität**: Wissen aktualisieren ohne Retraining
- **Faktentreue**: Reduziert Halluzinationen deutlich
- **Transparenz**: Quellen nachvollziehbar
- **Spezialisierung**: Modell wird Experte für spezifisches Wissensgebiet
- **Kosteneffizienz**: Günstiger als Fine-Tuning

## Limitierungen

- **Retrieval-Qualität**: Schlechte Suche → schlechte Antworten
- **Kontextlänge**: Zu viele Chunks → LLM verwirrt
- **Latenz**: Zusätzliche Suche verlängert Antwortzeit
- **Chunk-Granularität**: Zu klein = Kontext fehlt, zu groß = irrelevante Infos
- **Multimodale Inhalte**: Bilder, Tabellen schwieriger zu verarbeiten

## Häufige Fehler

### Agent gibt generische Antworten (ignoriert Dokumente)
**Ursache**: Retrieval funktioniert nicht oder Score Threshold zu hoch
**Lösung**:
- **Context** prüfen: Sind Dokumente verknüpft?
- **Top K erhöhen**: z.B. von 3 auf 5
- **Score Threshold senken**: z.B. von 0.7 auf 0.5

### Agent zitiert irrelevante Dokumententeile
**Ursache**: Chunking zu grob oder schlechte Embeddings
**Lösung**:
- **High Quality Mode** aktivieren
- Dokumente besser strukturieren (Überschriften, Absätze)
- Kleinere Chunk-Größe

### "No relevant documents found"
**Ursache**: Dokumente nicht indexiert oder Query zu spezifisch
**Lösung**:
- **Knowledge** → Prüfen ob Dokumente "Completed" Status haben
- Query umformulieren oder erweitern
- Metadata-Filter prüfen

## Erweiterte RAG-Varianten (2025)

### Self-RAG
Modell entscheidet selbst, wann Retrieval nötig ist (nicht bei jeder Anfrage).

### Corrective RAG
Bei ungenauen Retrieval-Ergebnissen: Websuche oder andere Fallback-Quellen.

### GraphRAG (Microsoft)
Nutzt Knowledge Graphs statt reinem Text für komplexe Zusammenhänge.

### Adaptive RAG
Passt Retrieval-Strategie dynamisch an Query-Komplexität an.

**Im Projekt**: Klassisches RAG (ausreichend für Dokumentations-Use-Case)

## Weiterführende Links

- **LangChain RAG Tutorial**: [https://python.langchain.com/docs/tutorials/rag/](https://python.langchain.com/docs/tutorials/rag/)
- **Dify Knowledge Base Docs**: [https://docs.dify.ai/features/knowledge-base](https://docs.dify.ai/features/knowledge-base)
- **2025 RAG Guide (EdenAI)**: [https://www.edenai.co/post/the-2025-guide-to-retrieval-augmented-generation-rag](https://www.edenai.co/post/the-2025-guide-to-retrieval-augmented-generation-rag)
- **AWS RAG Explanation**: [https://aws.amazon.com/what-is/retrieval-augmented-generation/](https://aws.amazon.com/what-is/retrieval-augmented-generation/)

---

**Quellen**: Eigene Zusammenstellung basierend auf RAG-Research 2025 und Projekt-Implementierung