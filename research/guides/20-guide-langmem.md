# 🚀 Guia Prático: LangMem — Memória para Agentes LangGraph

**Nível:** Intermediário  
**Tempo de setup:** 15 minutos  
**Pré-requisitos:** Python 3.9+, conhecimento básico de LangChain  
**Docs:** https://langchain-ai.github.io/langmem/  

---

## 🎯 O que você vai conseguir

- ✅ Agentes LangGraph que aprendem e se adaptam ao longo do tempo
- ✅ Memória semântica cross-session com qualquer store (RAM, PostgreSQL, Redis)
- ✅ Consolidação de memórias em background (o agente "reflete" e aprende)
- ✅ Multi-usuário com namespaces isolados por user_id

---

## 📦 Instalação

```bash
pip install -U langmem langgraph langchain-openai
```

---

## 🔑 Variáveis de Ambiente

```bash
export OPENAI_API_KEY="sk-sua-chave"
# OU
export ANTHROPIC_API_KEY="sk-ant-sua-chave"

# Opcional: rastreamento no LangSmith
# export LANGSMITH_API_KEY="ls-sua-chave"
```

---

## 🚀 Exemplo 1: Agente com Memória em 5 Minutos

```python
from langgraph.prebuilt import create_react_agent
from langgraph.store.memory import InMemoryStore
from langmem import create_manage_memory_tool, create_search_memory_tool

# ── STORE: onde as memórias ficam guardadas ──
# InMemoryStore = em RAM (para desenvolvimento)
store = InMemoryStore(
    index={
        "dims": 1536,
        "embed": "openai:text-embedding-3-small",
    }
)

# ── AGENT: com ferramentas de memória ──
agente = create_react_agent(
    model="openai:gpt-4o-mini",
    tools=[
        create_manage_memory_tool(namespace=("memorias",)),
        create_search_memory_tool(namespace=("memorias",)),
    ],
    store=store,
    prompt="Você é um assistente pessoal. Use a memória para personalizar respostas.",
)

# ── USAR O AGENTE ──
def conversar(mensagem: str, thread_id: str = "thread_1") -> str:
    config = {"configurable": {"thread_id": thread_id}}
    resultado = agente.invoke(
        {"messages": [{"role": "user", "content": mensagem}]},
        config=config
    )
    return resultado["messages"][-1].content

# Teste
print(conversar("Meu nome é Carlos e sou designer."))
print(conversar("Qual é a minha profissão?"))  # Vai lembrar!
```

---

## 🚀 Exemplo 2: Memória Persistente com PostgreSQL

Para uso em produção com dados que sobrevivem a restarts:

```bash
# Instalar dependências de store
pip install langchain-postgres psycopg2-binary

# Subir PostgreSQL com Docker
docker run -d \
  -e POSTGRES_DB=langmem \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=senha \
  -p 5432:5432 \
  pgvector/pgvector:pg16
```

```python
from langgraph.prebuilt import create_react_agent
from langchain_postgres import PostgresSaver
from langmem import create_manage_memory_tool, create_search_memory_tool

# Store persistente em PostgreSQL
store = PostgresSaver.from_conn_string(
    "postgresql://user:senha@localhost:5432/langmem"
)
store.setup()  # cria tabelas necessárias

# Agente com memória persistente
agente = create_react_agent(
    model="anthropic:claude-3-5-sonnet-20241022",
    tools=[
        create_manage_memory_tool(namespace=("usuarios", "{user_id}")),
        create_search_memory_tool(namespace=("usuarios", "{user_id}")),
    ],
    store=store,
)

# Cada user_id tem namespace isolado
def conversar(mensagem: str, user_id: str, thread_id: str) -> str:
    config = {
        "configurable": {
            "thread_id": thread_id,
            "user_id": user_id
        }
    }
    resultado = agente.invoke(
        {"messages": [{"role": "user", "content": mensagem}]},
        config=config
    )
    return resultado["messages"][-1].content

# Usuários diferentes, memórias separadas
print(conversar("Prefiro Python.", user_id="alice", thread_id="t1"))
print(conversar("Prefiro TypeScript.", user_id="bob", thread_id="t2"))

# Alice só lembra de Python, Bob só de TypeScript
print(conversar("Qual linguagem prefiro?", user_id="alice", thread_id="t1"))
```

---

## 🚀 Exemplo 3: Consolidação de Memória em Background

O agente "reflete" sobre conversas passadas e cria memórias mais ricas:

