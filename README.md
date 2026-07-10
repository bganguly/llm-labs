# LLM Labs — RAG · pgvector · Multi-Agent Orchestration · MCP

Portfolio of production-shaped LLM application patterns: retrieval-augmented generation over a vector
store, multi-agent orchestration with live reasoning visibility, and MCP server design. Each repo
ships with a `local-dev.sh` entry point, Docker Compose infra, and a `deploy.sh` stub ready for
cloud targeting.

---

| | |
|---|---|
| **LLM APIs** | Anthropic Claude (primary) · OpenAI GPT · NVIDIA NIM / Nemotron (OpenAI-compatible, `base_url` swap) |
| **Orchestration** | LangGraph `StateGraph` · parallel `Send` API · conditional routing · conversation state |
| **Vector search** | pgvector (PostgreSQL 16) · OpenAI `text-embedding-3-small` (1 536 dims) · IVFFlat index · cosine similarity |
| **RAG** | LangChain chunking + retrieval pipeline · context injection · source citation |
| **Streaming** | Token-level SSE via Vercel AI SDK `streamText` · raw FastAPI `StreamingResponse` for agent step events |
| **MCP** | `mcp` Python SDK server over stdio — tools connectable from Claude Desktop, Claude Code, or any MCP client |
| **Backend** | FastAPI 0.115 + asyncio (Python 3.12) |
| **Frontend** | Next.js 15 App Router · React 19 · TypeScript 5.7 · Tailwind CSS |
| **Infra** | Docker Compose · Dockerfiles for backend and frontend · `local-dev.sh` / `infra-down.sh` per repo |

---

## Repos

### 1. RAG + pgvector Demo

**Repository:** [rag-pgvector-demo](https://github.com/bganguly/rag-pgvector-demo)

Ingest any unstructured text → LangChain chunking → OpenAI embeddings → pgvector. Questions trigger
cosine-similarity retrieval; answers stream token-by-token via Vercel AI SDK `streamText`. Provider
toggle in the UI switches between Anthropic, OpenAI, and NVIDIA NIM — same interface, configurable
`base_url`.

```bash
./scripts/local-dev.sh --seed   # starts infra, seeds 6 Wikipedia articles, opens :3010 + :8001
```

```
Browser ──► Next.js :3010 ──► FastAPI :8001 ──► pgvector (postgres :5433)
              Vercel AI SDK       LangChain            IVFFlat cosine index
              streamText          /api/retrieve        1 536-dim embeddings
```

### 2. Multi-Agent Orchestration Demo

**Repository:** [agent-orchestration-demo](https://github.com/bganguly/agent-orchestration-demo)

LangGraph graph classifies each query, optionally decomposes it into parallel sub-queries via the
`Send` API, retrieves context from Wikipedia and DuckDuckGo, then synthesizes a grounded answer.
Every agent step streams to the browser as SSE events — pending / active / done — before the final
answer appears. The same tools are exposed as an MCP server over stdio.

```bash
./scripts/local-dev.sh   # starts infra, opens :3011 + :8002
```

```
Browser ──► Next.js :3011 ──► FastAPI :8002 ──► LangGraph graph
              SSE consumer       /api/agent/run     classify → decompose → retrieve × N → synthesize
              StepTracker UI                        Wikipedia API / DuckDuckGo API

Claude Desktop ──stdio──► app/mcp/server.py ──► same tools
```

### 3. Natural Language to LLM Query Comparison

**Repository:** [natural-language-to-llm-query-comparison](https://github.com/bganguly/natural-language-to-llm-query-comparison)

Side-by-side NL-to-SQL generation across Anthropic and OpenAI models. Generated SQL executes
in-browser via DuckDB-WASM against a Parquet snapshot of DOL H1B LCA disclosures. Schema is
auto-detected from Parquet metadata; no backend or database required.

### 4. Parse LCA Files to Parquet

**Repository:** [parse-lca-files-to-parquet](https://github.com/bganguly/parse-lca-files-to-parquet)

Pipeline that downloads and parses DOL Labor Condition Application disclosure Excel files into a
cleaned Parquet dataset. Powers the default data source for the NL-to-SQL comparison demo.

---

## Local Dev — All Repos

Each repo is self-contained with its own `scripts/local-dev.sh`. Prerequisites across all:

- **Docker** — for postgres/pgvector and redis
- **Python 3.12+**
- **Node 20+**
- **API keys** — `OPENAI_API_KEY` and `ANTHROPIC_API_KEY` (copy `.env.example` → `.env`)

Cloud deploy follows the same single-entry-point pattern as the other repos: `./scripts/deploy.sh`
provisions Artifact Registry, Cloud SQL (RAG demo only), and Cloud Run via Cloud Build — no local
Docker required. See each repo's README for details.
