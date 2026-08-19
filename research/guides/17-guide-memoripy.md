# 🚀 Guia Prático: Memoripy — Memória com Decaimento Humano

**Nível:** Iniciante Python  
**Tempo de setup:** 5 minutos  
**Pré-requisitos:** Python 3.8+  
**GitHub:** https://github.com/caspianmoon/memoripy  

---

## 🎯 O que você vai conseguir

- ✅ Memória que funciona como a memória humana (lembra mais o recente e relevante)
- ✅ Decaimento natural: memórias antigas e pouco usadas enfraquecem com o tempo
- ✅ 100% local com Ollama — zero custo, zero privacidade comprometida
- ✅ Setup em 5 minutos com `pip install memoripy`

---

## 📦 Instalação

```bash
pip install memoripy
```

Para uso com Ollama (recomendado para privacidade):
```bash
# Instalar Ollama
curl -fsSL https://ollama.ai/install.sh | sh  # macOS/Linux
# Windows: baixar em https://ollama.ai/download

# Baixar modelos necessários
ollama pull llama3.2
ollama pull mxbai-embed-large
```

---

## 🚀 OPÇÃO A — Com OpenAI (Mais Fácil)

```python
from memoripy import MemoryManager, JSONStorage

# Armazenamento em arquivo JSON local
storage = JSONStorage("minhas_memorias.json")

# Inicializar gerenciador
memoria = MemoryManager(
    api_key="SUA-OPENAI-KEY",
    chat_model="gpt-4o-mini",
    chat_model_provider="openai",
    embedding_model="text-embedding-3-small",
    embedding_model_provider="openai",
    storage=storage
)

# Adicionar uma interação (pergunta + resposta)
memoria.add_interaction(
    prompt="Qual linguagem de programação você prefere?",
    output="Prefiro Python para scripts e TypeScript para frontend."
)

memoria.add_interaction(
    prompt="Onde você trabalha?",
    output="Trabalho numa startup de fintech em São Paulo."
)

# Buscar memórias relevantes
short_term, long_term = memoria.load_memory_variables(
    query="Qual é o perfil profissional do usuário?",
    recent_interactions_limit=5,
    similarity_threshold=0.6
)

print("📋 Memória de curto prazo (recente):")
for m in short_term:
    print(f"  P: {m['prompt']}")
    print(f"  R: {m['output']}\n")

print("🧠 Memória de longo prazo (relevante):")
for m in long_term:
    print(f"  [{m.get('relevance_score', 0):.2f}] {m['prompt']}")
```

---

## 🚀 OPÇÃO B — Com Ollama (100% Local, Zero Custo)

```python
from memoripy import MemoryManager, JSONStorage

storage = JSONStorage("memorias_locais.json")

# Configuração com Ollama - nenhum dado sai do computador
memoria = MemoryManager(
    chat_model="llama3.2",
    chat_model_provider="ollama",
    embedding_model="mxbai-embed-large",
    embedding_model_provider="ollama",
    storage=storage
    # sem api_key: é gratuito e local
)

# Uso idêntico ao exemplo com OpenAI
memoria.add_interaction(
    prompt="Meu nome é Carla e sou médica.",
    output="Entendido! Vou lembrar que você é a Dra. Carla."
)

resultado_st, resultado_lt = memoria.load_memory_variables(
    query="Qual é a profissão do usuário?",
    recent_interactions_limit=3,
    similarity_threshold=0.5
)
```

---

## 🤖 Assistente Completo com Memoripy

