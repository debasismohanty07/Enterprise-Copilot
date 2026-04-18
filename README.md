## This is a private project built specifically for assisting pre-sales and solutioning  teams to craft curated and contextualized response to the RFI/RFP/RFQ, For more details contact through Linkedin -https://www.linkedin.com/in/debasis07/ 

# RFP Copilot Web App

An AI-augmented enterprise web application that assists proposal teams in generating structured, context-aware RFP responses using a **multi-agent AI pipeline**, intelligent caching, organizational memory, and a human-in-the-loop review workflow.

---

## Project Summary

RFP Copilot transforms how organizations respond to Requests for Proposal. Instead of a single LLM call, it orchestrates four specialized AI agents — Planner, Researcher, Writer, and Critic — each with a dedicated role and system prompt. The Researcher agent searches both your company knowledge base and a growing organizational memory of past approved responses, so the system gets smarter with every approval. Every response goes through a quality gate (Critic scores 1–10) and a human review workflow before being finalized, with approved responses automatically written back to cache and memory for future reuse.

### Key Highlights

| Capability | Detail |
|---|---|
| Multi-agent pipeline | Planner → Researcher → Writer → Critic (OpenAI) |
| Organizational memory | Approved responses persist and are surfaced by agents for future questions |
| Semantic memory search | Cosine similarity via text-embedding-3-small; keyword fallback when unavailable |
| Critic-gated quality | Responses scoring < 7/10 are auto-revised before delivery |
| Agent reasoning trace | Full audit trail of every agent action visible in the UI |
| Intelligent caching | Exact + Jaccard similarity matching — 60–80% API cost reduction |
| Human review workflow | Per-category routing, timestamped approvals/rejections, reviewer comments |
| Post-approval learning | Approved responses written to both cache and organizational memory |
| Knowledge base | Upload PDF/TXT/MD — relevance-filtered RAG (min 2 keyword overlap) |
| Multi-provider LLM | OpenAI (multi-agent) or AWS Bedrock (classic RAG fallback) |
| Zero external DB | Pure file-system storage — no Redis, no vector DB required |
| 564 tests | Unit + integration + 28 property-based tests (100+ iterations each) |

---

## Tech Stack & Dependencies

### Runtime & Language

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js 16.x or higher |
| Language | JavaScript (CommonJS — `require`/`module.exports` throughout) |
| Module system | CommonJS (no ESM/TypeScript compilation required) |
| Frontend | Vanilla HTML5 + CSS3 + JavaScript (no framework) |
| Storage | File system (JSON files — no external database) |

### Production Dependencies

| Package | Version | Role in this project |
|---------|---------|----------------------|
| `express` | ^4.18.2 | Web server framework — handles all REST API routing, middleware, static file serving, and JSON parsing |
| `dotenv` | ^16.3.1 | Loads environment variables from `.env` into `process.env` at startup — used for API keys, provider selection, cache config, and server settings |
| `cors` | ^2.8.5 | Cross-Origin Resource Sharing middleware — restricts browser access to the API to origins listed in `CORS_ORIGINS` |
| `openai` | ^4.104.0 | Official OpenAI Node.js client — used for chat completions (`callWithTools`) enabling the multi-agent pipeline (Planner, Researcher, Writer, Critic) with function/tool calling; also provides `embed()` via `text-embedding-3-small` for semantic memory search in `AgentMemory` |
| `@aws-sdk/client-bedrock-runtime` | ^3.450.0 | AWS SDK v3 client for Bedrock — invokes Nova Pro and Claude models via `InvokeModelCommand` for the classic RAG pipeline when Bedrock is the selected provider; model-aware request builder handles both Nova Pro (Messages API) and Claude 3.x formats |
| `@aws-sdk/client-bedrock-agent-runtime` | ^3.1014.0 | AWS SDK v3 client for Bedrock Agent Runtime — used for semantic vector search via `RetrieveCommand` against Bedrock Knowledge Base, and content filtering via Guardrails on KB retrieval results |
| `multer` | ^2.1.1 | Multipart form-data middleware for Express — handles document file uploads (PDF, TXT, MD) to the knowledge base endpoint; configured with memory storage and 10MB size limit |
| `pdf-parse` | ^1.1.1 | Extracts plain text from PDF file buffers — used by `TextExtractor` to chunk PDF documents for knowledge base indexing |

### Development Dependencies

