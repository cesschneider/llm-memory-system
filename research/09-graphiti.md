# 🧠 Graphiti — Deep Research

**Category:** Temporal Knowledge Graph Engine for AI Agents  
**GitHub:** [getzep/graphiti](https://github.com/getzep/graphiti)  
**Stars:** ~6,000 ⭐ (as of mid-2026)  
**License:** Apache 2.0  
**Maintained by:** Zep team  
**Official Docs:** https://help.getzep.com/graphiti  

---

## 📋 Summary

Graphiti is the open-source temporal knowledge graph engine that powers Zep's enterprise memory platform. It can also be used **standalone** as a graph memory backend for any LLM application. Graphiti tracks not just *what* is true, but *when* it became true and *when* it was superseded — making it uniquely suited for applications where facts evolve over time.

It is the most sophisticated graph-based memory solution in the open-source LLM ecosystem.

---

## 🌟 Why It Stands Out

- **Temporal edge support** — every relationship has validity windows (t_start, t_end)
- **Fact supersession** — automatically invalidates outdated facts when new ones arrive
- **Episodic + semantic + community nodes** — multi-granularity memory
- **Sub-200ms retrieval** (Zep's SLA, built on Graphiti)
- Production-proven: powers Zep's enterprise platform used by S&P Market Intelligence
- MCP server available — plug into Claude, Cursor
- **Contradiction detection** — flags when new facts conflict with existing graph nodes

---

## 🏗️ Architecture

```
Conversation / Data Input
           │
           ▼
   Graphiti Ingestion Pipeline
   ┌───────────────────────────────────────┐
   │  Entity Extraction (LLM)             │
   │  Relation Extraction (LLM)           │
   │  Temporal Metadata Assignment        │
   └───────────────────────────────────────┘
           │
           ▼
   ┌───────────────────────────────────────┐
   │         Knowledge Graph               │
   │  Nodes: Entities, Episodes, Facts     │
   │  Edges: Relations with validity range │
   │         (valid_at, invalid_at)        │
   └───────────────────────────────────────┘
           │
   ┌───────┴──────────────┐
   │                      │
Graph Traversal      Vector Search
(structural          (semantic
 queries)             similarity)
   │                      │
   └───────┬──────────────┘
           │
    Hybrid Retrieval → LLM Context
```

**Storage backends:**
- **Neo4j** (production recommended)
- **FalkorDB** (self-hosted, fast)
- In-memory (dev/testing)

---

## 📊 Benchmark Performance

| Metric | Graphiti/Zep | Notes |
|--------|-------------|-------|
| LongMemEval (temporal) | **63.8%** | Best-in-class temporal recall |
| LoCoMo F1 | **49.56** | Tied for highest open-domain score |
| Retrieval latency (Zep) | < 200ms | Production SLA |
| Graph build latency | Moderate | Background processing, not real-time |

---

## 🚀 Installation & Setup

### Install

```bash
pip install graphiti-core
```

### Requirements

```bash
# Neo4j (recommended for production)
docker run -p 7474:7474 -p 7687:7687 neo4j:latest

# Or FalkorDB (lighter alternative)
docker run -p 6379:6379 falkordb/falkordb:latest
```

### Minimal Example

```python
import asyncio
from graphiti_core import Graphiti
from graphiti_core.nodes import EpisodeType
from datetime import datetime, timezone

async def main():
    # Connect to Neo4j
    graphiti = Graphiti(
        neo4j_uri="bolt://localhost:7687",
        neo4j_user="neo4j",
        neo4j_password="password",
    )

    # Initialize the graph schema
    await graphiti.build_indices_and_constraints()

    # Add an episodic memory
    await graphiti.add_episode(
        name="user_session_001",
        episode_body="Alice told me she works as a software engineer at Acme Corp.",
        source_description="User conversation",
        reference_time=datetime.now(timezone.utc),
        source=EpisodeType.message,
    )

    # Add another episode with updated info
    await graphiti.add_episode(
        name="user_session_002",
        episode_body="Alice mentioned she recently changed jobs and now works at BetaStartup.",
        source_description="User conversation",
        reference_time=datetime.now(timezone.utc),
        source=EpisodeType.message,
    )
    # Graphiti automatically invalidates the old employer relation

    # Search the graph
    results = await graphiti.search("Where does Alice work?")
    for r in results:
        print(r.fact)  # "Alice works at BetaStartup" (old fact invalidated)

asyncio.run(main())
```

### MCP Server Setup

```bash
# Install Graphiti MCP
pip install graphiti-mcp

# Run MCP server (connects to your Neo4j instance)
graphiti-mcp --neo4j-uri bolt://localhost:7687 \
             --neo4j-user neo4j \
             --neo4j-password password
```

Add to Claude Desktop / Cursor config:
```json
{
  "mcpServers": {
    "graphiti": {
      "command": "graphiti-mcp",
      "args": ["--neo4j-uri", "bolt://localhost:7687",
               "--neo4j-user", "neo4j",
               "--neo4j-password", "your-password"]
    }
  }
}
```

---

## 🔌 Integrations

| Framework | Support |
|-----------|---------|
| Zep (cloud) | ✅ Powers Zep enterprise platform |
| Claude (MCP) | ✅ MCP server available |
| Cursor (MCP) | ✅ MCP server available |
| LangChain | ✅ Community integration |
| OpenAI | ✅ Via LLM config |
| Anthropic | ✅ Via LLM config |
| FastAPI / direct | ✅ Python SDK |

---

## 👤 Ease of Use

| Audience | Rating | Notes |
|----------|--------|-------|
| Non-technical | ⭐ / 5 | Requires Docker + Neo4j setup |
| Experienced developers | ⭐⭐⭐⭐ / 5 | Excellent for those comfortable with graphs |
| AI/ML engineers | ⭐⭐⭐⭐⭐ / 5 | Best-in-class for temporal graph memory |

**Quickest path to working memory:** ~30 minutes (Docker + Neo4j + Python setup)

---

## ✅ Pros

- Best temporal knowledge graph in open source
- Fact supersession — automatically invalidates outdated beliefs
- Contradiction detection built-in
- Sub-200ms retrieval (when used via Zep cloud)
- Production-proven at enterprise scale
- MCP-native for Claude and Cursor
- Apache 2.0 license

## ❌ Cons

- Most complex setup of any tool in this list
- Requires Docker + Neo4j or FalkorDB
- Background graph processing adds ingestion latency
- Overkill for simple Q&A memory use cases
- Not for non-technical users in self-hosted mode
- Team/community smaller than Mem0

---

## 💰 Pricing

| Option | Price | Notes |
|--------|-------|-------|
| graphiti-core | Free | Open source |
| Neo4j Community | Free | Self-hosted |
| Neo4j AuraDB | Paid | Managed Neo4j cloud |
| FalkorDB | Free (self-hosted) | Redis-protocol compatible |
| Zep Cloud (uses Graphiti) | Paid | Fully managed |

---

## 🎯 Best Use Cases

1. **CRM / sales intelligence** — tracking how relationships and facts evolve
2. **Healthcare AI** — patient records where facts change over time
3. **Financial agents** — portfolio changes, market condition updates
4. **Long-horizon research agents** — building durable knowledge bases
5. **Any app where "what was true" matters as much as "what is true"**

---

## 🔗 Resources

- [GitHub (graphiti-core)](https://github.com/getzep/graphiti)
- [Zep (uses Graphiti)](https://www.getzep.com)
- [Documentation](https://help.getzep.com/graphiti)
- [MCP Setup for Claude](https://github.com/Flo976/graphiti-mcp-ollama)
- [Claude Code + Graphiti Setup](https://github.com/lucasrosati/claude-code-memory-setup)
