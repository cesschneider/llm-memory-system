# 🚀 Guia Prático: Letta (ex-MemGPT) — Agentes com Memória Ilimitada

**Nível:** Intermediário → Avançado  
**Tempo de setup:** 20–30 minutos  
**Pré-requisitos:** Python 3.10+, Docker (opcional)  
**Site:** https://www.letta.com  
**Docs:** https://docs.letta.com  

---

## 🎯 O que você vai conseguir

- ✅ Agentes com contexto **ilimitado** — sem corte de contexto jamais
- ✅ Agente que aprende e melhora a cada sessão autonomamente
- ✅ Memória hierárquica: working memory + recall memory + archival memory
- ✅ Letta Code: agente de programação com memória de projetos

---

## 📦 Instalação

```bash
pip install letta
```

Para Letta Code (agente de programação):
```bash
pip install "letta[code]"
```

---

## 🚀 OPÇÃO A — Letta Cloud (Mais Fácil, 10 minutos)

### Passo 1: Criar conta

1. Acesse **https://app.letta.com**
2. Crie conta gratuita
3. Obtenha sua API key em **Settings → API Keys**

### Passo 2: Instalar e conectar

```bash
pip install letta
export LETTA_API_KEY="sua-api-key"
```

```python
from letta import create_client

client = create_client()  # usa LETTA_API_KEY do ambiente

# Criar um agente persistente
agente = client.create_agent(
    name="meu-assistente",
    model="openai/gpt-4o-mini",
    embedding="openai/text-embedding-3-small",
    memory_human="Nome: Carlos\nProfissão: engenheiro de dados\nCidade: Belo Horizonte",
    memory_persona="Sou um assistente pessoal organizado e direto ao ponto. Uso as memórias do usuário para personalizar minhas respostas.",
)

print(f"✅ Agente criado: {agente.id}")

# Conversar
resp = client.send_message(
    agent_id=agente.id,
    message="Qual é a minha profissão?",
    role="user"
)
print(resp.messages[-1].text)
# "Você é engenheiro de dados, Carlos!"
```

---

## 🚀 OPÇÃO B — Self-Hosted (Grátis, Controle Total)

### Passo 1: Subir o servidor Letta

```bash
# Com Docker (recomendado)
docker run -d \
  --name letta-server \
  -p 8283:8283 \
  -e OPENAI_API_KEY=sua-key \
  -v letta_data:/root/.letta \
  lettaai/letta:latest

# Verificar que está rodando
curl http://localhost:8283/v1/health
```

### Passo 2: Conectar ao servidor local

```python
from letta import create_client

# Conectar ao servidor local
client = create_client(base_url="http://localhost:8283")

# Criar agente no servidor local
agente = client.create_agent(
    name="assistente-local",
    model="openai/gpt-4o-mini",
    memory_human="Usuário autônomo — sem dados para cloud",
    memory_persona="Assistente local com total privacidade de dados.",
)
```

---

## 🚀 OPÇÃO C — Com Modelos Locais (Ollama)

```bash
# Instalar Ollama
curl -fsSL https://ollama.ai/install.sh | sh
ollama pull llama3.1:8b
ollama pull nomic-embed-text

# Subir Letta apontando para Ollama
docker run -d \
  --name letta-ollama \
  -p 8283:8283 \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  lettaai/letta:latest
```

```python
from letta import create_client, LLMConfig, EmbeddingConfig

client = create_client(base_url="http://localhost:8283")

agente = client.create_agent(
    name="agente-totalmente-local",
    llm_config=LLMConfig(
        model="ollama/llama3.1:8b",
        model_endpoint_type="openai",
        model_endpoint="http://localhost:11434/v1",
    ),
    embedding_config=EmbeddingConfig(
        embedding_model="ollama/nomic-embed-text",
        embedding_endpoint="http://localhost:11434",
        embedding_dim=768,
    ),
)
```

---

## 🧠 Entendendo a Memória do Letta

O Letta implementa um sistema de memória em 3 camadas inspirado em sistemas operacionais:

