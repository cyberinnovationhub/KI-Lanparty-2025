# Azure OpenAI Service - Cloud-basierte LLMs

## Was ist Azure OpenAI?

Azure OpenAI Service ist eine Cloud-Plattform von Microsoft, die Zugriff auf OpenAI-Modelle wie GPT-4o, GPT-4, GPT-3.5 und DALL-E über eine REST API bietet. Im Gegensatz zu OpenAI direkt läuft der Service in Microsofts Azure-Infrastruktur mit zusätzlichen Enterprise-Features (SLAs, Compliance, private Netzwerke).

**Vergleich**:
- **OpenAI API**: Direkt von OpenAI (api.openai.com)
- **Azure OpenAI**: Gleiche Modelle, aber auf Azure gehostet (*.openai.azure.com)

## Wofür wird es im Projekt verwendet?

Azure OpenAI wird für **Agent 2** (Dokumentations-Experte) verwendet:

- **GPT-4o**: Höhere Qualität als lokale 8B-Modelle für komplexe Dokumentations-Anfragen
- **Multimodalität**: GPT-4o kann Bilder und Audio verarbeiten (zukünftig erweiterbar)
- **Zuverlässigkeit**: Konsistente Antwortqualität und Verfügbarkeit
- **Deployment**: `ai-lanparty` Deployment mit API-Version `2025-01-01-preview`

**Hybrid-Ansatz**: Agent 2 kann zwischen lokalem Llama/Qwen und Azure GPT-4o wechseln.

## Kernkonzepte

### 1. Deployments

Azure OpenAI nutzt **Deployments** statt direkter Modellnamen:

**OpenAI API**:
```json
{
  "model": "gpt-4o"
}
```

**Azure OpenAI API**:
```json
{
  "deployment": "ai-lanparty",
  "model": "gpt-4o"  // Base Model
}
```

**Warum?**
- Mehrere Deployments des gleichen Modells möglich (z.B. dev, prod)
- Versionskontrolle und Traffic-Routing
- Individuelle Rate Limits pro Deployment

**Im Projekt**: Deployment-Name = `ai-lanparty`

### 2. API Versions

Azure OpenAI hat monatliche API-Versionen mit neuen Features:

**Aktuelle Versionen (2025)**:
- **2024-10-21**: Latest GA (General Availability)
- **2025-01-01-preview**: Preview mit neuesten Features
- **Legacy**: 2024-06-01, 2023-12-01, etc.

**Im Projekt**: `2025-01-01-preview` für Zugriff auf neueste GPT-4o Features

**Best Practice**: Preview für Entwicklung, GA für Production

### 3. Endpoint-Struktur

Azure OpenAI Endpoints haben spezifisches Format:

```
https://[RESOURCE_NAME].openai.azure.com/openai/deployments/[DEPLOYMENT_NAME]/chat/completions?api-version=[API_VERSION]
```

**Im Projekt**:
```
https://ai-lanparty.openai.azure.com/openai/deployments/gpt-4o/chat/completions?api-version=2025-01-01-preview
```

**Komponenten**:
- **Resource**: `ai-lanparty` (Azure-Ressource)
- **Deployment**: `gpt-4o` (Deployment-Name)
- **Endpoint**: `/chat/completions` (Chat-API)
- **API Version**: `2025-01-01-preview`

### 4. Authentifizierung

**API Key** (verwendet im Projekt):
```bash
curl https://ai-lanparty.openai.azure.com/... \
  -H "api-key: YOUR_API_KEY"
```

**Azure AD (Enterprise)**:
```bash
curl https://ai-lanparty.openai.azure.com/... \
  -H "Authorization: Bearer [AZURE_AD_TOKEN]"
```

**Im Projekt**: API Key (einfacher für Workshop)

### 5. Rate Limits und Quotas

Azure OpenAI hat Limits pro Deployment:

**Tokens Per Minute (TPM)**:
- **Standard**: 30k - 150k TPM (abhängig von Region/Tier)
- **PTU (Provisioned Throughput Units)**: Garantierte Kapazität

