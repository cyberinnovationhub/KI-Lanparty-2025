# Embeddings und Vector Databases

## Was sind Embeddings?

Embeddings sind numerische Repräsentationen (Vektoren) von Text, die semantische Bedeutung erfassen. Ähnliche Texte haben ähnliche Vektoren, unabhängig von der exakten Wortwahl. Das ermöglicht es Computern, Textähnlichkeit mathematisch zu berechnen.

**Beispiel**:
```
"Hund" → [0.8, 0.3, -0.2, ...]
"Katze" → [0.75, 0.35, -0.15, ...]
"Auto" → [0.1, -0.6, 0.9, ...]
```
→ "Hund" und "Katze" haben ähnliche Vektoren (beide Tiere)
→ "Auto" ist sehr unterschiedlich

## Wofür werden sie im Projekt verwendet?

Embeddings sind die Grundlage für RAG in **Agent 2**:

- **Dokument-Indexierung**: Jeder Text-Chunk wird in Embedding umgewandelt
- **Semantische Suche**: User-Frage wird als Embedding dargestellt und mit gespeicherten Embeddings verglichen
- **Embedding-Modell**: `nomic-embed-text` (via Ollama)
- **Vector Database**: pgvector (PostgreSQL-Erweiterung) in Dify High Quality Mode

## Kernkonzepte

### 1. Wie entstehen Embeddings?

**Embedding-Modelle** sind spezialisierte neuronale Netzwerke:

1. **Input**: Text (z.B. "Docker ist eine Container-Plattform")
2. **Verarbeitung**: Modell analysiert Kontext und Semantik
3. **Output**: Vektor mit fester Dimension (z.B. 1024 Zahlen)

**Im Projekt**:
- **Modell**: `nomic-embed-text` (via Ollama)
- **Dimensionen**: 1024 (Balance zwischen Qualität und Speicher)
- **Kontext**: Bis 8192 Tokens (sehr lange Texte möglich)

### 2. Eigenschaften von Embeddings

**Semantische Ähnlichkeit**:
- "Fahrzeug" ≈ "Auto" ≈ "PKW" (hohe Ähnlichkeit)
- "Fahrzeug" ≠ "Tomate" (geringe Ähnlichkeit)

**Kontextsensitiv**:
- "Bank" (Geldinstitut) vs. "Bank" (Sitzgelegenheit)
- Modell erkennt Bedeutung aus Kontext

**Sprachübergreifend** (bei multilingualen Modellen):
- "Hund" (Deutsch) ≈ "Dog" (Englisch)
- `nomic-embed-text` unterstützt multiple Sprachen

### 3. Cosine Similarity - Ähnlichkeit messen

Ähnlichkeit zwischen Vektoren wird mit **Cosine Similarity** gemessen:

```
Cosine Similarity = (A · B) / (||A|| × ||B||)
```

**Ergebnis**: Wert zwischen -1 und 1
- **1.0**: Identisch
- **0.8-0.9**: Sehr ähnlich
- **0.5-0.7**: Moderat ähnlich
- **< 0.5**: Unähnlich

**Beispiel**:
```
Query: "Wie starte ich Docker?"
Chunk 1: "Docker Desktop starten" → Similarity: 0.87
Chunk 2: "Dify mit Ollama verbinden" → Similarity: 0.42
Chunk 3: "docker compose up -d ausführen" → Similarity: 0.91
```
→ Chunk 3 wird priorisiert (höchste Similarity)

### 4. Vector Databases

Vector Databases speichern Embeddings und ermöglichen schnelle Ähnlichkeitssuche:

**Herausforderung**: Millionen von Vektoren durchsuchen in Millisekunden

**Lösung**: Spezielle Indexierungs-Algorithmen
- **HNSW** (Hierarchical Navigable Small World): Schnell, präzise
- **IVF** (Inverted File Index): Skaliert gut
- **LSH** (Locality-Sensitive Hashing): Trade-off Geschwindigkeit/Genauigkeit

### 5. Vector Database Optionen

