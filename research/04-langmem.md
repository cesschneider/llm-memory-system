# 🧠 LangMem — Deep Research

**Category:** Memory Library for LangGraph / LangChain Agents  
**GitHub:** [langchain-ai/langmem](https://github.com/langchain-ai/langmem)  
**Stars:** ~2,000 ⭐ (as of mid-2026)  
**License:** MIT  
**Official Docs:** https://langchain-ai.github.io/langmem/  
**Part of:** LangChain / LangGraph ecosystem  

---

## 📋 Summary

LangMem is LangChain's official memory library for LangGraph agents. It provides tools for agents to **learn and adapt from interactions over time** — storing, searching, and updating memories using LangGraph's persistent store abstraction. It is the go-to choice for anyone already building on the LangChain/LangGraph stack.

---

## 🌟 Why It Stands Out

- **Official LangChain memory solution** — deeply integrated, first-class support
- **Clean API**: two tools (`manage_memory`, `search_memory`) cover all use cases
- Works with any LangGraph-compatible store (in-memory, Redis, PostgreSQL, etc.)
- **Background memory consolidation** — reflexion-based learning between sessions
- Supports both **in-thread** (reactive) and **cross-thread** (background) memory patterns
- Compatible with CrewAI, AutoGen, and other agent frameworks

---

## 🏗️ Architecture

LangMem uses LangGraph's `BaseStore` as its persistence abstraction:

```
Agent Conversation
        │
        ▼
  Memory Tools (in-context)
  ┌──────────────────────┐
  │  manage_memory_tool  │ → Create / update / delete memories
  │  search_memory_tool  │ → Semantic search over stored memories
  └──────────────────────┘
        │
        ▼
  LangGraph BaseStore
  ┌─────────────────────────────────────┐
  │  InMemoryStore (dev/testing)        │
  │  PostgresStore (production)         │
  │  RedisStore (high-performance)      │
  │  Custom backends (plug-in)          │
  └─────────────────────────────────────┘

Background Process (optional):
  ReflectionExecutor → Consolidates memories between sessions
```

**Memory types supported:**
- **Semantic memory**: facts, preferences, knowledge
- **Episodic memory**: conversation history summaries
- **Procedural memory**: instructions and system prompts that evolve

---

## 📊 Benchmark Performance

| Benchmark | LangMem | Notes |
|-----------|---------|-------|
| LoCoMo | Competitive | Depends on underlying store and embedder |
| LongMemEval | Not published separately | Benchmark scores vary by config |
| Latency | Low | No background processing by default |
| Token efficiency | High | Minimal overhead over raw LangGraph |

> LangMem's performance is highly configurable — results depend on the embedding model, store backend, and retrieval strategy used.

---

## 🚀 Installation & Setup

### Install

```bash
pip install -U langmem
```

### Environment Setup

```bash
export ANTHROPIC_API_KEY="sk-..."   # or OPENAI_API_KEY
export LANGSMITH_API_KEY="..."      # optional, for tracing
```

### Quickstart (In-Context Memory)

```python
from langgraph.prebuilt import create_react_agent
from langgraph.store.memory import InMemoryStore
from langmem import create_manage_memory_tool, create_search_memory_tool

# Set up storage with embedding index
store = InMemoryStore(
    index={
        "dims": 1536,
        "embed": "openai:text-embedding-3-small",
    }
)

# Create agent with memory tools
agent = create_react_agent(
    "anthropic:claude-3-5-sonnet-latest",
    tools=[
        create_manage_memory_tool(namespace=("memories",)),
        create_search_memory_tool(namespace=("memories",)),
    ],
    store=store,
)

# The agent now decides when to save and retrieve memories
response = agent.invoke({
    "messages": [{"role": "user", "content": "My name is Alice and I love Python."}]
})
```

### Background Memory (Cross-Session Learning)

```python
from langmem import ReflectionExecutor, create_memory_store_manager
from langchain.chat_models import init_chat_model

llm = init_chat_model("claude-3-5-sonnet-latest", model_provider="anthropic")

# Manager extracts structured memories from conversations
manager = create_memory_store_manager(
    llm,
    namespace=("users", "{user_id}"),
    store=store,
)

# ReflectionExecutor runs background consolidation
executor = ReflectionExecutor(manager)

# After a conversation, schedule background reflection
executor.submit(
    {"messages": conversation_history},
    after_seconds=10,  # consolidate 10 seconds after last message
    config={"configurable": {"user_id": "alice"}}
)
```

### CrewAI Integration

```python
from crewai import Agent, Crew, Task
from langgraph.store.memory import InMemoryStore
from langmem import create_manage_memory_tool, create_search_memory_tool

store = InMemoryStore(index={"dims": 1536, "embed": "openai:text-embedding-3-small"})

tools = [
    create_manage_memory_tool(namespace="memories", store=store),
    create_search_memory_tool(namespace="memories", store=store),
]

agent = Agent(
    role="Knowledge Learner",
    goal="Learn and store new information",
    backstory="You build a knowledge base from every conversation",
    tools=tools,
)
```

---

## 🔌 Integrations

| Framework | Support |
|-----------|---------|
| LangGraph | ✅ Native — uses BaseStore directly |
| LangChain | ✅ Official package |
| CrewAI | ✅ Documented guide |
| AutoGen | ⚠️ Possible via tool wrapping |
| OpenAI | ✅ Via LLM config |
| Anthropic | ✅ Via LLM config |
| Any LangGraph store | ✅ PostgreSQL, Redis, custom |

---

## 👤 Ease of Use

| Audience | Rating | Notes |
|----------|--------|-------|
| Non-technical | ⭐⭐ / 5 | Requires Python + LangGraph knowledge |
| LangChain developers | ⭐⭐⭐⭐⭐ / 5 | Perfect fit, minimal learning curve |
| Other framework users | ⭐⭐⭐ / 5 | Works but requires some adaptation |

**Quickest path:** ~10 minutes if you already use LangGraph

---

## ✅ Pros

- First-class LangChain/LangGraph integration
- Clean two-tool API (manage + search)
- Background reflection for proactive memory consolidation
- Namespace-based multi-user / multi-agent memory isolation
- Works with any persistent store backend
- Part of LangChain's actively maintained ecosystem

## ❌ Cons

- Tightly coupled to LangGraph's patterns — harder to use standalone
- Fewer independent benchmarks than Mem0 or Zep
- Background processing adds complexity for simple use cases
- Memory quality depends heavily on the embedding model chosen
- Smaller community than Mem0 or Letta

---

## 💰 Pricing

LangMem itself is **free and open source** (MIT license).  
Costs come from:
- LLM API calls (for extraction and search)
- Embedding model API calls
- Your chosen store backend (e.g. PostgreSQL hosting, Redis)

---

## 🎯 Best Use Cases

1. **LangGraph agents** requiring persistent memory
2. **Multi-session chatbots** on the LangChain stack
3. **Background learning agents** that consolidate knowledge overnight
4. **CrewAI teams** sharing a knowledge base
5. Any team **already invested in LangChain** tooling

---

## 🔗 Resources

- [GitHub](https://github.com/langchain-ai/langmem)
- [Documentation](https://langchain-ai.github.io/langmem/)
- [Background Quickstart](https://langchain-ai.github.io/langmem/background_quickstart/)
- [Memory Tools Guide](https://langchain-ai.github.io/langmem/guides/memory_tools/)
- [CrewAI Integration Guide](https://langchain-ai.github.io/langmem/guides/use_tools_in_crewai/)
