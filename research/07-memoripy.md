# 🧠 Memoripy — Deep Research

**Category:** Lightweight AI Memory Layer (Python)  
**GitHub:** [caspianmoon/memoripy](https://github.com/caspianmoon/memoripy)  
**Stars:** ~1,500 ⭐ (as of mid-2026)  
**License:** MIT  
**Install:** `pip install memoripy`  

---

## 📋 Summary

Memoripy is a lightweight, human-inspired AI memory layer for Python applications. It implements **short-term and long-term memory** with semantic clustering and optional **memory decay** — modeled on how human memory fades over time without reinforcement. It is one of the easiest memory libraries to install and use, making it accessible to developers and semi-technical users alike.

---

## 🌟 Why It Stands Out

- **Human-like memory decay** — memories fade over time (configurable, like Ebbinghaus forgetting curve)
- **Simplest installation**: `pip install memoripy` — zero infra
- Short-term + long-term memory in a single package
- Semantic clustering groups related memories for smarter retrieval
- **In-process storage** — no external database required by default
- JSON file persistence option for simple deployment
- OpenAI and Ollama (local) support

---

## 🏗️ Architecture

```
Conversation Input
        │
        ▼
  Memory Manager
  ┌─────────────────────────────────────┐
  │  Short-Term Memory Buffer           │
  │  (recent N interactions, in-RAM)    │
  └──────────────┬──────────────────────┘
                 │ consolidation trigger
                 ▼
  ┌─────────────────────────────────────┐
  │  Long-Term Memory Store             │
  │  ┌──────────────┐ ┌──────────────┐ │
  │  │  Embeddings  │ │  Decay Model │ │
  │  │  (semantic   │ │  (relevance  │ │
  │  │   clustering)│ │   weighting) │ │
  │  └──────────────┘ └──────────────┘ │
  └─────────────────────────────────────┘
        │
        ▼
  Relevant Memory Retrieval
  (recency + semantic + importance scores)
```

**Memory scoring formula:**
```
score = α × semantic_similarity + β × recency_weight + γ × importance
```

Where weights (α, β, γ) are configurable and decay over time.

---

## 📊 Performance

| Metric | Memoripy | Notes |
|--------|---------|-------|
| Setup time | < 2 minutes | pip install, no infra |
| Memory footprint | Low | In-process, JSON storage |
| Semantic search quality | Good | Depends on embedder choice |
| Decay modeling | ✅ Native | Ebbinghaus-inspired |
| Benchmark scores | Not published | No LoCoMo/LongMemEval data |

> Memoripy is not designed for enterprise-scale production use. It excels at simplicity and human-like memory behavior in small-to-medium applications.

---

## 🚀 Installation & Setup

### Install

```bash
pip install memoripy
```

### Environment

```bash
export OPENAI_API_KEY="your-api-key"
# Or use Ollama for local (no API key)
```

### Full Example

```python
from memoripy import MemoryManager, JSONStorage

# Choose storage: JSONStorage (file-based) or InMemoryStorage
storage = JSONStorage("memory_store.json")

# Initialize memory manager
# Supports: "openai" or "ollama" for chat + embedding
memory = MemoryManager(
    api_key="your-openai-api-key",  # omit if using Ollama
    chat_model="gpt-4o-mini",
    chat_model_provider="openai",
    embedding_model="mxbai-embed-large",
    embedding_model_provider="ollama",  # or "openai"
    storage=storage
)

# Store a new interaction (input + response pair)
memory.add_interaction(
    prompt="My favorite programming language is Python.",
    output="Got it! I'll remember you prefer Python."
)

# Load relevant past memories for a new prompt
relevant_memories, short_term, long_term = memory.load_memory_variables(
    query="What programming languages does the user prefer?",
    recent_interactions_limit=5,
    similarity_threshold=0.7
)

# Use retrieved memories as context
for mem in relevant_memories:
    print(f"[Memory] {mem['prompt']} → {mem['output']}")
```

### With Ollama (Fully Local — No API Key)

```python
from memoripy import MemoryManager, JSONStorage

storage = JSONStorage("local_memory.json")

memory = MemoryManager(
    chat_model="llama3.2",
    chat_model_provider="ollama",
    embedding_model="mxbai-embed-large",
    embedding_model_provider="ollama",
    storage=storage
)
```

---

## 🔌 Integrations

| Framework | Support |
|-----------|---------|
| OpenAI | ✅ Native |
| Ollama (local LLMs) | ✅ Native |
| Any Python app | ✅ Library, works anywhere |
| LangChain | ⚠️ Manual wrapping |
| FastAPI / Flask | ✅ Easy to integrate |

---

## 👤 Ease of Use

| Audience | Rating | Notes |
|----------|--------|-------|
| Non-technical | ⭐⭐⭐ / 5 | Requires Python basics, but very simple |
| Python beginners | ⭐⭐⭐⭐ / 5 | Clear API, great for learning |
| Experienced developers | ⭐⭐⭐ / 5 | Good for prototyping; not production-scale |

**Quickest path to working memory:** ~5 minutes

---

## ✅ Pros

- Simplest real memory library to install and use
- Human-inspired decay makes memory feel natural
- Works fully offline with Ollama (no API costs)
- JSON file storage — no database setup
- Great for prototyping and learning
- MIT license — fully free

## ❌ Cons

- Not designed for production scale (thousands of users)
- No built-in knowledge graph
- No temporal fact tracking
- No multi-user namespacing out of the box
- Memory quality depends on external embedder
- Not benchmarked on standard evals (LoCoMo, LongMemEval)

---

## 💰 Pricing

**Completely free** (MIT license). Costs only from:
- OpenAI API calls (if used)
- Otherwise zero — Ollama backend is fully local/free

---

## 🎯 Best Use Cases

1. **Personal AI assistants** with natural forgetting behavior
2. **Prototyping** memory-augmented chatbots quickly
3. **Educational projects** demonstrating memory concepts
4. **Offline / local LLM** applications (with Ollama)
5. **Solo developer tools** where scale isn't a concern

---

## 🔗 Resources

- [GitHub](https://github.com/caspianmoon/memoripy)
- [PyPI](https://pypi.org/project/memoripy/)
- [YouTube Demo](https://www.youtube.com/watch?v=roUBR2ElKiw)
- [AI Tool Review](https://eliteai.tools/tool/memoripy)