| Package | Version | Role in this project |
|---------|---------|----------------------|
| `jest` | ^29.7.0 | Test runner — executes all unit, integration, and property-based tests; configured via `jest.config.js` |
| `fast-check` | ^3.15.0 | Property-based testing library — generates 100+ random inputs per test to verify correctness properties of CacheManager, SimilarityCalculator, KnowledgeBase, ResponseGenerator, and other components |
| `supertest` | ^6.3.3 | HTTP assertion library for Express — used in server and integration tests to make in-process HTTP requests without starting a real server |

### Node.js Built-in Modules Used

| Module | Usage |
|--------|-------|
| `fs` | Synchronous file I/O for all JSON persistence (cache, knowledge base, memory, reviews) |
| `path` | Cross-platform file path construction |
| `crypto` | SHA-256 hashing for exact cache key generation |

### Frontend Stack (no build step required)

| Technology | Usage |
|-----------|-------|
| HTML5 | Single-page layout (`public/index.html`) with 5 panel sections |
| CSS3 | Custom properties (CSS variables), flexbox layout, responsive sidebar |
| Vanilla JavaScript | All UI logic in `public/app.js` — fetch API, DOM manipulation, event handling |
| No framework | Zero dependencies on React, Vue, Angular, or any bundler |

---

## Features

- **Multi-Agent Orchestration**: Four specialized agents coordinate response generation — Planner decides strategy, Researcher searches KB and organizational memory, Writer drafts the response, Critic scores and revises
- **Organizational Memory**: Approved responses accumulate as institutional knowledge in `data/memory/` — the Researcher agent references past RFP wins for future similar questions
- **Semantic Memory Search**: `AgentMemory` uses OpenAI `text-embedding-3-small` (1536-dim vectors) for cosine similarity search; automatically falls back to keyword overlap when embeddings are unavailable or the API is unreachable
- **Agent Reasoning Trace**: Collapsible UI panel shows every agent action, tool call, and critique score for full transparency
- **Intelligent Response Caching**: Exact and Jaccard similarity-based caching reduces API costs by 60–80%; approved responses also written to cache
- **Human Review Workflow**: Per-category review tasks routed to designated reviewers; approve/reject with timestamps and optional comments; approved responses written to cache and memory
- **Knowledge Base Integration**: Two modes — Amazon Bedrock KB (semantic vector search via Titan Embeddings v2) when `BEDROCK_KB_ID` is set, or local keyword-based retrieval as fallback; Researcher only uses KB when relevant (relevance score ≥ 2)
- **Bedrock Guardrails**: Content filtering on KB retrieval — blocks harmful content, detects PII, validates topic relevance before injecting context into prompts
- **Source Citations**: Responses include hyperlinked document badges showing which KB files were referenced
- **Multi-file Upload**: Upload multiple documents at once with sequential progress tracking
- **Multiple LLM Provider Support**: OpenAI (gpt-4o-mini, multi-agent) or AWS Bedrock (Claude, classic RAG)
- **Web Interface**: Dark sidebar UI with Ask, Knowledge Base, Cache, Settings, and Reviews panels
- **Export Functionality**: Export responses in JSON or plain text format
- **Robust Error Handling**: Automatic retry with exponential backoff (3 attempts: 1s → 2s → 4s)
- **Configuration Validation**: Test LLM provider connectivity before processing questions

---

## Prerequisites

- Node.js 16.x or higher
- npm or yarn package manager
- API credentials for either:
  - OpenAI API key (recommended — enables full multi-agent mode), or
  - AWS account with Bedrock access (uses classic RAG pipeline)

---

## Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd rfp-copilot-web-app
```

2. Install dependencies:
```bash
npm install
```

3. Configure environment variables:
```bash
cp .env.example .env
```

4. Edit `.env` with your configuration (see Configuration section below)

---

## Configuration

### Environment Variables

#### LLM Provider Selection

```bash
# Choose either 'openai' or 'bedrock'
LLM_PROVIDER=openai
```

#### OpenAI Configuration (Recommended)

```bash
OPENAI_API_KEY=sk-your-api-key-here
OPENAI_MODEL=gpt-4o-mini
```

> **Note:** OpenAI enables the full multi-agent pipeline (Planner→Researcher→Writer→Critic) with function calling. Each agent has its own system prompt and tool set.

**Getting an OpenAI API Key:**
1. Sign up at https://platform.openai.com/
2. Navigate to API Keys section
3. Create a new API key and add billing credits
4. Copy the key to your `.env` file

#### AWS Bedrock Configuration

```bash
LLM_PROVIDER=bedrock
AWS_REGION=us-east-1
AWS_BEDROCK_MODEL=amazon.nova-pro-v1:0
```

> **Note:** Bedrock uses the classic RAG pipeline (classify → KB search → generate). The multi-agent pipeline is not available for Bedrock providers. The BedrockProvider is model-aware — it handles both Nova Pro (Messages API) and Claude 3.x (Anthropic API) request formats automatically.

**Supported Bedrock Models:**
- `amazon.nova-pro-v1:0` — Amazon Nova Pro (recommended, ~$0.80/1M input tokens)
- `anthropic.claude-3-sonnet-*` — Claude 3 Sonnet
- `anthropic.claude-3-haiku-*` — Claude 3 Haiku

**AWS Credentials** — configure using one of:
- AWS CLI: `aws configure`
- Environment variables: `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` (local dev only)
- IAM role (EC2/ECS) — recommended for production, no keys in `.env`
- AWS credentials file (`~/.aws/credentials`)

**Required IAM Permissions:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["bedrock:InvokeModel"],
      "Resource": "arn:aws:bedrock:*:*:foundation-model/*"
    },
    {
      "Effect": "Allow",
      "Action": ["bedrock:Retrieve"],
      "Resource": "arn:aws:bedrock:*:*:knowledge-base/*"
    },
    {
      "Effect": "Allow",
      "Action": ["bedrock:ApplyGuardrail"],
      "Resource": "arn:aws:bedrock:*:*:guardrail/*"
    }
  ]
}
```

#### Server Configuration

```bash
PORT=3000
CORS_ORIGINS=http://localhost:3000,http://localhost:3001
```

#### Processing Limits (Optional)

```bash
MAX_QUESTION_LENGTH=10000
MIN_RESPONSE_LENGTH=50
MAX_RESPONSE_LENGTH=2000
REQUEST_TIMEOUT=30000
MAX_QUESTIONS_PER_MINUTE=10
```

#### Cache Configuration (Optional)

```bash
# Enable or disable response caching (default: true)
ENABLE_CACHE=true

# Days before cached responses expire (default: 30)
CACHE_EXPIRATION_DAYS=30

# Maximum cached entries (default: 1000)
MAX_CACHE_ENTRIES=1000

# Jaccard similarity threshold for similar question matching, 0.0–1.0 (default: 0.85)
CACHE_SIMILARITY_THRESHOLD=0.85
```

#### Knowledge Base Configuration (Optional)

```bash
# Maximum document upload size in MB (default: 10)
MAX_DOCUMENT_SIZE_MB=10

# Bedrock Knowledge Base — semantic vector search (optional, replaces local keyword search)
BEDROCK_KB_ID=OG1VQD4BEW
BEDROCK_KB_REGION=us-east-1

# Bedrock Guardrail — content filtering on KB retrieval (optional)
BEDROCK_GUARDRAIL_ID=90zc7vnruh89
BEDROCK_GUARDRAIL_VERSION=1
```

> **Note:** When `BEDROCK_KB_ID` is set, the Knowledge Base uses Amazon Bedrock's semantic vector search (Titan Embeddings v2 + OpenSearch Serverless) instead of local keyword matching. If the Bedrock KB call fails, it automatically falls back to local keyword search. The Guardrail filters harmful, off-topic, or PII-containing content from KB retrieval results.

#### Review Workflow Configuration (Optional)

```bash
# Enable or disable human review workflow (default: true)
ENABLE_REVIEW=true
```

**Supported Formats:** `.pdf` (text-based only), `.txt`, `.md`

---

## Usage

### Starting the Server

Development mode (auto-reload on file changes):
```bash
npm run dev
```

Production mode:
```bash
npm start
```

Server starts on port 3000 (default):
```
RFP Copilot Web App server listening on port 3000
Health check available at http://localhost:3000/health
Initialized OpenAI provider with model: gpt-4o-mini
```

### Web Interface

Navigate to `http://localhost:3000` and use the five sidebar panels:

**Ask Panel**
1. Enter your RFP question (up to 10,000 characters)
2. Click "Generate Response"
3. View the response with category badges, confidence scores, and source citations
4. Expand the **Agent Reasoning Trace** to see every agent's actions, tool calls, and the Critic's score
5. Source document badges are hyperlinked — click to navigate to the Knowledge Base panel
6. Export the response as JSON or plain text

