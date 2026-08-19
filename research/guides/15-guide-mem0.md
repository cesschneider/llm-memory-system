# 🚀 Guia Prático: Mem0 — Memória Inteligente para Agentes de IA

**Nível:** Iniciante → Intermediário  
**Tempo de setup:** 10–30 minutos  
**Pré-requisitos:** Python 3.8+ instalado  
**Documentação oficial:** https://docs.mem0.ai  

---

## 🎯 O que você vai conseguir

- ✅ IA que lembra preferências, projetos e contexto do usuário entre sessões
- ✅ Memória automática sem precisar repetir contexto a cada conversa
- ✅ 91% menos tokens por conversa vs. passar contexto completo manualmente
- ✅ Funciona com OpenAI, Anthropic, Ollama e outros

---

## 📦 OPÇÃO A — Cloud (Mais Fácil, 10 minutos)

### Passo 1: Criar conta no Mem0

1. Acesse **https://app.mem0.ai**
2. Clique em **Sign Up** → crie conta com email ou Google
3. Vá em **API Keys** → copie sua chave (começa com `m0-...`)

---

### Passo 2: Instalar a biblioteca

Abra o terminal e execute:

```bash
pip install mem0ai
```

> 💡 Se não tem Python, instale em https://python.org/downloads

---

### Passo 3: Seu primeiro script

Crie um arquivo `memoria.py` e cole:

```python
from mem0 import MemoryClient

# Conecta ao Mem0 Cloud
client = MemoryClient(api_key="m0-SUA-CHAVE-AQUI")

# === SALVAR uma memória ===
client.add(
    "Me chamo João, trabalho como engenheiro de software e prefiro Python.",
    user_id="joao"
)
print("✅ Memória salva!")

# === BUSCAR memórias relevantes ===
resultados = client.search(
    "Qual é a linguagem de programação preferida do usuário?",
    user_id="joao"
)
for mem in resultados:
    print(f"🧠 {mem['memory']}")

# === VER TODAS as memórias ===
todas = client.get_all(user_id="joao")
print(f"\n📋 Total de memórias: {len(todas)}")
for mem in todas:
    print(f"  - {mem['memory']}")
```

Execute:
```bash
python memoria.py
```

---

### Passo 4: Integrar com ChatGPT ou Claude

```python
from mem0 import MemoryClient
from openai import OpenAI

mem = MemoryClient(api_key="m0-SUA-CHAVE-AQUI")
openai = OpenAI(api_key="SUA-OPENAI-KEY")

USER_ID = "usuario_123"

def chat_com_memoria(pergunta: str) -> str:
    # 1. Busca memórias relevantes
    memorias = mem.search(pergunta, user_id=USER_ID)
    contexto = "\n".join([m["memory"] for m in memorias])

    # 2. Monta prompt com contexto de memória
    mensagens = []
    if contexto:
        mensagens.append({
            "role": "system",
            "content": f"O que você sabe sobre o usuário:\n{contexto}"
        })
    mensagens.append({"role": "user", "content": pergunta})

    # 3. Chama a IA
    resposta = openai.chat.completions.create(
        model="gpt-4o-mini",
        messages=mensagens
    )
    texto_resposta = resposta.choices[0].message.content

    # 4. Salva a interação como memória
    mem.add(
        [
            {"role": "user", "content": pergunta},
            {"role": "assistant", "content": texto_resposta}
        ],
        user_id=USER_ID
    )

    return texto_resposta

# Teste
print(chat_com_memoria("Meu nome é Maria e adoro café!"))
print(chat_com_memoria("Qual é o meu nome?"))  # Vai lembrar!
```

---

## 📦 OPÇÃO B — Self-Hosted (Grátis, Sem Limites, 20 minutos)

### Passo 1: Instalar com todas as dependências

```bash
pip install "mem0ai[qdrant]"
```

### Passo 2: Subir o Qdrant (banco de vetores) com Docker

```bash
# Instale Docker em https://docker.com/get-started se não tiver
docker run -d -p 6333:6333 qdrant/qdrant
```

### Passo 3: Configurar e usar

```python
from mem0 import Memory

config = {
    "vector_store": {
        "provider": "qdrant",
        "config": {
            "collection_name": "minha_memoria",
            "host": "localhost",
            "port": 6333,
        }
    },
    "llm": {
        "provider": "openai",
        "config": {
            "model": "gpt-4o-mini",
            "api_key": "SUA-OPENAI-KEY"
        }
    },
    "embedder": {
        "provider": "openai",
        "config": {"model": "text-embedding-3-small"}
    }
}

m = Memory.from_config(config)

# Adicionar memória
m.add("Prefiro reuniões de manhã e odeio videoconferências longas.", user_id="ana")

# Buscar
resultado = m.search("Quando agendar reuniões com Ana?", user_id="ana")
for r in resultado:
    print(r["memory"])
```