**Requests Per Minute (RPM)**:
- **Standard**: 300 - 1800 RPM

**Im Projekt**: Standard-Tier mit shared Quota (ausreichend für Workshop)

**Bei Überschreitung**: HTTP 429 "Too Many Requests" → Retry nach 1-60 Sekunden

### 6. GPT-4o Modell-Features

**Multimodal**:
- **Text**: Hauptfunktion
- **Images**: Bilder in Prompts verarbeiten (z.B. Screenshots analysieren)
- **Audio**: Spracheingabe/-ausgabe (realtime API)

**Context Window**: 128k Tokens (~96.000 Wörter)

**Output**: Bis 16k Tokens

**Training Data**: Bis Oktober 2023 (Stand 2025)

**Im Projekt**: Nur Text-Modus (für Dokumentations-Anfragen)

## Typische Workflows

### Azure OpenAI in Dify einrichten

1. **Dify UI**: Profilbild → **Settings** → **Model Providers**

2. **Azure OpenAI Service** auswählen

3. **Deployment konfigurieren**:
   - **Deployment Name**: `ai-lanparty`
   - **Model Type**: `LLM`
   - **Authorization Name**: `Authorization GPT-4o` (Display-Name)
   - **API Endpoint URL**:
     ```
     https://ai-lanparty.openai.azure.com/openai/deployments/gpt-4o/chat/completions?api-version=2025-01-01-preview
     ```
   - **API Key**: [Von Event-Veranstalter erhalten]
   - **API Version**: `2025-01-01-preview`
   - **Base Model**: `gpt-4o`

4. **Save** → Modell in Agent auswählen

### API direkt nutzen (cURL)

```bash
curl https://ai-lanparty.openai.azure.com/openai/deployments/gpt-4o/chat/completions?api-version=2025-01-01-preview \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_API_KEY" \
  -d '{
    "messages": [
      {"role": "system", "content": "Du bist ein hilfreicher Assistent."},
      {"role": "user", "content": "Erkläre Docker in 2 Sätzen"}
    ],
    "max_tokens": 100,
    "temperature": 0.7
  }'
```

### Python-SDK nutzen

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key="YOUR_API_KEY",
    api_version="2025-01-01-preview",
    azure_endpoint="https://ai-lanparty.openai.azure.com"
)

response = client.chat.completions.create(
    model="gpt-4o",  # Deployment-Name
    messages=[
        {"role": "system", "content": "Du bist ein hilfreicher Assistent."},
        {"role": "user", "content": "Erkläre Docker in 2 Sätzen"}
    ]
)

print(response.choices[0].message.content)
```

## Kosten (Stand 2025)

**GPT-4o Pricing** (Azure, kann variieren):
- **Input**: ~$2.50 / 1M Tokens
- **Output**: ~$10.00 / 1M Tokens

**Beispielrechnung**:
- **1000 Anfragen** à 500 Input + 200 Output Tokens:
  - Input: 0.5M Tokens × $2.50 = $1.25
  - Output: 0.2M Tokens × $10.00 = $2.00
  - **Gesamt**: ~$3.25

**Im Projekt**: Shared Budget für Workshop (Event-Veranstalter übernimmt Kosten)

## Vorteile von Azure OpenAI

- **Enterprise-Features**: SLAs, Private Endpoints, VNet-Integration
- **Compliance**: DSGVO-konform, ISO-zertifiziert, HIPAA-compliant
- **Regionale Verfügbarkeit**: Daten bleiben in EU (wichtig für Compliance)
- **Integration**: Nahtlos mit Azure-Services (Cognitive Search, Functions, etc.)
- **Versionskontrolle**: Mehrere Deployments für Staging/Production

## Limitierungen

- **Kosten**: Pay-per-Token (vs. lokal kostenlos)
- **Latenz**: Netzwerk-Overhead (vs. lokal instant)
- **Datenschutz**: Daten gehen zu Microsoft (bei sensiblen Daten problematisch)
- **Vendor Lock-in**: Abhängigkeit von Azure-Infrastruktur
- **Quota-Limits**: Bei hohem Traffic können Limits erreicht werden

## Häufige Fehler

### "Resource not found"
**Ursache**: Falscher Resource-Name oder Deployment-Name
**Lösung**:
- Prüfe Endpoint-URL: `https://[RESOURCE].openai.azure.com`
- Prüfe Deployment-Name im Endpoint: `/deployments/[DEPLOYMENT]`