**Knowledge Base Panel**
- Upload documents (PDF, TXT, MD — up to 10MB each)
- Select multiple files at once for batch upload
- View uploaded documents with chunk count and file size
- Delete documents when no longer needed

**Cache Panel**
- View hit rate, total entries, exact hits, and similar hits
- Clear all cache entries or entries older than 30 days

**Settings Panel**
- Validate LLM provider configuration and connectivity

**Reviews Panel**
- View pending, approved, and rejected review tasks
- Filter by status; pending count shown as badge on nav item
- Approve with optional edited response and reviewer comment
- Reject with required reason and optional comment
- All actions are timestamped; approved responses are written to cache and organizational memory

### How the Multi-Agent Pipeline Works

When using OpenAI, four specialized agents coordinate to produce the response:

```
User Question
      ↓
1. PLANNER — classifies question, sets strategy, decides if KB/memory search needed
      ↓
2. RESEARCHER — tool-calling loop (max 4 iterations):
   • search_knowledge_base → company documents (relevance score ≥ 2 filter)
   • search_memory → past approved responses from organizational memory
   → produces research brief with cited sources
      ↓
3. WRITER — drafts professional response using plan + research brief
      ↓
4. CRITIC — scores draft 1-10; if score < 7, provides revised response
      ↓
Final response + agent trace returned to UI
```

> If you ask about a company not in your KB, the Planner sets `needsKB: false` and the Researcher skips the KB search entirely — no false citations.

> As you approve responses, the organizational memory grows. The Researcher will begin referencing past wins for future similar questions, improving response quality over time.

### Using the API Directly

#### Process a Question

```bash
curl -X POST http://localhost:3000/api/process-question \
  -H "Content-Type: application/json" \
  -d '{"question": "What is your experience with cloud infrastructure?"}'
```

Response (OpenAI, fresh generation):
```json
{
  "success": true,
  "data": {
    "responseText": "Our company has extensive experience...\n\nSources:\n1. company-overview.pdf",
    "categories": [{"category": "experience", "confidence": 0.92}],
    "timestamp": "2026-03-21T07:30:00.000Z",
    "processingTime": 14200,
    "needsManualReview": false,
    "provider": "openai",
    "model": "gpt-4o-mini",
    "cacheHit": false,
    "knowledgeBaseUsed": true,
    "referencedDocuments": [{"id": "uuid", "filename": "company-overview.pdf"}],
    "contextChunkCount": 3,
    "agentIterations": 4,
    "critiqueScore": 8,
    "memoryHitsUsed": 2,
    "plan": {"category": "experience", "confidence": 0.92, "strategy": "..."},
    "agentTrace": [
      {"agent": "planner", "action": "plan_created"},
      {"agent": "researcher", "action": "kb_search", "chunks": 3},
      {"agent": "researcher", "action": "memory_search", "hits": 2},
      {"agent": "writer", "action": "draft_ready"},
      {"agent": "critic", "action": "review_done", "score": 8, "approved": true}
    ],
    "reviewStatus": "pending",
    "reviewTasks": [{"id": "rev_...", "category": "experience", "reviewer": "debasismohanty07@outlook.com", "status": "pending"}]
  }
}
```

#### Validate Configuration

```bash
curl http://localhost:3000/api/validate-config
```

#### Export Response

```bash
curl -X POST http://localhost:3000/api/export \
  -H "Content-Type: application/json" \
  -d '{"response": { ... }, "format": "json"}' \
  --output response.json
```

---

## API Reference

### Core Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| POST | `/api/process-question` | Process RFP question (multi-agent or RAG) |
| GET | `/api/validate-config` | Validate LLM connectivity |
| POST | `/api/export` | Export response (json/text) |

### Cache Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/cache/stats` | Hit rate, entries, storage size |
| POST | `/api/cache/invalidate?all=true` | Clear all entries |
| POST | `/api/cache/invalidate?beforeDate=ISO` | Clear entries before date |
| POST | `/api/cache/invalidate?category=name` | Clear entries by category |

### Knowledge Base Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/knowledge-base/upload` | Upload document (multipart/form-data) |
| GET | `/api/knowledge-base/documents` | List all documents |
| DELETE | `/api/knowledge-base/documents/:id` | Delete document + chunks |
| GET | `/api/knowledge-base/stats` | Document count, chunks, storage |