```python
import asyncio
from langchain.chat_models import init_chat_model
from langmem import ReflectionExecutor, create_memory_store_manager
from langgraph.store.memory import InMemoryStore

llm = init_chat_model("gpt-4o-mini", model_provider="openai")

store = InMemoryStore(
    index={"dims": 1536, "embed": "openai:text-embedding-3-small"}
)

# Manager extrai memórias estruturadas das conversas
manager = create_memory_store_manager(
    llm=llm,
    namespace=("usuarios", "{user_id}"),
    store=store,
)

# ReflectionExecutor roda consolidação assíncrona em background
executor = ReflectionExecutor(manager)

# Simulação de conversa
historico_conversa = [
    {"role": "user", "content": "Trabalho como desenvolvedor backend."},
    {"role": "assistant", "content": "Entendido!"},
    {"role": "user", "content": "Uso principalmente Go e PostgreSQL."},
    {"role": "assistant", "content": "Ótimas escolhas para backend!"},
    {"role": "user", "content": "Meu maior desafio é performance em queries complexas."},
]

# Agenda consolidação para 5 segundos após a última mensagem
executor.submit(
    {"messages": historico_conversa},
    after_seconds=5,  # delay para não sobrecarregar durante a conversa
    config={"configurable": {"user_id": "desenvolvedor_1"}}
)

print("⏳ Consolidação agendada em background...")
# O agente vai criar memórias como:
# - "Desenvolvedor backend especializado em Go e PostgreSQL"
# - "Principal desafio: otimização de queries complexas"
```

---

## 🔧 Configuração Multi-Agente (Time de Agentes com Memória Compartilhada)

```python
from langgraph.prebuilt import create_react_agent
from langgraph.store.memory import InMemoryStore
from langmem import create_manage_memory_tool, create_search_memory_tool

# Store compartilhado entre agentes
store_compartilhado = InMemoryStore(
    index={"dims": 1536, "embed": "openai:text-embedding-3-small"}
)

# Namespace da empresa — compartilhado
NS_EMPRESA = ("empresa", "conhecimento")

def criar_agente_especialista(papel: str, modelo: str) -> object:
    return create_react_agent(
        model=modelo,
        tools=[
            create_manage_memory_tool(namespace=NS_EMPRESA),
            create_search_memory_tool(namespace=NS_EMPRESA),
        ],
        store=store_compartilhado,
        prompt=f"Você é um especialista em {papel}. Compartilhe e consulte o conhecimento da equipe.",
    )

# Agentes especializados com memória compartilhada
agente_backend = criar_agente_especialista("backend/APIs", "openai:gpt-4o-mini")
agente_frontend = criar_agente_especialista("frontend/UX", "openai:gpt-4o-mini")
agente_devops = criar_agente_especialista("infraestrutura/DevOps", "openai:gpt-4o-mini")

# O que um agente aprende, os outros podem consultar
```

---

## 🔄 Workflow: Assistente que Evolui com o Uso

```python
from langgraph.prebuilt import create_react_agent
from langgraph.store.memory import InMemoryStore
from langmem import create_manage_memory_tool, create_search_memory_tool

store = InMemoryStore(index={"dims": 1536, "embed": "openai:text-embedding-3-small"})

agente = create_react_agent(
    model="openai:gpt-4o-mini",
    tools=[
        create_manage_memory_tool(namespace=("usuario",)),
        create_search_memory_tool(namespace=("usuario",)),
    ],
    store=store,
    prompt="""Você é um assistente pessoal que:
    1. Sempre salva preferências e fatos importantes sobre o usuário
    2. Sempre busca memórias relevantes antes de responder
    3. Personaliza respostas com base no histórico
    4. Sinaliza quando uma nova informação contradiz algo memorizado""",
)

# Sessão 1
agente.invoke({"messages": [
    {"role": "user", "content": "Detesto reuniões de segunda de manhã."}
]}, config={"configurable": {"thread_id": "s1"}})

# Sessão 2 (dias depois, thread diferente)
resultado = agente.invoke({"messages": [
    {"role": "user", "content": "Pode marcar uma reunião de alinhamento?"}
]}, config={"configurable": {"thread_id": "s2"}})

# O agente vai sugerir horários que não sejam segunda de manhã!
print(resultado["messages"][-1].content)
```

---

## ⚡ Otimização de Tokens

```python
# Use namespace hierárquico para busca precisa
# Em vez de buscar em toda a memória, filtre por categoria

# Memória separada por tipo
tools_trabalho = [
    create_manage_memory_tool(namespace=("usuario", "trabalho")),
    create_search_memory_tool(namespace=("usuario", "trabalho")),
]
tools_pessoal = [
    create_manage_memory_tool(namespace=("usuario", "pessoal")),
    create_search_memory_tool(namespace=("usuario", "pessoal")),
]

# Resultado: a IA busca só no namespace relevante para o contexto
# = menos tokens por busca, mais precisão
```

---

## 🔗 Recursos

- [GitHub LangMem](https://github.com/langchain-ai/langmem)
- [Documentação](https://langchain-ai.github.io/langmem/)
- [LangGraph Docs](https://langchain-ai.github.io/langgraph/)
- [← Pesquisa sobre LangMem](../04-langmem.md)