### "401 Unauthorized"
**Ursache**: API Key falsch oder abgelaufen
**Lösung**:
- API Key vom Veranstalter neu erhalten
- Header korrekt: `api-key: YOUR_KEY` (nicht `Authorization: Bearer ...`)

### "429 Too Many Requests"
**Ursache**: Rate Limit überschritten
**Lösung**:
- Exponential Backoff implementieren (Retry nach 1s, 2s, 4s, ...)
- Anfragen batchen oder Queue nutzen
- PTU (Provisioned Throughput) upgraden (kostenintensiv)

### "Invalid API version"
**Ursache**: API-Version nicht unterstützt oder veraltet
**Lösung**:
- Aktuelle GA-Version nutzen: `2024-10-21`
- Preview-Features: `2025-01-01-preview`
- Dokumentation prüfen: [API Version Lifecycle](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/api-version-lifecycle)

## Azure OpenAI vs. OpenAI vs. Lokal

| Aspekt | Azure OpenAI | OpenAI API | Lokal (Ollama) |
|--------|--------------|-----------|----------------|
| **Qualität** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ (8B) |
| **Kosten** | $2-10/1M Tok | $2-15/1M Tok | Kostenlos |
| **Latenz** | ~500-1000ms | ~300-800ms | ~100-5000ms |
| **Privacy** | Microsoft Azure | OpenAI Cloud | 100% lokal |
| **Compliance** | DSGVO, ISO, HIPAA | DSGVO | Keine Limits |
| **Setup** | API Key | API Key | Ollama installieren |
| **Offline** | ❌ | ❌ | ✅ |
| **Enterprise** | ✅ SLAs, VNet | ❌ | ❌ |

**Im Projekt**: Hybrid für Agent 2 (lokale Modelle + Azure GPT-4o)

## 2025 Updates

### V1 API (August 2025)
- **Vereinfachte API**: Keine monatlichen API-Version-Wechsel mehr
- **Opt-in**: `api-version=v1` für immer aktuellste Features
- **Backward Compatible**: Alte Versionen weiter unterstützt

### GPT-4o Audio (2025)
- **Realtime API**: Sprach-zu-Sprach mit <300ms Latenz
- **Audio Generation**: TTS mit natürlichen Stimmen
- **Multimodal**: Audio + Text gleichzeitig verarbeiten

### Azure AI Foundry
- **Neuer Name**: Azure OpenAI wird Teil von "Azure AI Foundry"
- **Mehr Modelle**: Integration von Anthropic Claude, Mistral, etc.
- **Unified API**: Ein Endpoint für mehrere Modell-Anbieter

## Weiterführende Links

- **Azure OpenAI Dokumentation**: [https://learn.microsoft.com/en-us/azure/ai-foundry/openai/overview](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/overview)
- **What's New**: [https://learn.microsoft.com/en-us/azure/ai-foundry/openai/whats-new](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/whats-new)
- **API Version Lifecycle**: [https://learn.microsoft.com/en-us/azure/ai-foundry/openai/api-version-lifecycle](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/api-version-lifecycle)
- **Pricing**: [https://azure.microsoft.com/en-us/pricing/details/cognitive-services/openai-service/](https://azure.microsoft.com/en-us/pricing/details/cognitive-services/openai-service/)
- **Python SDK**: [https://github.com/openai/openai-python](https://github.com/openai/openai-python) (unterstützt Azure)

---

**Quellen**: Eigene Zusammenstellung basierend auf Microsoft Azure OpenAI Dokumentation (2025) und Projekt-Setup