### Review Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/reviews` | List tasks (`?status=pending\|approved\|rejected`) |
| GET | `/api/reviews/stats` | Pending / approved / rejected / total counts |
| GET | `/api/reviews/:id` | Single review task |
| POST | `/api/reviews/:id/approve` | Approve (optional: `editedResponse`, `comment`) |
| POST | `/api/reviews/:id/reject` | Reject (required: `reason`; optional: `comment`) |

### Response Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `responseText` | string | Generated response (50–3,000 chars) |
| `categories` | array | `[{category, confidence}]` — classification results |
| `cacheHit` | boolean | True if served from cache |
| `cacheType` | string | `"exact"` or `"similar"` |
| `knowledgeBaseUsed` | boolean | True if KB chunks were injected |
| `referencedDocuments` | array | `[{id, filename}]` — documents used |
| `contextChunkCount` | number | Number of KB chunks used |
| `agentIterations` | number | Always 4 for multi-agent pipeline |
| `critiqueScore` | number | Critic agent quality score (1-10) — OpenAI only |
| `memoryHitsUsed` | number | Past approved responses used by Researcher |
| `plan` | object | Planner output: category, confidence, strategy, keyPoints |
| `agentTrace` | array | Full audit trail of agent actions — OpenAI only |
| `reviewStatus` | string | `"pending"` \| `"not_required"` \| `"approved"` |
| `reviewTasks` | array | `[{id, category, reviewer, status}]` |
| `needsManualReview` | boolean | True if critic score < 7 or all confidence < 0.5 |

### Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| `VALIDATION_ERROR` | 400 | Invalid input |
| `AUTH_ERROR` | 401 | Invalid or missing credentials |
| `TIMEOUT` | 408 | Request timed out |
| `RATE_LIMIT` | 429 | Rate limit exceeded |
| `SERVICE_UNAVAILABLE` | 503 | LLM provider unavailable |
| `UNKNOWN_ERROR` | 500 | Unexpected error |

---

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────┐
│   Browser (Vanilla JS + HTML)                           │
│   Ask / KB / Cache / Settings / Reviews panels          │
│   Agent Reasoning Trace (collapsible)                   │
└──────────────────────────┬──────────────────────────────┘
                           │ HTTP REST
                           ▼