| Datenbank | Typ | Use Case | Im Projekt |
|-----------|-----|----------|-----------|
| **pgvector** | PostgreSQL-Erweiterung | Einfach, SQL-basiert | ✅ Dify High Quality |
| **ChromaDB** | Standalone, file-based | Prototyping, lokal | Alternativ möglich |
| **Qdrant** | Server, REST API | Production, skalierbar | Alternativ möglich |
| **Pinecone** | Cloud-Service | Managed, keine Wartung | Nicht im Projekt |
| **Weaviate** | GraphQL, hybrid search | Komplexe Queries | Nicht im Projekt |

**Im Projekt**: **pgvector** wird automatisch von Dify verwendet (High Quality Mode).

### 6. nomic-embed-text - Das Projekt-Embedding-Modell

**Eigenschaften**:
- **Dimension**: 1024 (Variable mit Matryoshka Embedding in v1.5)
- **Kontextlänge**: 8192 Tokens (~6000 Wörter)
- **Sprachen**: Multilingual (Deutsch, Englisch, etc.)
- **Performance**: Besser als OpenAI `text-embedding-ada-002` bei Long-Context-Tasks

**Installation**:
```bash
ollama pull nomic-embed-text
```

**In Dify konfigurieren**:
1. **Settings** → **Model Providers** → **Ollama**
2. **Add model**:
   - **Model Type**: `Embedding`
   - **Model Name**: `nomic-embed-text`
   - **API Endpoint**: `http://host.docker.internal:11434`

## Typische Workflows

### Embedding-Modell in Dify einrichten

1. **Ollama**: Modell herunterladen
   ```bash
   ollama pull nomic-embed-text
   ```

2. **Dify**: Provider konfigurieren
   - **Settings** → **Model Providers** → **Ollama**
   - **Model Type**: `Embedding`
   - **Model Name**: `nomic-embed-text`

3. **Knowledge Base**: High Quality Mode aktivieren
   - **Index Mode**: `High Quality`
   - Embedding-Modell wird automatisch verwendet

### Knowledge Base indexieren (Hintergrund)

Beim Hochladen von Dokumenten in Dify:

1. **Chunking**: Dokument in ~500 Token Chunks aufteilen
2. **Embedding**: Jeder Chunk → `nomic-embed-text` → 1024-dim Vektor
3. **Speicherung**: Vektoren + Original-Text in pgvector
4. **Index**: HNSW-Index für schnelle Suche erstellen

**Dauer**: Abhängig von Dokumentgröße (z.B. 10 Seiten ≈ 30 Sekunden)

### Retrieval durchführen (bei jeder Query)

1. **User-Query**: "Wie verbinde ich Dify mit Ollama?"
2. **Query Embedding**: Query → `nomic-embed-text` → Vektor
3. **Similarity Search**: Finde Top-K ähnlichste Vektoren in pgvector
4. **Chunk Retrieval**: Hole zugehörige Text-Chunks
5. **Reranking** (optional): Bewerte Relevanz mit separatem Modell
6. **Augmentation**: Füge Chunks in LLM-Prompt ein

## Vorteile von Embeddings

- **Semantisches Verständnis**: "Container" findet auch "Docker" (ohne Keyword-Match)
- **Sprachflexibel**: Verschiedene Formulierungen führen zum gleichen Ergebnis
- **Skalierbar**: Millionen Dokumente durchsuchbar
- **Kontext-bewusst**: Mehrdeutigkeiten werden aufgelöst

## Limitierungen

- **Dimensionality**: Mehr Dimensionen = bessere Qualität, aber mehr Speicher/Rechenzeit
- **Out-of-Domain**: Embeddings funktionieren schlechter bei sehr speziellen Fachbegriffen
- **Cold Start**: Ohne Trainingsdaten für spezifische Domäne kann Qualität leiden
- **Rechenaufwand**: Embedding-Generierung benötigt Zeit (besonders bei großen Dokumenten)

## Economical vs. High Quality (Dify)

