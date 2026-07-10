# LLM Implementations

Portfolio of LLM application patterns — RAG pipelines, vector search, multi-agent orchestration, and provider integrations.

---

| | |
|---|---|
| **LLM APIs** | Anthropic Claude, OpenAI GPT, NVIDIA NIM / Nemotron (OpenAI-compatible) |
| **Orchestration** | LangChain, LangGraph, MCP server |
| **Vector search** | pgvector (PostgreSQL), cosine similarity, semantic chunking |
| **Streaming** | Token-level SSE via Vercel AI SDK and raw FastAPI `StreamingResponse` |
| **Backend** | FastAPI + asyncio (Python 3.12) |
| **Frontend** | Next.js 15 App Router, React 19, TypeScript |
| **Infra** | Docker Compose, GitHub Actions |

---

## Projects

### 1. RAG + pgvector Demo

**Repository:** [rag-pgvector-demo](https://github.com/bganguly/rag-pgvector-demo)

- Ingest any unstructured text → chunk via LangChain `RecursiveCharacterTextSplitter` → embed → store in pgvector
- Query: cosine similarity retrieval → context injection → token-level SSE streaming via Vercel AI SDK `streamText`
- Provider toggle: Anthropic / OpenAI / NVIDIA NIM (Nemotron) — same interface, configurable `base_url`
- FastAPI handles vector operations; Next.js API routes handle LLM streaming
- Docker Compose: `pgvector/pgvector:pg16`, Redis; seed script pulls Wikipedia articles via REST API

```
Browser ──► Next.js :3010 ──► FastAPI :8001
              Vercel AI SDK       pgvector (postgres :5433)
              streamText          LangChain pipeline
```

### 2. Multi-Agent Orchestration Demo

**Repository:** [agent-orchestration-demo](https://github.com/bganguly/agent-orchestration-demo)

- LangGraph `StateGraph`: classify → decompose (parallel `Send`) → retrieve → synthesize
- Simple queries take the direct path; complex queries decompose into parallel sub-queries
- Tools: `wikipedia_search`, `duckduckgo_search` — no API key required, live data
- MCP server (`app/mcp/server.py`) exposes the same tools over stdio for Claude Desktop integration
- Agent steps streamed to the browser as SSE events; frontend shows each node live

```
Browser ──► Next.js :3011 ──► FastAPI :8002
              SSE step events     LangGraph graph
                                  classify → decompose → retrieve (parallel) → synthesize
                                  MCP server (stdio) ← Claude Desktop
```

### 3. Natural Language to LLM Query Comparison

**Repository:** [natural-language-to-llm-query-comparison](https://github.com/bganguly/natural-language-to-llm-query-comparison)

- Compare NL-to-SQL generation across Anthropic and OpenAI models side-by-side
- Executes generated SQL in-browser via DuckDB-WASM against any Parquet endpoint
- Schema auto-detection from Parquet metadata; no backend required
- React 19 + Vite

### 4. Parse LCA Files to Parquet

**Repository:** [parse-lca-files-to-parquet](https://github.com/bganguly/parse-lca-files-to-parquet)

- Pipeline that parses Labor Condition Application (LCA) disclosure files into Parquet format
- Powers the default data source for the NL-to-SQL comparison demo

---

## Architecture Notes

The RAG demo and agent demo are intentionally separate: the first shows the retrieval/vector layer in depth, the second shows the orchestration layer. The agent's `retrieve` tool can be wired to the RAG service's `/api/retrieve` endpoint to compose them.

The MCP server in the agent demo means the same tools are accessible from Claude Desktop, Claude Code, or any MCP-compatible client — the tool layer is client-agnostic.