```python
from memoripy import MemoryManager, JSONStorage
from openai import OpenAI

# Setup
client = OpenAI(api_key="SUA-KEY")
storage = JSONStorage("assistente_memoria.json")
memoria = MemoryManager(
    api_key="SUA-KEY",
    chat_model="gpt-4o-mini",
    chat_model_provider="openai",
    embedding_model="text-embedding-3-small",
    embedding_model_provider="openai",
    storage=storage
)

def conversar(pergunta: str) -> str:
    # Recuperar memórias relevantes
    st, lt = memoria.load_memory_variables(
        query=pergunta,
        recent_interactions_limit=4,
        similarity_threshold=0.6
    )

    # Montar contexto para a IA
    contexto_partes = []
    if lt:
        contexto_partes.append("O que sei sobre você (memória longa):")
        for m in lt[:3]:
            contexto_partes.append(f"- {m['output']}")
    if st:
        contexto_partes.append("\nConversa recente:")
        for m in st[-2:]:
            contexto_partes.append(f"  Você: {m['prompt']}")
            contexto_partes.append(f"  Eu: {m['output']}")

    sistema = "\n".join(contexto_partes) if contexto_partes else "Primeira conversa com este usuário."

    # Gerar resposta
    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": f"Você é um assistente pessoal.\n{sistema}"},
            {"role": "user", "content": pergunta}
        ]
    )
    resposta = resp.choices[0].message.content

    # Salvar interação na memória
    memoria.add_interaction(prompt=pergunta, output=resposta)

    return resposta

# Loop de conversa
print("🤖 Assistente com memória humana (digite 'sair' para parar)\n")
while True:
    entrada = input("Você: ").strip()
    if entrada.lower() in ["sair", "exit", "quit"]:
        break
    if not entrada:
        continue
    print(f"IA: {conversar(entrada)}\n")
```

---

## 🧠 Entendendo o Decaimento de Memória

O Memoripy implementa algo parecido com a **curva de esquecimento de Ebbinghaus**:

```
Relevância da memória ao longo do tempo:

100% ──┐
       │\
       │  \
 60%   │    \___
       │         ──────
 20%   │               ──────────────────
       └──────────────────────────────────▶ Tempo
       Hoje  1 sem  1 mês  3 meses  1 ano

Mas: cada vez que a memória é ACESSADA, ela volta ao topo:
100% ──┐         ┐
       │  \      │  \
       │    \    │    \___
       └─────────┴──────────▶ Acesso = reforço
```

**Na prática:**
- Memórias frequentemente relevantes ficam fortes
- Informações desatualizadas naturalmente enfraquecem
- Sem necessidade de limpeza manual

---

## 🔧 Configurações Avançadas

```python
# Controle fino do decaimento
memoria = MemoryManager(
    api_key="SUA-KEY",
    chat_model="gpt-4o-mini",
    chat_model_provider="openai",
    embedding_model="text-embedding-3-small",
    embedding_model_provider="openai",
    storage=storage,
    # Parâmetros de memória (valores padrão):
    # short_term_limit=5,      # quantas interações recentes manter
    # decay_rate=0.01,         # velocidade do esquecimento (menor = lembra mais)
    # similarity_threshold=0.5 # threshold padrão de relevância
)

# Busca mais restrita (só memórias muito relevantes)
st, lt = memoria.load_memory_variables(
    query="...",
    similarity_threshold=0.80  # só 80%+ de relevância semântica
)

# Busca mais ampla (captura mais contexto)
st, lt = memoria.load_memory_variables(
    query="...",
    similarity_threshold=0.40,
    recent_interactions_limit=10
)
```

---

## 💡 Casos de Uso Ideais

### Tutor Personalizado
```python
# Lembra o nível do aluno e o que já foi ensinado
memoria.add_interaction(
    "O que é uma lista em Python?",
    "Expliquei listas básicas com exemplos simples"
)
# Na próxima sessão, não repete o básico
```

### Assistente de Escrita
```python
# Lembra estilo e projetos em andamento
memoria.add_interaction(
    "Qual é o tom do meu livro?",
    "O usuário escreve ficção científica com tom melancólico e foco em personagens"
)
```

### Diário com IA
```python
# Reflexões diárias que acumulam contexto de vida
memoria.add_interaction(
    "Como foi meu dia hoje?",
    "Usuário teve reunião difícil com o chefe sobre prazo do projeto X"
)
```

---

## 🔗 Recursos

- [GitHub](https://github.com/caspianmoon/memoripy)
- [PyPI](https://pypi.org/project/memoripy/)
- [Ollama Models](https://ollama.ai/library)
- [← Pesquisa sobre Memoripy](../07-memoripy.md)