| Aspekt | Economical Mode | High Quality Mode |
|--------|-----------------|-------------------|
| **Suche** | Keyword (BM25) | Semantic (Embeddings) |
| **Embedding-Modell** | ❌ Nicht nötig | ✅ `nomic-embed-text` |
| **Vector DB** | ❌ Keine | ✅ pgvector |
| **Beispiel Query** | "Docker Container" | "Wie isoliere ich Apps?" |
| **Findet** | Exakte Matches | Semantisch ähnliche |
| **Setup** | Sofort einsatzbereit | Embedding-Modell nötig |
| **Performance** | Schneller | Langsamer |
| **Qualität** | Gut für exakte Begriffe | Besser für Konzepte |

## Häufige Fehler

### "Embedding model not found"
**Ursache**: `nomic-embed-text` nicht heruntergeladen oder nicht in Dify konfiguriert
**Lösung**:
```bash
ollama pull nomic-embed-text
```
Dann in Dify: Settings → Model Providers → Ollama → Embedding-Modell hinzufügen

### Schlechte Retrieval-Ergebnisse trotz High Quality
**Ursache**: Chunks zu groß/klein oder Dokumente unstrukturiert
**Lösung**:
- Dokumente besser strukturieren (Überschriften, Absätze)
- Chunk-Größe anpassen (in Dify: Settings → Knowledge)
- Top-K erhöhen (mehr Chunks abrufen)

### Indexierung dauert sehr lange
**Ursache**: Große Dokumente und CPU-Inferenz (kein GPU)
**Lösung**:
- Dokumente vorverarbeiten (unnötige Inhalte entfernen)
- GPU-Beschleunigung für Ollama aktivieren
- Geduld haben (einmalige Operation)

## Advanced: Matryoshka Embeddings (nomic-embed-text v1.5)

**Matryoshka Embeddings** = Variable Dimensionen aus einem Modell:

```
nomic-embed-text v1.5 kann liefern:
- 1024 Dimensionen (maximale Qualität)
- 512 Dimensionen (schneller, weniger Speicher)
- 256 Dimensionen (noch schneller)
- ...
```

**Trade-off**: Weniger Dimensionen = schneller, aber etwas weniger präzise

**Im Projekt**: Standard 1024 Dimensionen (optimale Balance)

## Alternative Embedding-Modelle

| Modell | Dimensionen | Context | Qualität | Im Projekt |
|--------|------------|---------|----------|-----------|
| **nomic-embed-text** | 1024 | 8192 | Sehr gut | ✅ Verwendet |
| **all-MiniLM-L6-v2** | 384 | 512 | Mittel | Alternativ |
| **mxbai-embed-large** | 1024 | 512 | Gut | Alternativ |
| **OpenAI ada-002** | 1536 | 8191 | Sehr gut | Cloud-only |

**Empfehlung**: `nomic-embed-text` (beste Balance für lokalen Einsatz)

## Visualisierung von Embeddings

Embeddings können in 2D/3D visualisiert werden (mit t-SNE oder UMAP):

```
[Hochdimensionaler Raum] → [Projektion] → [2D-Plot]
- Cluster = semantisch ähnliche Dokumente
- Abstand = Unähnlichkeit
```

**Tools**: TensorBoard, ChromaDB Visualization, Weights & Biases

## Weiterführende Links

- **Ollama Embedding Models**: [https://ollama.com/blog/embedding-models](https://ollama.com/blog/embedding-models)
- **ChromaDB Embeddings Guide**: [https://docs.trychroma.com/docs/embeddings/embedding-functions](https://docs.trychroma.com/docs/embeddings/embedding-functions)
- **nomic-embed-text auf Ollama**: [https://ollama.com/library/nomic-embed-text](https://ollama.com/library/nomic-embed-text)
- **Real Python ChromaDB Tutorial**: [https://realpython.com/chromadb-vector-database/](https://realpython.com/chromadb-vector-database/)

---

**Quellen**: Eigene Zusammenstellung basierend auf Embedding-Research 2025 und Projekt-Implementierung