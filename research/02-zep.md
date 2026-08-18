# 🧠 Zep — Deep Research

**Category:** Enterprise LLM Memory Platform  
**GitHub:** [getzep/zep](https://github.com/getzep/zep)  
**Stars:** ~3,500 ⭐ (as of mid-2026)  
**License:** Apache 2.0 (community) / Commercial (cloud)  
**Official Site:** https://www.getzep.com  
**Docs:** https://help.getzep.com  

---

## 📋 Summary

Zep is a production-grade, enterprise-focused memory platform that combines **vector search** with **temporal knowledge graphs** for AI agents. It is purpose-built for long-running sessions with complex factual requirements — particularly where facts evolve over time (e.g. "user changed their address" or "company was acquired"). It is SOC 2 Type II certified, making it suitable for regulated industries.

---

## 🌟 Why It Stands Out

- **Best temporal reasoning**: 63.8% LongMemEval score (vs Mem0's 49.0% in the original eval)
- **Hybrid architecture**: vector embeddings + Graphiti knowledge graph
- **Enterprise-ready**: SOC 2 Type II, sub-200ms retrieval SLA
- **Validated by S&P Market Intelligence**
- Fact supersession: knows when a user's old data has been replaced by new data

---

## 🏗️ Architecture

Zep uses **Graphiti** (its open-source temporal knowledge graph engine) under the hood alongside vector embeddings:

```
Conversation Input
       │
       ▼
  Context Extractor
  ┌────┴────────────────┐
  │                     │
Vector Index         Graphiti Graph
(semantic search)    (temporal facts,
                      entity relations,
                      validity windows)
  │                     │
  └────────┬────────────┘
           │
    Hybrid Retriever
           │
           ▼
  Context-Enriched Agent Response
```

**Key architectural differentiator:** Graphiti tracks **when** a fact became true and **when** it was superseded — critical for domains like healthcare, finance, and CRM where data changes.

---

## 📊 Benchmark Performance

| Benchmark | Zep Score | Notes |
|-----------|-----------|-------|
| LongMemEval (temporal) | **63.8%** | Best-in-class for temporal reasoning |
| LoCoMo F1 (open-domain) | **49.56** (highest) | Narrowly beats Mem0 |
| LoCoMo J score | **76.60** | Top score on this metric |
| p95 Retrieval Latency | < 200ms | SLA-guaranteed |
| Memory footprint | 600K+ tokens | Higher than Mem0 (trade-off for graph richness) |

> Zep and Mem0 trade performance leadership depending on the benchmark. Zep dominates temporal tasks; Mem0 leads on token efficiency and general conversation recall.

---

## 🚀 Installation & Setup

### Cloud (Recommended for most users)

```bash
pip install zep-cloud
```

```python
from zep_cloud.client import Zep

client = Zep(api_key="your-zep-api-key")

# Add a session
session_id = "session_abc"
client.memory.add_session(session_id=session_id, user_id="alice")

# Add messages
from zep_cloud.types import Message
messages = [
    Message(role="user", role_type="user", content="My name is Alice and I live in Seattle."),
    Message(role="assistant", role_type="assistant", content="Nice to meet you, Alice!"),
]
client.memory.add(session_id, messages=messages)

# Search memory
results = client.memory.search_sessions(
    user_id="alice",
    text="Where does the user live?"
)
for r in results.results:
    print(r.fact)
```

### Self-Hosted (Community Edition)

```bash
# Via Docker Compose
git clone https://github.com/getzep/zep.git
cd zep
docker-compose up
```

### LangChain Integration

```python
from langchain_community.memory import ZepMemory

memory = ZepMemory(
    session_id="session_abc",
    url="https://api.getzep.com",
    api_key="your-api-key",
    memory_key="chat_history",
)
```

---

## 🔌 Integrations

| Framework | Support |
|-----------|---------|
| LangChain | ✅ Official ZepMemory class |
| LangGraph | ✅ Store adapter |
| OpenAI | ✅ Direct API compatible |
| Anthropic Claude | ✅ Compatible |
| AutoGen | ✅ Supported |
| Flowise | ✅ Node available |
| n8n | ✅ Community node |

---

## 👤 Ease of Use

| Audience | Rating | Notes |
|----------|--------|-------|
| Non-technical | ⭐⭐ / 5 | Requires API key + some setup; not zero-config |
| Python developers | ⭐⭐⭐⭐ / 5 | Clean SDK, good docs |
| Enterprise teams | ⭐⭐⭐⭐⭐ / 5 | Best enterprise DX; SOC 2, SLAs, dedicated support |

**Quickest path to working memory:** ~10–15 minutes with cloud API key

---

## ✅ Pros

- Best temporal reasoning (fact validity windows, supersession)
- SOC 2 Type II certified — suitable for regulated industries
- Sub-200ms retrieval SLA
- Knowledge graph enables complex relationship queries
- Business Data API — inject CRM, product, company data into memory
- Strong LangChain ecosystem integration

## ❌ Cons

- Memory footprint: 600K+ tokens per conversation (vs Mem0's 1,764)
- Immediate post-ingestion retrieval can fail — graph processing has latency
- More complex setup than Mem0 for simple use cases
- Smaller GitHub community than Mem0
- Graph processing adds background latency

---

## 💰 Pricing

| Tier | Price | Notes |
|------|-------|-------|
| Community (self-hosted) | Free | Open source, self-managed |
| Cloud Developer | Free tier available | Limited sessions |
| Cloud Production | Pay-per-session | Enterprise pricing |
| Enterprise | Custom | SOC 2, SLAs, dedicated support |

---

## 🎯 Best Use Cases

1. **CRM / Sales AI** — tracking evolving customer data over time
2. **Healthcare AI** — patient data that changes (medications, diagnoses)
3. **Financial advisors** — portfolio and preference changes
4. **Long-running agent sessions** — multi-week or multi-month interactions
5. **Enterprise chatbots** — compliance-required memory with audit trails

---

## 🔗 Resources

- [GitHub](https://github.com/getzep/zep)
- [Official Site](https://www.getzep.com)
- [Documentation](https://help.getzep.com)
- [Graphiti (the underlying graph engine)](https://github.com/getzep/graphiti)
- [Benchmark Comparison](https://atlan.com/know/best-ai-agent-memory-frameworks-2026/)