```
┌─────────────────────────────────────────────────────┐
│  WORKING MEMORY (context window — rápida, limitada) │
│                                                     │
│  📝 memory_human   ← fatos sobre o usuário          │
│  🤖 memory_persona ← personalidade e instruções     │
│  💬 conversation   ← últimas N mensagens             │
└─────────────────────────────────────────────────────┘
          │ "paginação" automática quando cheio
          ▼
┌─────────────────────────────────────────────────────┐
│  RECALL MEMORY (episódica — busca por mensagens)    │
│  Histórico completo de todas as conversas           │
│  Agente busca automaticamente quando relevante      │
└─────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────┐
│  ARCHIVAL MEMORY (semântica — conhecimento geral)   │
│  Fatos e documentos indexados por embedding         │
│  Capacidade ilimitada                               │
└─────────────────────────────────────────────────────┘
```

---

## 🔧 Gerenciar Memória do Agente

```python
from letta import create_client

client = create_client()
AGENT_ID = "agent-id-aqui"

# ── VER memória atual ──
agente = client.get_agent(agent_id=AGENT_ID)
print("=== WORKING MEMORY ===")
print("Sobre o humano:", agente.memory.get_block("human").value)
print("Persona:", agente.memory.get_block("persona").value)

# ── ATUALIZAR working memory ──
client.update_agent_memory_block(
    agent_id=AGENT_ID,
    label="human",
    value="Nome: Carlos\nProfissão: Diretor de IA (promovido em ago/2026)\nCidade: São Paulo"
)

# ── ADICIONAR à archival memory ──
client.insert_archival_memory(
    agent_id=AGENT_ID,
    memory="Carlos completou o curso de MLOps na Coursera em julho/2026"
)

# ── BUSCAR archival memory ──
resultados = client.get_archival_memory(
    agent_id=AGENT_ID,
    query="cursos e certificações de Carlos"
)
for mem in resultados:
    print(f"📚 {mem.text}")

# ── VER recall memory (mensagens passadas) ──
mensagens = client.get_messages(agent_id=AGENT_ID, limit=20)
for msg in mensagens:
    print(f"[{msg.role}] {msg.text[:100]}")
```

---

## 🧑‍💻 Letta Code: Agente de Programação com Memória

```bash
# Instalar Letta Code
pip install "letta[code]"

# Iniciar sessão de programação com memória
letta code

# O agente terá:
# - Acesso ao shell
# - Memória de projetos anteriores
# - Skills de programação pré-configuradas
# - Subagentes para tarefas paralelas
```

---

## 🔄 Workflow: Agente de Pesquisa que Aprende

```python
from letta import create_client

client = create_client()

# Agente especializado em pesquisa com memória acumulativa
agente = client.create_agent(
    name="pesquisador-llm-memory",
    model="openai/gpt-4o",
    memory_human="Projeto de pesquisa: Sistemas de memória para LLMs\nRepositório: github.com/cesschneider/llm-memory-system",
    memory_persona="""Sou um pesquisador especializado em memória para LLMs.
Mantenho um registro detalhado do que já estudei para não repetir pesquisas.
Sempre salvo insights importantes na archival memory.
Conecto novos conhecimentos com o que já aprendi.""",
)

def pesquisar(topico: str) -> str:
    """O agente pesquisa e lembra o que aprendeu."""
    resp = client.send_message(
        agent_id=agente.id,
        message=f"Pesquise e me dê um resumo sobre: {topico}. Salve os insights principais na sua memória.",
        role="user"
    )
    return resp.messages[-1].text

# Sessão 1
print(pesquisar("diferenças entre Mem0 e Zep"))
# Sessão 2 (pode ser dias depois, thread diferente)
print(pesquisar("quando usar Mem0 vs Zep na prática"))
# O agente já sabe o que pesquisou antes e conecta os conhecimentos
```

---

## 🔗 Recursos

- [GitHub](https://github.com/letta-ai/letta)
- [Site Oficial](https://www.letta.com)
- [Documentação](https://docs.letta.com)
- [MemGPT Paper](https://arxiv.org/abs/2310.08560)
- [← Pesquisa sobre Letta](../03-letta-memgpt.md)
