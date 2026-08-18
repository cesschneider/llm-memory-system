# 🧠 Letta (formerly MemGPT) — Deep Research

**Category:** Stateful Agent Platform with Virtual Memory  
**GitHub:** [letta-ai/letta](https://github.com/letta-ai/letta)  
**Stars:** ~36,000 ⭐ (as of mid-2026)  
**License:** Apache 2.0  
**Official Site:** https://www.letta.com  
**Docs:** https://docs.letta.com  

---

## 📋 Summary

Letta (formerly MemGPT) is a platform for building **stateful, self-improving agents** with persistent memory and real computer access. Originally based on the MemGPT paper (2023), which introduced the concept of OS-inspired virtual context management for LLMs, it has since evolved into a full agent infrastructure platform. In 2026, MemGPT was fully absorbed into Letta.

The core insight: treat the LLM like a CPU with limited registers (context window), and build a hierarchical memory paging system — just like a computer OS manages RAM vs disk.

---

## 🌟 Why It Stands Out

- **Pioneered virtual context management** for LLMs (MemGPT paper, 2023)
- **36K GitHub stars** — third largest memory-related repo
- Agents that **learn from their own experience** and improve over time
- **Letta Code**: coding agent with skills, subagents, and persistent memory
- First framework to treat LLM memory as an OS-level concern
- Supports continual learning — memories are updated, not just appended

---

## 🏗️ Architecture

Letta implements an **LLM Operating System** model:

```
┌─────────────────────────────────────────────┐
│              LLM (the "CPU")                 │
│  ┌───────────────────────────────────────┐  │
│  │         Context Window (RAM)           │  │
│  │  ┌──────────┐  ┌──────────────────┐  │  │
│  │  │  System  │  │  Conversation    │  │  │
│  │  │  Prompt  │  │  Buffer          │  │  │
│  │  └──────────┘  └──────────────────┘  │  │
│  └─────────────────────────────────────  │  │
└─────────────────────────────────────────────┘
              │ Memory Paging
              ▼
┌─────────────────────────────────────────────┐
│            External Storage (Disk)           │
│  ┌────────────┐  ┌──────────┐  ┌─────────┐ │
│  │  Archival  │  │  Recall  │  │  Files  │ │
│  │  Memory    │  │  Memory  │  │         │ │
│  │ (semantic) │  │(episodic)│  │         │ │
│  └────────────┘  └──────────┘  └─────────┘ │
└─────────────────────────────────────────────┘
```

**Memory layers:**
- **In-context (working memory):** System prompt + recent conversation
- **Recall memory:** Searchable episodic history of past conversations
- **Archival memory:** Vector-indexed long-term knowledge store
- **File memory:** Document and tool outputs persisted across sessions

---

## 📊 Benchmark Performance

| Benchmark | Letta/MemGPT | Notes |
|-----------|-------------|-------|
| LoCoMo | Competitive | Specific scores vary by model backend |
| DMR (Deep Memory Retrieval) | Created by MemGPT team | Evaluates their own paradigm |
| Context paging efficiency | High | Designed for unbounded context |
| Latency | Higher | Multiple memory tool calls per response |

> Letta's benchmarks are harder to compare directly — their DMR benchmark measures the paging architecture specifically. On LOCOMO/LongMemEval, Letta lags behind Mem0 and Zep for raw recall accuracy, but excels at self-improvement and unbounded context tasks.

---

## 🚀 Installation & Setup

### Via pip

```bash
pip install letta
```

### Run the Letta Server

```bash
# Start the server (runs locally)
letta server

# Or with Docker
docker run -p 8283:8283 lettaai/letta
```

### Create and Use an Agent

```python
from letta import create_client

client = create_client()

# Create a persistent agent
agent = client.create_agent(
    name="my-assistant",
    memory_human="Name: Alice\nJob: Software Engineer",
    memory_persona="You are a helpful AI assistant with excellent memory.",
)

# Chat with the agent
response = client.send_message(
    agent_id=agent.id,
    message="What's my name?",
    role="user"
)
print(response.messages[-1].text)  # "Your name is Alice!"

# The agent persists across sessions
# Next time you run, load the same agent by ID
agent = client.get_agent(agent_id=agent.id)
```

### Letta Code (CLI)

```bash
# Install Letta Code for coding tasks
pip install "letta[code]"

# Run Letta Code agent
letta code
# Agent has pre-built memory, shell access, and subagent skills
```

---

## 🔌 Integrations

| Framework | Support |
|-----------|---------|
| OpenAI | ✅ Default LLM backend |
| Anthropic Claude | ✅ Supported |
| Groq / local models | ✅ Via Ollama, LM Studio |
| LangChain | ⚠️ Indirect (bring-your-own) |
| REST API | ✅ Full HTTP API |
| Python SDK | ✅ Official |

---

## 👤 Ease of Use

| Audience | Rating | Notes |
|----------|--------|-------|
| Non-technical | ⭐⭐ / 5 | Requires server setup; not plug-and-play |
| Python developers | ⭐⭐⭐⭐ / 5 | Good SDK but more concepts to learn |
| Researchers / AI engineers | ⭐⭐⭐⭐⭐ / 5 | Most powerful model for stateful agents |

**Quickest path to working memory:** ~15–20 minutes (server + agent creation)

---

## ✅ Pros

- Most powerful model for truly stateful, self-improving agents
- Unbounded context via memory paging (no hard context window limit)
- Agent learns from its own work history
- Full computer access (files, shell, web) for coding agents
- Large, active community (36K stars)
- First-principles memory architecture

## ❌ Cons

- Higher latency — memory tool calls add roundtrips per response
- More complex to set up than Mem0 or LangMem
- Overkill for simple chatbot memory needs
- DMR benchmark is self-defined; harder to compare with peers
- Self-hosting the server adds infrastructure overhead

---

## 💰 Pricing

| Tier | Price | Notes |
|------|-------|-------|
| Open Source | Free | Self-hosted server |
| Letta Cloud | Free tier | Hosted agents with limits |
| Letta Cloud Pro | Paid | Production-scale hosted agents |
| Enterprise | Custom | Dedicated infrastructure |

---

## 🎯 Best Use Cases

1. **Coding agents** that remember past work and improve over time
2. **Research assistants** building cumulative knowledge
3. **Long-horizon autonomous agents** running over days/weeks
4. **Personalized tutors** adapting to learning progress
5. **AI companions** requiring deep relationship memory

---

## 🔗 Resources

- [GitHub](https://github.com/letta-ai/letta)
- [Official Site](https://www.letta.com)
- [MemGPT Paper (2023)](https://arxiv.org/abs/2310.08560)
- [MemGPT → Letta Announcement](https://www.letta.com/blog/memgpt-and-letta)
- [Letta Documentation](https://docs.letta.com)
