# 🧠 claude-mem / cmem — Deep Research

**Category:** Hook-Based Memory for Claude Code & AI Coding Agents  
**GitHub:** [thedotmack/claude-mem](https://github.com/thedotmack/claude-mem) (original)  
**Stars:** ~9,000 ⭐ (claude-mem, as of mid-2026)  
**Managed Platform:** https://cmem.ai  
**License:** MIT (claude-mem) / Commercial (cmem cloud)  
**Install:** `npx claude-mem install`  

---

## 📋 Summary

claude-mem is a memory layer specifically designed for **Claude Code** and AI coding agents. It uses Claude Code's **hooks system** (SessionStart, PreTool, PostTool) to automatically inject relevant memories into context and capture new ones — without requiring manual API calls. The managed cloud version, **cmem**, extends this with a full database, vector index, sync service, and MCP server in one package.

---

## 🌟 Why It Stands Out

- **~9,000 GitHub stars** — most popular Claude Code memory extension
- **Zero config** — `npx claude-mem install` wires into your agent automatically
- **Hook-based** — triggered automatically at session start and around tool calls
- **Progressive disclosure** — only surfaces memories relevant to the current task
- Supports both **local** (file-based) and **cloud** (cmem) backends
- First memory solution purpose-built for coding agents (not just chatbots)

---

## 🏗️ Architecture

claude-mem uses Claude Code's hook system to intercept and augment the agent lifecycle:

```
Claude Code Session Lifecycle
              │
     ┌────────┴────────┐
     │                 │
SessionStart Hook    PreTool Hook
"What do I know      "What's relevant
 about this           before this
 project/user?"       tool call?"
     │                 │
     ▼                 ▼
  Memory Retrieval (vector search)
     │
     ▼
  Injected into System Prompt / Context
     │
  Claude Code Runs
     │
  PostTool Hook
  "What's worth remembering from this?"
     │
     ▼
  Memory Extraction + Storage
```

**Storage options:**
- `~/.claude-mem/` — local JSON/SQLite (default, offline)
- cmem cloud — fully managed vector DB + sync + MCP server

---

## 📊 Performance

| Metric | claude-mem | Notes |
|--------|-----------|-------|
| Token savings | Up to 71.5x* | Per session vs full context |
| Setup time | < 2 min | Single npx command |
| Hook overhead | Minimal | Lightweight retrieval at hook points |
| Multi-project | ✅ | Namespaced per project |

*\*71.5x figure from lucasrosati/claude-code-memory-setup benchmark using Obsidian + Graphify pipeline*

---

## 🚀 Installation & Setup

### For Claude Code Users (Technical)

```bash
# One-command install — wires hooks into Claude Code automatically
npx claude-mem install
```

That's it. On next `claude` session start, memory hooks are active.

### Manual Hook Configuration

If you prefer to configure manually, add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "hooks": {
    "SessionStart": [
      {
        "command": "claude-mem recall --session-start",
        "timeout": 5000
      }
    ],
    "PreTool": [
      {
        "command": "claude-mem recall --pre-tool",
        "timeout": 2000
      }
    ],
    "PostTool": [
      {
        "command": "claude-mem store --post-tool",
        "timeout": 3000
      }
    ]
  }
}
```

### Using cmem (Cloud, Non-Technical Friendly)

```bash
# Install the engine
npx claude-mem install

# Link to cmem cloud for managed storage
npx claude-mem login

# Done — memory is now synced across devices via cmem cloud
```

### Programmatic API

```bash
# Save a memory manually
claude-mem store "This project uses TypeScript with strict mode."

# Retrieve relevant memories
claude-mem recall "TypeScript configuration"

# List all memories
claude-mem list

# Delete a memory
claude-mem delete <memory-id>
```

---

## 🔌 Integrations

| Platform | Support |
|----------|---------|
| Claude Code | ✅ Primary target — hook-native |
| OpenClaw | ✅ Supported |
| Any CLI agent | ✅ Hooks are shell commands |
| MCP clients | ✅ cmem provides MCP server |
| VSCode (via Claude) | ✅ Indirect |
| Cursor | ⚠️ Partial (no native hooks) |

---

## 👤 Ease of Use

| Audience | Rating | Notes |
|----------|--------|-------|
| Non-technical | ⭐⭐⭐ / 5 | npx install is easy; requires Claude Code |
| Claude Code users | ⭐⭐⭐⭐⭐ / 5 | Zero friction, automatic |
| Developers (other tools) | ⭐⭐ / 5 | Hook system is Claude Code-specific |

**Quickest path to working memory:** ~2 minutes (npx install)

---

## ✅ Pros

- Easiest memory for Claude Code specifically
- Automatic — no manual memory.add() calls required
- Progressive disclosure avoids context pollution
- Local option is fully private and offline
- Active community (~9K stars)
- cmem cloud adds sync across machines
- Pair well with graph memory tools (Graphiti, knowledge-graph MCP)

## ❌ Cons

- Claude Code-specific — not useful for other LLM frameworks
- Requires Node.js / npx for installation
- Cloud (cmem) is a managed service with its own pricing
- Hook-based architecture may conflict with other Claude Code customizations
- No built-in knowledge graph (pairs well with Graphiti for that)
- Memory quality for non-code context is basic vs specialized tools

---

## 💰 Pricing

| Option | Price | Notes |
|--------|-------|-------|
| claude-mem (local) | Free | MIT license, fully local |
| cmem Free | Free | Limited cloud storage |
| cmem Pro | Paid | Full sync, unlimited storage |

---

## 🎯 Best Use Cases

1. **Claude Code users** wanting persistent memory across sessions
2. **Coding agents** that learn from previous projects and decisions
3. **Team codebases** where context needs to persist across sessions
4. **Cross-device Claude Code** usage (via cmem cloud sync)
5. **Reducing token costs** in long coding sessions

---

## 🔗 Resources

- [claude-mem GitHub](https://github.com/thedotmack/claude-mem)
- [cmem.ai](https://cmem.ai)
- [Claude Code Memory Setup (Obsidian + Graphiti)](https://github.com/lucasrosati/claude-code-memory-setup)
- [Graph Memory for Claude Code](https://github.com/amarodeabreu/claude-graph-memory)
