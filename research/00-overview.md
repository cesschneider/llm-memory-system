# 📊 LLM Memory Systems — Research Overview

**Phase:** 1 — Tool Landscape Survey  
**Last Updated:** 2026-08  
**Scope:** Top 10 memory plugins, tools, and frameworks for LLM-based systems

---

## 🎯 Research Objective

Identify, evaluate, and document the leading memory solutions available for LLM-based systems. Selection criteria:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| **Popularity** | 35% | GitHub stars, community size, adoption |
| **Ease of Use** | 35% | Setup complexity, documentation quality, non-technical accessibility |
| **Performance** | 30% | Benchmark scores (LoCoMo, LongMemEval, BEAM), latency, token efficiency |

---

## 🏆 Top 10 — At a Glance

| Rank | Tool | GitHub Stars | Best For | Technical Level |
|------|------|-------------|---------|-----------------|
| 1 | [Mem0](./01-mem0.md) | ~48K | Chatbots, personal assistants | ⭐⭐ Easy |
| 2 | [Zep](./02-zep.md) | ~3.5K | Enterprise, temporal reasoning | ⭐⭐⭐ Medium |
| 3 | [Letta (MemGPT)](./03-letta-memgpt.md) | ~36K | Self-improving agents | ⭐⭐⭐ Medium |
| 4 | [LangMem](./04-langmem.md) | ~2K | LangGraph/LangChain stacks | ⭐⭐⭐ Medium |
| 5 | [OpenAI ChatGPT Memory](./05-openai-chatgpt-memory.md) | N/A (built-in) | Non-technical end users | ⭐ Easiest |
| 6 | [Cognee](./06-cognee.md) | ~4K | Structured knowledge graphs | ⭐⭐⭐ Medium |
| 7 | [Memoripy](./07-memoripy.md) | ~1.5K | Lightweight, human-like decay | ⭐⭐ Easy |
| 8 | [claude-mem / cmem](./08-claude-mem.md) | ~9K | Claude Code agent memory | ⭐⭐ Easy |
| 9 | [Graphiti](./09-graphiti.md) | ~6K | Temporal knowledge graphs | ⭐⭐⭐⭐ Advanced |
| 10 | [Memvid](./10-memvid.md) | ~12K | Video-encoded compressed memory | ⭐⭐ Easy |

---

## 📐 Benchmark Reference

Scores referenced in individual tool documents use these standard benchmarks:

### LoCoMo (Long Conversation Memory)
- Tests multi-session memory over long dialogues
- Measures: accuracy, recall, coherence
- Top score 2026: **Mem0 ~92.5%**

### LongMemEval (ICLR 2025)
- 500 users × ~53 sessions each; tests fact update tracking
- Top score 2026: **Mem0 ~94.4%** (newer algorithm)

### BEAM
- Tests memory under 1M token context windows
- Measures scalability and retrieval at scale

### Key Efficiency Metrics
- **Token cost**: Mem0 ~1,764 tokens/conv vs full-context ~26,031
- **p95 latency**: Mem0 ~0.200s search retrieval

---

## 🧩 Memory Type Coverage

| Tool | Semantic | Episodic | Procedural | Temporal | Graph |
|------|----------|----------|------------|----------|-------|
| Mem0 | ✅ | ✅ | ✅ | ⚠️ | ⚠️ |
| Zep | ✅ | ✅ | ✅ | ✅ | ✅ |
| Letta | ✅ | ✅ | ✅ | ⚠️ | ❌ |
| LangMem | ✅ | ✅ | ✅ | ⚠️ | ❌ |
| OpenAI Memory | ✅ | ✅ | ❌ | ⚠️ | ❌ |
| Cognee | ✅ | ✅ | ✅ | ✅ | ✅ |
| Memoripy | ✅ | ✅ | ❌ | ✅ | ❌ |
| claude-mem | ✅ | ⚠️ | ✅ | ❌ | ⚠️ |
| Graphiti | ✅ | ✅ | ✅ | ✅ | ✅ |
| Memvid | ✅ | ✅ | ❌ | ❌ | ❌ |

*✅ Native support | ⚠️ Partial/manual | ❌ Not supported*

---

## 📁 Document Index

Each tool has a dedicated research file:

- [`01-mem0.md`](./01-mem0.md) — Mem0 deep dive
- [`02-zep.md`](./02-zep.md) — Zep deep dive
- [`03-letta-memgpt.md`](./03-letta-memgpt.md) — Letta / MemGPT deep dive
- [`04-langmem.md`](./04-langmem.md) — LangMem deep dive
- [`05-openai-chatgpt-memory.md`](./05-openai-chatgpt-memory.md) — OpenAI ChatGPT Memory deep dive
- [`06-cognee.md`](./06-cognee.md) — Cognee deep dive
- [`07-memoripy.md`](./07-memoripy.md) — Memoripy deep dive
- [`08-claude-mem.md`](./08-claude-mem.md) — claude-mem / cmem deep dive
- [`09-graphiti.md`](./09-graphiti.md) — Graphiti deep dive
- [`10-memvid.md`](./10-memvid.md) — Memvid deep dive

---

## 🗺️ Decision Guide

```
Are you a non-technical user?
  └─ Yes → OpenAI ChatGPT Memory (#5) or Mem0 Cloud (#1)

Are you building a LangChain/LangGraph app?
  └─ Yes → LangMem (#4)

Do you need enterprise-grade temporal reasoning?
  └─ Yes → Zep (#2) or Graphiti (#9)

Are you working with Claude Code?
  └─ Yes → claude-mem / cmem (#8)

Do you need self-improving, stateful agents?
  └─ Yes → Letta (#3)

Do you need structured knowledge graphs?
  └─ Yes → Cognee (#6) or Graphiti (#9)

Do you need lightweight, easy Python memory with decay?
  └─ Yes → Memoripy (#7)

Do you need compressed memory from large document sets?
  └─ Yes → Memvid (#10)

Default / best overall → Mem0 (#1)
```

---

*Next Phase: Hands-on experiments and integration examples for each tool.*