---

## 📦 OPÇÃO C — 100% Local com Ollama (Zero custo, total privacidade)

```bash
# 1. Instalar Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# 2. Baixar modelos
ollama pull llama3.2
ollama pull nomic-embed-text

# 3. Instalar mem0
pip install mem0ai
```

```python
from mem0 import Memory

config = {
    "llm": {
        "provider": "ollama",
        "config": {
            "model": "llama3.2",
            "base_url": "http://localhost:11434"
        }
    },
    "embedder": {
        "provider": "ollama",
        "config": {
            "model": "nomic-embed-text",
            "base_url": "http://localhost:11434"
        }
    },
    "vector_store": {
        "provider": "chroma",
        "config": {"collection_name": "memoria_local"}
    }
}

m = Memory.from_config(config)
m.add("Trabalho remotamente e prefiro comunicação por texto.", user_id="pedro")
resultado = m.search("Como Pedro prefere se comunicar?", user_id="pedro")
print(resultado[0]["memory"])
```

---

## 🔄 Workflows Práticos

### Workflow 1: Assistente Pessoal com Memória Persistente

```python
from mem0 import MemoryClient
from anthropic import Anthropic

mem = MemoryClient(api_key="m0-SUA-CHAVE")
claude = Anthropic(api_key="SUA-ANTHROPIC-KEY")

USER_ID = "meu_usuario"

def assistente(mensagem: str) -> str:
    # Recupera contexto relevante
    contexto = mem.search(mensagem, user_id=USER_ID, limit=5)
    contexto_texto = "\n".join([c["memory"] for c in contexto])

    resposta = claude.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        system=f"Você é um assistente pessoal. O que sabe sobre o usuário:\n{contexto_texto}",
        messages=[{"role": "user", "content": mensagem}]
    )
    texto = resposta.content[0].text

    # Aprende com a conversa
    mem.add([
        {"role": "user", "content": mensagem},
        {"role": "assistant", "content": texto}
    ], user_id=USER_ID)

    return texto

# Chat interativo
print("🤖 Assistente com memória (digite 'sair' para encerrar)")
while True:
    entrada = input("\nVocê: ")
    if entrada.lower() == "sair":
        break
    print(f"IA: {assistente(entrada)}")
```

### Workflow 2: Gerenciar e Inspecionar Memórias

```python
from mem0 import MemoryClient

client = MemoryClient(api_key="m0-SUA-CHAVE")
USER_ID = "meu_usuario"

# Ver todas as memórias
print("=== TODAS AS MEMÓRIAS ===")
for mem in client.get_all(user_id=USER_ID):
    print(f"[{mem['id'][:8]}] {mem['memory']}")

# Atualizar uma memória
client.update(memory_id="ID-DA-MEMORIA", data="Texto atualizado da memória")

# Deletar uma memória específica
client.delete(memory_id="ID-DA-MEMORIA")

# Limpar tudo (cuidado!)
client.delete_all(user_id=USER_ID)
```

---

## 🎯 Casos de Uso Prontos

### Para Suporte ao Cliente
```python
# Cada cliente tem seu user_id, a IA lembra histórico de problemas
mem.add("Cliente relatou problema com pagamento em 2026-07", user_id="cliente_456")
```

### Para Assistente de Estudos
```python
# A IA lembra o que você já aprendeu e adapta as explicações
mem.add("Entendo conceitos básicos de Python mas ainda aprendo orientação a objetos", user_id="estudante")
```

### Para CRM Leve
```python
# Lembra preferências e histórico de cada contato
mem.add("João prefere contato por WhatsApp às quintas-feiras", user_id="contato_joao")
```

---

## ⚡ Dicas de Otimização de Tokens

```python
# Use limit para buscar só o necessário
memorias = client.search(query, user_id=USER_ID, limit=3)  # só 3 mais relevantes

# Use threshold para filtrar memórias pouco relevantes
memorias = client.search(query, user_id=USER_ID, threshold=0.7)

# Namespacing: separe memórias por categoria
client.add("Preferência X", user_id=USER_ID, metadata={"categoria": "trabalho"})
```

---

## 🔗 Recursos

- [Documentação Oficial](https://docs.mem0.ai)
- [GitHub](https://github.com/mem0ai/mem0)
- [Playground Online](https://app.mem0.ai)
- [← Pesquisa sobre Mem0](../01-mem0.md)