┌─────────────────────────────────────────────────────────┐
│              Express API Server (port 3000)             │
│                                                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Cache Manager                                    │  │
│  │  - Exact match (SHA-256 hash)                     │  │
│  │  - Similarity match (Jaccard ≥ 0.85)              │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Multi-Agent Orchestrator (OpenAI)                │  │
│  │  1. Planner  — strategy + classification          │  │
│  │  2. Researcher — KB search + memory search        │  │
│  │  3. Writer   — draft response                     │  │
│  │  4. Critic   — score + revise (threshold: 7/10)   │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Knowledge Base                                   │  │
│  │  - Bedrock KB: semantic vector search (Titan v2)  │  │
│  │  - Guardrail: content filtering on retrieval      │  │
│  │  - Local fallback: keyword Jaccard (score ≥ 2)    │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Agent Memory                                     │  │
│  │  - Semantic search (cosine, text-embedding-3-small│  │
│  │  - Keyword fallback (Jaccard) when no embeddings  │  │
│  │  - Stores vectors in data/memory/embeddings.json  │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Review Manager                                   │  │
│  │  - Per-category routing to reviewers              │  │
│  │  - Timestamped approve/reject + comments          │  │
│  │  - Post-approval: writes to cache + memory        │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │  LLM Provider Adapter                             │  │
│  │  - callWithTools() — multi-agent (OpenAI)         │  │
│  │  - callLLM()       — classic RAG (Bedrock)        │  │
│  └───────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     │
              ┌──────┴──────┐
              ▼             ▼
        ┌──────────┐  ┌──────────────┐
        │  OpenAI  │  │   Bedrock    │
        │gpt-4o-mini│  │  Nova Pro    │
        └──────────┘  │  + KB + Guard│
                      └──────────────┘

┌─────────────────────────────────────────────────────────┐
│     File System Storage                                 │
│  data/cache/      — entries.json, stats.json            │
│  data/knowledge/  — documents.json, chunks.json         │
│  data/memory/     — approved_responses.json             │
│  data/reviews/    — tasks.json                          │
└─────────────────────────────────────────────────────────┘
```

### Request Flow

```
User Question
      ↓
Cache lookup (exact SHA-256) ──→ HIT: return instantly (< 50ms), reviewStatus: approved
      ↓ miss
Cache lookup (Jaccard ≥ 0.85) ─→ HIT: return cached response, reviewStatus: approved
      ↓ miss
Multi-Agent Pipeline (OpenAI):
  Planner  → classify + strategy + needsKB/needsMemory flags
  Researcher → search_knowledge_base + search_memory (tool-calling loop, max 4 iter)
  Writer   → draft response using plan + research brief
  Critic   → score 1-10; revise if score < 7
      ↓
Store in cache + create review tasks (per category, routed to reviewer)
      ↓
Return to UI (responseText + agentTrace + critiqueScore + plan + memoryHitsUsed)
      ↓
On Review Approval:
  → Write to cache (future similar questions get instant hit)
  → Write to organizational memory (agents reference for future questions)
  → Record reviewedAt timestamp + reviewer comment
```

### Data Persistence

| Store | Path | Contents |
|-------|------|----------|
| Cache entries | `data/cache/entries.json` | Question-response pairs |
| Cache stats | `data/cache/stats.json` | Hit/miss counters |
| KB documents | `data/knowledge/documents.json` | Document metadata |
| KB chunks | `data/knowledge/chunks.json` | Text chunks + keywords |
| Org memory | `data/memory/approved_responses.json` | Approved Q&A pairs |
| Memory vectors | `data/memory/embeddings.json` | text-embedding-3-small vectors (1536 dims) for semantic search |
| Review tasks | `data/reviews/tasks.json` | Review task queue |

All data survives application restarts. No external database required.

---

## Cache API

#### Get Cache Statistics

```bash
curl http://localhost:3000/api/cache/stats
```

#### Clear All Cache

```bash
curl -X POST "http://localhost:3000/api/cache/invalidate?all=true"
```

#### Clear Old Cache Entries

```bash
curl -X POST "http://localhost:3000/api/cache/invalidate?beforeDate=2026-01-01T00:00:00Z"
```

#### Clear by Category

```bash
curl -X POST "http://localhost:3000/api/cache/invalidate?category=technical"
```

> Parameters are **query string** parameters, not request body.

---

## Knowledge Base API

#### Upload a Document

```bash
curl -X POST http://localhost:3000/api/knowledge-base/upload \
  -F "file=@company-overview.pdf" \
  -F "title=Company Overview" \
  -F "description=General company information"
```

#### List Documents

```bash
curl http://localhost:3000/api/knowledge-base/documents
```

#### Delete a Document

```bash
curl -X DELETE http://localhost:3000/api/knowledge-base/documents/<documentId>
```

#### Get Knowledge Base Statistics

```bash
curl http://localhost:3000/api/knowledge-base/stats
```

---

## Review Workflow

### Reviewer Routing

| Category | Reviewer |
|----------|----------|
| technical | debasismohanty07@outlook.com |
| pricing | debasismohanty070@yahoo.com |
| compliance, experience, references, timeline, general | abc_test@gmail.com |

### Approve a Task

```bash
curl -X POST http://localhost:3000/api/reviews/<id>/approve \
  -H "Content-Type: application/json" \
  -d '{"comment": "Looks good, approved for use"}'
```

### Reject a Task

```bash
curl -X POST http://localhost:3000/api/reviews/<id>/reject \
  -H "Content-Type: application/json" \
  -d '{"reason": "Too generic, needs specific pricing details", "comment": "Please add cost breakdown"}'
```

Post-approval, the response is automatically written to:
1. **Cache** — future identical/similar questions return instantly
2. **Organizational Memory** — Researcher agent references this for future questions

---

## Testing

```bash
# Run all tests once
npm test

# Watch mode
npm run test:watch
```

**Test suite: 564 tests** including 28 property-based tests (100+ iterations each via fast-check)

| Category | Files |
|----------|-------|
| Cache Manager | `cache-manager.test.js`, `cache-manager.property.test.js` |
| Knowledge Base | `knowledge-base.test.js`, `knowledge-base.property.test.js` |
| Storage Backend | `storage-backend.test.js`, `storage-backend.property.test.js` |
| Text Extractor | `text-extractor.test.js`, `text-extractor.property.test.js` |
| Similarity Calculator | `similarity.test.js`, `similarity.property.test.js` |
| Response Generator | `response-generator.test.js`, `response-generator.property.test.js` |
| Question Classifier | `classifier.test.js`, `classifier.property.test.js` |
| Server / API | `server.test.js`, `server.property.test.js` |
| Integration | `cache-flow-integration.test.js`, `kb-flow-integration.test.js`, `response-generator-kb-integration.test.js`, `cache-config-integration.test.js` |

---

## Performance

| Scenario | Response Time |
|----------|--------------|
| Exact cache hit | < 50ms |
| Similar cache hit | < 50ms |
| Multi-agent (no KB/memory) | 8–12 seconds |
| Multi-agent (with KB + memory) | 12–20 seconds |
| Bedrock classic RAG | 3–8 seconds |

- Cache reduces LLM API calls by **60–80%** for repeated questions
- Each multi-agent response = 4 LLM calls (one per agent)
- Stateless design supports horizontal scaling

---

## Troubleshooting

### Port Already in Use

```bash
# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# Linux/Mac
lsof -ti:3000 | xargs kill -9
```

Or use `npx kill-port 3000`.

### Configuration Issues

**"Configuration error: Missing required field"**
- OpenAI: set `LLM_PROVIDER`, `OPENAI_API_KEY`, `OPENAI_MODEL`
- Bedrock: set `LLM_PROVIDER`, `AWS_REGION`, `AWS_BEDROCK_MODEL`

**"Invalid or missing credentials"**
- OpenAI: verify key at https://platform.openai.com/ and ensure billing credits are available
- Bedrock: run `aws sts get-caller-identity` to verify credentials

### Cache Issues

**Cache always showing misses** — verify `ENABLE_CACHE=true` in `.env`

**Cache hit rate too low** — lower `CACHE_SIMILARITY_THRESHOLD` (e.g., 0.85 → 0.80)

**Cache clear not working** — ensure parameters are query strings: `?all=true`, not request body

### Knowledge Base Issues

**Responses not using KB context** — check `knowledgeBaseUsed` field; the Planner may have set `needsKB: false` if the question doesn't warrant company-specific context — this is correct behavior

**PDF text extraction fails** — verify PDF is text-based (not scanned) and not password-protected

### AWS Bedrock Issues

**"Model not found" or "Access denied"** — request model access in the AWS Bedrock console and verify `bedrock:InvokeModel` IAM permission

---

## Security Considerations

- API keys stored in `.env` (excluded from git via `.gitignore`)
- Credentials never exposed in HTTP responses or logs
- CORS restricted to `CORS_ORIGINS` — tighten for production
- Input validation: 1–10,000 character limit, 1MB JSON body limit
- Rate limiting: 10 questions/minute per user
- Use HTTPS in production via reverse proxy (nginx/Apache)
- Review audit trail: every approval/rejection timestamped with reviewer comment

---

## Deployment

### Local Development

```bash
npm run dev
```

### Production (PM2)

```bash
npm install -g pm2
pm2 start src/server.js --name rfp-copilot
pm2 save && pm2 startup
```

### Docker

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "src/server.js"]
```

```bash
docker build -t rfp-copilot .
docker run -p 3000:3000 --env-file .env rfp-copilot
```

### AWS + Bedrock AgentCore Deployment (Deployed — 2026-03-21)

The application is deployed on AWS using Bedrock AgentCore as the managed agent runtime and EC2 as the Express server host.

**Architecture:**
```
[Client / AWS Console Sandbox]
         ↓
[Bedrock AgentCore Runtime]   ← agentcore_agent.py (Python, Docker, linux/arm64)
         ↓ HTTP POST :3000
[EC2 t2.micro — rfp-copilot]  ← Node.js Express app (PM2, Amazon Linux 2023)
  Elastic IP: 98.82.229.99
         ↓
[Amazon Bedrock — Nova Pro]   ← Primary LLM (amazon.nova-pro-v1:0, IAM role auth)
         ↓
[Bedrock KB — OG1VQD4BEW]    ← Semantic vector search (Titan Embeddings v2)
  + Guardrail (90zc7vnruh89)  ← Content filtering on KB retrieval
         ↓
[OpenAI API — gpt-4o-mini]   ← Fallback LLM (multi-agent pipeline, optional)
         ↓
[EBS 8GB gp3]                 ← data/ (cache, knowledge, memory, reviews)
```

**AgentCore Entry Point — agentcore_agent.py**

This file was created locally in the project root before running `agentcore configure`. It is the Python wrapper that AgentCore executes inside its managed Docker container. It contains no RFP business logic — it is a pure HTTP proxy that forwards questions to the EC2 Express server.

How it was created:

1. Install the AgentCore Python packages locally:
```bash
pip install bedrock-agentcore-starter-toolkit
pip install bedrock-agentcore
```

2. Create `agentcore_agent.py` in the project root:
```python
from bedrock_agentcore import BedrockAgentCoreApp
import json

app = BedrockAgentCoreApp()

@app.entrypoint
def handle(payload, context):
    question = payload.get("prompt") or payload.get("question", "")
    if not question:
        return {"error": "No question provided"}

    import urllib.request
    import urllib.error

    try:
        data = json.dumps({"question": question}).encode("utf-8")
        req = urllib.request.Request(
            "http://98.82.229.99:3000/api/process-question",
            data=data,
            headers={"Content-Type": "application/json"},
            method="POST"
        )
        with urllib.request.urlopen(req, timeout=60) as resp:
            result = json.loads(resp.read().decode("utf-8"))
            return {
                "response": result.get("data", {}).get("responseText", ""),
                "category": result.get("data", {}).get("categories", [{}])[0].get("category", ""),
                "critiqueScore": result.get("data", {}).get("critiqueScore"),
                "cacheHit": result.get("data", {}).get("cacheHit", False),
                "reviewStatus": result.get("data", {}).get("reviewStatus", "")
            }
    except urllib.error.URLError as e:
        return {"error": f"Failed to reach RFP Copilot server: {str(e)}"}
    except Exception as e:
        return {"error": str(e)}

if __name__ == "__main__":
    app.run()
```

3. Create `requirements.txt` in the project root (AgentCore container dependencies):
```
bedrock-agentcore
requests
```

4. Configure AgentCore (generates `.bedrock_agentcore.yaml` and auto-generates `Dockerfile`):
```powershell
agentcore configure --entrypoint agentcore_agent.py
# Prompts: agent name (Enter), dependency file (Enter), region: us-east-1, role (Enter)
```

5. Deploy to AgentCore (zips source → S3 → CodeBuild → ECR → AgentCore runtime):
```powershell
agentcore deploy
# CodeBuild builds Docker image in AWS cloud (~1m 20s) — no local Docker required
```

6. Test via CLI:
```powershell
agentcore invoke --payload '{\"prompt\": \"What is your pricing policy?\"}'
```

**Key design decisions:**
- `agentcore_agent.py` is a thin proxy — all RFP logic stays in Node.js
- Uses `urllib` (Python stdlib) — no extra install needed inside the container
- URL hardcoded to Elastic IP `98.82.229.99` — static, survives EC2 stop/start
- `timeout=60` — multi-agent pipeline can take up to 20s; 60s gives headroom
- Accepts both `"prompt"` and `"question"` keys — compatible with AgentCore sandbox and direct API calls
- Docker build runs entirely in AWS CodeBuild — Docker Desktop not required locally

**Deployed resources:**

| Resource | Value |
|----------|-------|
| Agent ARN | `arn:aws:bedrock-agentcore:us-east-1:542508027922:runtime/agentcore_agent-KQWbDV81qZ` |
| EC2 Web UI | `http://98.82.229.99:3000` |
| ECR Repository | `542508027922.dkr.ecr.us-east-1.amazonaws.com/bedrock-agentcore-agentcore_agent` |
| AgentCore Memory | `agentcore_agent_mem-CkD59wAwom` |

See `AGENTCORE_DEPLOYMENT_GUIDE.txt` for the full step-by-step deployment log with all commands and outputs.

### Other Cloud Options

- **AWS**: EC2, ECS, Elastic Beanstalk, Lambda + API Gateway
- **Azure**: App Service, Container Instances
- **Google Cloud**: Cloud Run, App Engine
- **Heroku**: `git push heroku main`

---

## Future Enhancements

- LangGraph agent workflow (stateful multi-step graph with self-correction loops)
- Vector embeddings for KB (replace Jaccard with semantic similarity — memory already uses this)
- Confidence-gated escalation — auto-escalate to senior reviewer if confidence < threshold
- Streaming responses — token-level streaming to UI for real-time rendering
- Auto RFP document ingestion (watch folder / S3 trigger)
- Proposal generation (DOCX export)
- Multi-turn conversation (session-level memory across questions)
- Multi-user collaboration with role-based access
- AWS deployment (ElastiCache + OpenSearch + S3 + Bedrock)

---

## License

MIT

## Support

For issues, questions, or contributions, please open an issue in the repository.
