# 🧠 Cognee — Deep Research

**Category:** Open-Source AI Memory Platform (Knowledge Graph)  
**GitHub:** [topoteretes/cognee](https://github.com/topoteretes/cognee)  
**Stars:** ~4,000 ⭐ (as of mid-2026)  
**License:** Apache 2.0  
**Official Site:** https://www.cognee.ai  
**Docs:** https://docs.cognee.ai  

---

## 📋 Summary

Cognee is an open-source AI memory platform that combines **vector search** and **knowledge graphs** to give agents persistent, structured long-term memory. Unlike pure vector stores, Cognee continuously builds a self-hosted knowledge graph from ingested data — enabling relational queries, ontology grounding, and structured understanding that goes beyond semantic similarity search.

Reported accuracy: **92.5% on complex knowledge graph scenarios** (vendor benchmark).

---

## 🌟 Why It Stands Out

- **Dual-engine**: vector embeddings + knowledge graph in one unified API
- **`cognee.remember()`** — a single method to permanently store any information
- Supports ingestion of PDFs, text, URLs, code, structured data
- **MCP (Model Context Protocol) server** — plug into Claude, Cursor, any MCP client
- Self-hosted and fully local — no data leaves your infrastructure
- Multimodal support (text, documents, structured data)
- Cross-agent knowledge sharing

---

## 🏗️ Architecture

```
Data Sources (text, PDF, URL, code, DB)
              │
              ▼
        Cognee Ingestion
      ┌────────┴────────┐
      │                 │
   Cognify          Chunking &
  (LLM-based        Embedding
  extraction)
      │                 │
      ▼                 ▼
Knowledge Graph    Vector Index
(entities,         (semantic
 relations,         similarity)
 ontologies)
      │                 │
      └────────┬────────┘
               │
         Hybrid Search
               │
               ▼
      Context for LLM / Agent
```

**Storage backends:**
- Vector: Qdrant, Weaviate, PGVector, ChromaDB, LanceDB
- Graph: Neo4j, FalkorDB, NetworkX (local)
- Relational: SQLite (default), PostgreSQL

---

## 📊 Benchmark Performance

| Metric | Cognee | Notes |
|--------|--------|-------|
| Complex knowledge graph accuracy | **92.5%** | Vendor benchmark |
| Retrieval precision | High | Graph-guided retrieval improves precision |
| MCP compatibility | ✅ | Native MCP server available |
| Self-hosted latency | Moderate | Graph processing adds overhead |

> Note: 92.5% accuracy is reported by Cognee's team on their internal knowledge graph benchmark. Independent cross-tool comparisons (LoCoMo, LongMemEval) are not yet widely published for Cognee.

---

## 🚀 Installation & Setup

### Install

```bash
# Recommended: use uv
uv pip install cognee

# Or standard pip
pip install cognee
```

### Minimal Example

```python
import cognee
import asyncio

async def main():
    # Configure LLM
    cognee.config.llm_api_key = "your-openai-api-key"

    # Store information permanently in the knowledge graph
    # (runs add + cognify + improve internally)
    await cognee.remember("Cognee turns documents into AI memory.")

    # Search the knowledge graph
    results = await cognee.search("What does Cognee do?")
    for result in results:
        print(result)

asyncio.run(main())
```

### Full Pipeline

```python
import cognee
import asyncio

async def main():
    # Reset for fresh start
    await cognee.prune.prune_data()
    await cognee.prune.prune_system()

    # Add documents / text
    await cognee.add("path/to/document.pdf")
    await cognee.add("https://example.com/article")
    await cognee.add("The capital of France is Paris.")

    # Process and build the knowledge graph
    await cognee.cognify()

    # Search
    results = await cognee.search(
        query_text="What is the capital of France?",
        query_type="INSIGHTS"  # or "CHUNKS", "GRAPH_COMPLETION"
    )
    print(results)

asyncio.run(main())
```

### MCP Server Setup (for Claude, Cursor, etc.)

```json
// claude_desktop_config.json or .cursor/mcp.json
{
  "mcpServers": {
    "cognee": {
      "command": "uvx",
      "args": ["cognee-mcp"],
      "env": {
        "OPENAI_API_KEY": "your-key",
        "DB_PROVIDER": "sqlite"
      }
    }
  }
}
```

Once connected, Claude or Cursor can use Cognee to persist and retrieve knowledge across sessions.

---

## 🔌 Integrations

| Framework | Support |
|-----------|---------|
| Claude (MCP) | ✅ Native MCP server |
| Cursor (MCP) | ✅ Native MCP server |
| OpenAI | ✅ Default LLM backend |
| LangChain | ✅ Community integration |
| LlamaIndex | ✅ Integration available |
| Anthropic | ✅ Via LLM config |
| Ollama (local) | ✅ Local LLM support |

---

## 👤 Ease of Use

| Audience | Rating | Notes |
|----------|--------|-------|
| Non-technical | ⭐⭐ / 5 | MCP install is simple; raw API needs Python |
| Python developers | ⭐⭐⭐⭐ / 5 | Clean async API, good docs |
| AI/ML engineers | ⭐⭐⭐⭐⭐ / 5 | Excellent for structured knowledge tasks |

**Quickest path (via MCP):** ~5 minutes to plug into Claude Desktop

---

## ✅ Pros

- Knowledge graph + vector hybrid (best structured recall)
- Fully self-hosted — data never leaves your machine
- MCP-native — easy to plug into Claude, Cursor, other MCP clients
- Ingests diverse sources: PDFs, URLs, code, structured data
- Multimodal support
- Cross-agent knowledge sharing via shared graph
- Active development team

## ❌ Cons

- Async-only API (requires asyncio)
- Graph processing can be slow on large datasets
- Setup more complex than Mem0 for simple use cases
- Independent benchmarks (LoCoMo, LongMemEval) not yet widely published
- Smaller community than Mem0 or Letta

---

## 💰 Pricing

| Tier | Price | Notes |
|------|-------|-------|
| Open Source | Free | Fully self-hosted |
| Cognee Cloud | Coming soon | Managed hosting |

---

## 🎯 Best Use Cases

1. **Document-heavy agents** processing PDFs, reports, contracts
2. **Research assistants** building structured knowledge graphs
3. **Claude / Cursor users** wanting persistent cross-session knowledge
4. **Multi-agent systems** sharing a unified knowledge base
5. **Ontology-grounded applications** (medical, legal, scientific)

---

## 🔗 Resources

- [GitHub](https://github.com/topoteretes/cognee)
- [Official Site](https://www.cognee.ai)
- [Documentation](https://docs.cognee.ai)
- [MCP Integration Guide](https://medium.com/@cognee/cognee-model-context-protocol-cognee-llm-memory-made-simple-795f1247b237)
- [Dev.to Overview](https://dev.to/om_shree_0709/cognee-building-the-next-generation-of-memory-for-ai-agents-oss-3jm1)
