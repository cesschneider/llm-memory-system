# 🧠 LLM Memory System

A comprehensive research repository exploring memory management strategies for LLM-based systems — covering architectures, tools, benchmarks, and real-world implementation patterns.

---

## 📌 Overview

Large Language Models are stateless by nature: each inference call starts from scratch. Effective memory management is what bridges this gap — enabling LLMs to reason across sessions, retain context, personalize responses, and act as true long-term agents.

This repo documents an ongoing research effort to evaluate, compare, and implement different memory paradigms for LLM-powered applications. It includes:

- 📚 Research summaries and literature reviews
- 🧪 Experiments and benchmark results
- 💡 Implementation examples and code snippets
- 🛠️ Setup guides and how-to instructions
- 📊 Comparisons across memory systems and frameworks

---

## 🗂️ Repository Structure

```
llm-memory-system/
├── research/           # Literature reviews, papers, and findings
├── systems/            # Per-system deep dives (MemGPT, Zep, Mem0, etc.)
├── experiments/        # Test setups, prompts, benchmarks, results
├── examples/           # Working code examples and integrations
├── docs/               # How-to guides and architecture notes
└── README.md
```

---

## 🧩 Memory Taxonomy

LLM memory can be categorized into four types, inspired by cognitive science:

| Type | Description | Examples |
|------|-------------|---------|
| **Sensory** | Immediate context window | System prompt, current conversation |
| **Short-term** | Session-scoped working memory | In-context summarization, sliding window |
| **Long-term** | Persistent cross-session memory | Vector DBs, knowledge graphs, key-value stores |
| **Episodic** | Event-based autobiographical memory | Timestamped logs, event streams |

---

## 🔬 Systems Under Research

### Frameworks & Libraries
- **[MemGPT](https://memgpt.ai/)** — OS-inspired virtual context management
- **[Mem0](https://mem0.ai/)** — Intelligent memory layer for AI assistants
- **[Zep](https://getzep.com/)** — Long-term memory for LLM apps
- **[LangChain Memory](https://python.langchain.com/docs/how_to/#memory)** — Various memory abstractions
- **[LlamaIndex](https://docs.llamaindex.ai/)** — Memory-augmented retrieval
- **[claude-mem](https://github.com/thedotmack/claude-mem)** — Hook-based memory for Claude Code

### Storage Backends
- **Vector Stores**: Pinecone, Weaviate, Chroma, Qdrant, pgvector
- **Graph DBs**: Neo4j, Memgraph
- **Key-Value**: Redis, Upstash
- **Relational**: PostgreSQL, SQLite

---

## 📋 Research Areas

- [ ] Context window optimization strategies
- [ ] Retrieval-Augmented Generation (RAG) for memory
- [ ] Hierarchical memory compression and summarization
- [ ] Memory consolidation and forgetting mechanisms
- [ ] Personalization and user modeling
- [ ] Multi-agent shared memory architectures
- [ ] Evaluation metrics for memory quality
- [ ] Latency vs. recall trade-offs

---

## 🧪 Experiments

Each experiment lives in `experiments/` with the following structure:

```
experiments/
└── <experiment-name>/
    ├── README.md        # Goal, setup, hypothesis
    ├── setup.py         # Environment and dependencies
    ├── run.py           # Experiment runner
    └── results/         # Outputs, charts, observations
```

---

## 🚀 Getting Started

```bash
# Clone the repo
git clone https://github.com/cesschneider/llm-memory-system.git
cd llm-memory-system

# (Optional) Create a virtual environment
python -m venv .venv && source .venv/bin/activate

# Install base dependencies (updated per experiment)
pip install -r requirements.txt
```

---

## 📖 How to Use This Repo

1. **Browse `research/`** for summaries of papers and tools
2. **Explore `systems/`** for deep dives into specific memory frameworks
3. **Run experiments** from `experiments/` to reproduce benchmarks
4. **Reference `examples/`** for copy-paste integration patterns
5. **Read `docs/`** for architecture guidance and decision guides

---

## 🤝 Contributing

This is a personal research repository. Notes, corrections, and experiment ideas are welcome via issues or pull requests.

---

## 📄 License

MIT — see [LICENSE](LICENSE) for details.

---

*Research in progress. Results and recommendations will be updated as experiments complete.*
