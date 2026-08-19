# ⚡ Guia de Otimização de Tokens — Reduza Custos e Aumente Performance

**Tipo:** Guia Estratégico + Técnicas Práticas  
**Aplica-se a:** ChatGPT, Claude, Gemini e qualquer LLM  
**Última atualização:** 2026-08  

---

## 🎯 Por que Otimização de Tokens Importa

| Situação | Tokens gastos | Custo estimado (GPT-4o) |
|----------|--------------|------------------------|
| Passar histórico completo de 1 mês | ~800.000 | ~$2,40 por conversa |
| Usar Mem0 (busca semântica) | ~1.764 | ~$0,005 por conversa |
| Usar resumo comprimido manual | ~2.000–5.000 | ~$0,006–$0,015 |
| **Economia potencial** | **99%+ de redução** | **de centavos a frações** |

**Além do custo:** menos tokens = respostas mais rápidas e contexto mais preciso (sem ruído).

---

## 🗺️ Mapa de Estratégias

```
ESTRATÉGIAS DE OTIMIZAÇÃO DE TOKENS
│
├── 1. Compressão de Contexto
│   ├── Resumo progressivo
│   ├── Extração de fatos-chave
│   └── Poda de mensagens antigas
│
├── 2. Recuperação Seletiva (RAG)
│   ├── Busca semântica (Mem0, Memvid)
│   ├── Busca por grafo (Graphiti, Zep)
│   └── Retrieval híbrido
│
├── 3. Prompts Eficientes
│   ├── Instruções de sistema compactas
│   ├── Templates reutilizáveis
│   └── Prefill de resposta (Claude)
│
├── 4. Gerenciamento de Sessão
│   ├── Thread splitting
│   ├── Context handoff
│   └── Memory distillation
│
└── 5. Modelo Certo para cada Tarefa
    ├── Routing inteligente
    ├── Cascata de modelos
    └── Cache de respostas
```

---

## 🔧 ESTRATÉGIA 1: Compressão de Contexto

### Técnica 1A: Resumo Progressivo

Em vez de passar toda a conversa, comprima ao final de cada sessão:

```python
from openai import OpenAI

client = OpenAI(api_key="SUA-KEY")

def comprimir_historico(historico: list[dict], max_tokens: int = 500) -> str:
    """Comprime um histórico longo em um resumo conciso."""
    historico_texto = "\n".join([
        f"{m['role'].upper()}: {m['content']}"
        for m in historico
    ])

    resp = client.chat.completions.create(
        model="gpt-4o-mini",  # modelo barato para compressão
        messages=[
            {
                "role": "system",
                "content": f"Resuma esta conversa em no máximo {max_tokens} tokens. "
                           "Mantenha: fatos importantes, decisões, preferências do usuário, "
                           "contexto de projetos. Descarte: repetições, amenidades, exemplos simples."
            },
            {"role": "user", "content": historico_texto}
        ],
        max_tokens=max_tokens
    )
    return resp.choices[0].message.content

# Uso prático
historico_longo = [
    {"role": "user", "content": "Meu nome é Ana, trabalho com marketing digital..."},
    # ... 50 mensagens ...
]

resumo = comprimir_historico(historico_longo)
print(f"Original: ~{len(str(historico_longo).split())} palavras")
print(f"Comprimido: ~{len(resumo.split())} palavras")
print(f"\nResumo:\n{resumo}")
```

### Técnica 1B: Extração de Fatos-Chave

```python
def extrair_fatos(conversa: str) -> list[str]:
    """Extrai apenas os fatos importantes de uma conversa."""
    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": """Extraia fatos importantes desta conversa.
Formato: uma lista de fatos curtos e objetivos.
Inclua: preferências, identidade, projetos, decisões, restrições.
Exclua: perguntas respondidas, exemplos, análises temporárias.
Cada fato em uma linha começando com '-'."""
            },
            {"role": "user", "content": conversa}
        ],
        max_tokens=300
    )
    fatos_texto = resp.choices[0].message.content
    return [f.strip("- ").strip() for f in fatos_texto.split("\n") if f.strip().startswith("-")]

fatos = extrair_fatos("Ana é diretora de marketing. Prefere relatórios visuais...")
for fato in fatos:
    print(f"• {fato}")
```

### Técnica 1C: Poda de Mensagens Antigas (Sliding Window)

```python
def janela_deslizante(
    historico: list[dict],
    max_mensagens: int = 10,
    resumo_antigo: str = ""
) -> tuple[str, list[dict]]:
    """Mantém apenas as N mensagens mais recentes + resumo do resto."""

    if len(historico) <= max_mensagens:
        return resumo_antigo, historico

    # Separa mensagens antigas das recentes
    antigas = historico[:-max_mensagens]
    recentes = historico[-max_mensagens:]

    # Comprime as antigas
    novo_resumo = comprimir_historico(antigas)
    if resumo_antigo:
        # Mescla com resumo anterior
        novo_resumo = comprimir_historico([
            {"role": "user", "content": f"Resumo anterior: {resumo_antigo}\nNovo contexto: {novo_resumo}"}
        ])

    return novo_resumo, recentes

# Uso
resumo, recentes = janela_deslizante(historico_completo, max_mensagens=8)

# Montar mensagens para enviar à IA
mensagens_para_enviar = []
if resumo:
    mensagens_para_enviar.append({
        "role": "system",
        "content": f"Contexto anterior comprimido:\n{resumo}"
    })
mensagens_para_enviar.extend(recentes)
```

---

## 🔧 ESTRATÉGIA 2: Recuperação Seletiva (RAG)

### Técnica 2A: Busca Semântica com Mem0

```python
from mem0 import MemoryClient

mem = MemoryClient(api_key="m0-SUA-CHAVE")
USER_ID = "usuario"

def contexto_relevante(pergunta: str, max_memorias: int = 3) -> str:
    """Recupera apenas memórias relevantes para a pergunta atual."""
    memorias = mem.search(
        pergunta,
        user_id=USER_ID,
        limit=max_memorias,
        threshold=0.7  # só memórias com 70%+ de relevância semântica
    )

    if not memorias:
        return ""

    return "Contexto relevante:\n" + "\n".join([
        f"• {m['memory']}" for m in memorias
    ])

# Resultado: apenas 2-3 fatos relevantes em vez do histórico completo
contexto = contexto_relevante("Qual editor de código você prefere?")
# "• Prefere VS Code com Vim keybindings"  ← só isso, 7 palavras!
```

### Técnica 2B: Cache de Contexto (Claude)

Claude suporta cache nativo de prompt — reduz custo em 90% para contextos repetidos:

```python
from anthropic import Anthropic

client = Anthropic(api_key="SUA-KEY")

# Contexto estático que não muda (documentação, instruções, base de conhecimento)
CONTEXTO_ESTATICO = """
[Sua documentação ou base de conhecimento longa aqui]
""" * 50  # simula contexto grande

def perguntar_com_cache(pergunta: str) -> str:
    resp = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        system=[
            {
                "type": "text",
                "text": CONTEXTO_ESTATICO,
                "cache_control": {"type": "ephemeral"}  # ← cacheia este bloco
            },
            {
                "type": "text",
                "text": "Responda com base no contexto acima."
            }
        ],
        messages=[{"role": "user", "content": pergunta}]
    )
    return resp.content[0].text

# Primeira chamada: processa o contexto completo
# Chamadas seguintes: usa o cache — 90% mais barato e 2x mais rápido!
print(perguntar_com_cache("Como funciona o módulo X?"))
print(perguntar_com_cache("Quais são os endpoints disponíveis?"))
```

### Técnica 2C: Prefill de Resposta (Claude)

Force o início da resposta para eliminar tokens de preâmbulo:

```python
resp = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=500,
    messages=[
        {"role": "user", "content": "Liste as 3 principais vantagens do Mem0."},
        # Prefill: força o Claude a começar diretamente na lista
        {"role": "assistant", "content": "1."}
    ]
)
# Resposta começa direto no conteúdo, sem "Claro! As principais vantagens são..."
# Economiza 15-30 tokens de preâmbulo por resposta
```

---

## 🔧 ESTRATÉGIA 3: Prompts Eficientes

### Técnica 3A: Sistema de Prompt Compacto

```python
# ❌ INEFICIENTE: 847 tokens de sistema
sistema_verboso = """
Você é um assistente de IA extremamente útil, prestativo e amigável.
Você sempre deve ser respeitoso, educado e profissional em suas respostas.
Você deve responder em português do Brasil, usando linguagem clara e acessível.
Você não deve inventar informações que não sabe. [... continua por mais 800 tokens ...]
"""

# ✅ EFICIENTE: 52 tokens, mesmo resultado
sistema_compacto = """
Assistente em pt-BR. Direto, sem preâmbulo. Não invente fatos.
Contexto do usuário: dev backend, prefere tópicos, sem formalidade.
"""

# Redução: 94% menos tokens de sistema
```

### Técnica 3B: Templates Reutilizáveis

```python
# Defina uma vez, reutilize sempre
TEMPLATES = {
    "resumo": "Resuma em {n} bullets:\n{conteudo}",
    "codigo": "Código {linguagem} para: {tarefa}. Sem explicação, só código.",
    "decisao": "Decida entre {opcao_a} e {opcao_b} para {contexto}. Resposta: uma linha.",
    "debug": "Erro: {erro}\nCódigo: {codigo}\nSolução direta:",
}

def usar_template(nome: str, **kwargs) -> str:
    return TEMPLATES[nome].format(**kwargs)

# Uso — tokens mínimos, resultado máximo
prompt = usar_template("resumo", n=3, conteudo="[texto longo]")
prompt = usar_template("codigo", linguagem="Python", tarefa="ordenar lista de dicts por chave 'score'")
prompt = usar_template("decisao", opcao_a="PostgreSQL", opcao_b="MongoDB", contexto="app de e-commerce")
```

### Técnica 3C: Compressão de Contexto com XML (Claude)

```python
# Claude processa XML eficientemente — use para estruturar contexto
contexto_xml = """<memoria>
  <perfil>dev backend, Go/Python, 8 anos exp</perfil>
  <projeto>API de pagamentos, FastAPI, PostgreSQL</projeto>
  <prefs>commits semânticos, testes antes de código</prefs>
  <pendente>PR #45 revisão, deploy sexta</pendente>
</memoria>"""

# XML é mais denso que texto natural — menos tokens, mesma informação
```

---

## 🔧 ESTRATÉGIA 4: Gerenciamento de Sessão

### Técnica 4A: Handoff entre Sessões

```python
def gerar_handoff(conversa: list[dict]) -> str:
    """Gera um documento de handoff para a próxima sessão."""
    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {
                "role": "system",
                "content": """Gere um documento de handoff conciso para continuar esta conversa.
Formato:
STATUS: [o que foi feito]
PENDENTE: [o que falta]
CONTEXTO: [informações críticas para continuar]
PRÓXIMO PASSO: [primeira ação da próxima sessão]

Máximo: 200 palavras."""
            },
            *conversa
        ]
    )
    return resp.choices[0].message.content

# No início da próxima sessão:
handoff = gerar_handoff(sessao_anterior)
nova_sessao = [
    {"role": "system", "content": f"Continuando da sessão anterior:\n{handoff}"},
    {"role": "user", "content": "Vamos continuar de onde paramos."}
]
```

### Técnica 4B: Memory Distillation com Mem0

```python
from mem0 import MemoryClient

mem = MemoryClient(api_key="m0-SUA-CHAVE")

def finalizar_sessao(conversa: list[dict], user_id: str):
    """Extrai e salva memórias importantes ao final de cada sessão."""
    # Mem0 extrai automaticamente os fatos relevantes da conversa
    mem.add(conversa, user_id=user_id)

    # Verificar o que foi aprendido
    memorias = mem.get_all(user_id=user_id)
    print(f"✅ {len(memorias)} memórias totais para este usuário")
    print("Última adição:")
    for m in memorias[-3:]:
        print(f"  • {m['memory']}")

# Uso ao encerrar uma conversa
finalizar_sessao(historico_da_sessao, user_id="usuario_123")
```

---

## 🔧 ESTRATÉGIA 5: Modelo Certo para Cada Tarefa

### Técnica 5A: Cascata de Modelos (Model Routing)

```python
def escolher_modelo(pergunta: str) -> str:
    """Escolhe o modelo mais barato que consegue resolver a tarefa."""
    pergunta_lower = pergunta.lower()

    # Tarefas simples → modelo barato
    if any(kw in pergunta_lower for kw in ["resumir", "traduzir", "listar", "formatar"]):
        return "gpt-4o-mini"          # ~20x mais barato que GPT-4o

    # Tarefas de código → modelo especializado
    if any(kw in pergunta_lower for kw in ["código", "bug", "função", "script"]):
        return "gpt-4o"               # melhor em código

    # Raciocínio complexo → modelo mais capaz
    if any(kw in pergunta_lower for kw in ["arquitetura", "estratégia", "analisar", "planejar"]):
        return "claude-3-5-sonnet-20241022"  # melhor em raciocínio profundo

    # Default: modelo balanceado
    return "gpt-4o-mini"

# Uso
modelo = escolher_modelo("Resuma este artigo em 3 pontos")
resp = client.chat.completions.create(model=modelo, messages=[...])
```

### Técnica 5B: Cache de Respostas para Perguntas Repetidas

```python
import hashlib
import json
from pathlib import Path

CACHE_DIR = Path(".cache/llm_responses")
CACHE_DIR.mkdir(parents=True, exist_ok=True)

def cache_key(modelo: str, mensagens: list) -> str:
    conteudo = json.dumps({"model": modelo, "messages": mensagens}, sort_keys=True)
    return hashlib.md5(conteudo.encode()).hexdigest()

def chamar_com_cache(modelo: str, mensagens: list, ttl_horas: int = 24) -> str:
    """Cacheia respostas para evitar chamar a API desnecessariamente."""
    import time

    chave = cache_key(modelo, mensagens)
    arquivo_cache = CACHE_DIR / f"{chave}.json"

    # Verificar cache
    if arquivo_cache.exists():
        dados = json.loads(arquivo_cache.read_text())
        idade_horas = (time.time() - dados["timestamp"]) / 3600
        if idade_horas < ttl_horas:
            print(f"💾 Cache hit! Economizou tokens.")
            return dados["resposta"]

    # Chamar API
    resp = client.chat.completions.create(model=modelo, messages=mensagens)
    resposta = resp.choices[0].message.content

    # Salvar no cache
    arquivo_cache.write_text(json.dumps({
        "resposta": resposta,
        "timestamp": time.time()
    }))

    return resposta
```

---

## 📊 Comparativo de Estratégias

| Estratégia | Redução de Tokens | Complexidade | Melhor Para |
|------------|------------------|--------------|-------------|
| Resumo progressivo | 60–80% | Baixa | Conversas longas |
| Busca semântica (Mem0) | 85–95% | Média | Multi-sessão |
| Prefill (Claude) | 5–15% | Baixa | Todas as chamadas |
| Cache de prompt (Claude) | 90%* | Baixa | Contexto repetido |
| Sliding window | 50–70% | Baixa | Sessões únicas longas |
| Model routing | 70–80%** | Média | Apps com vários tipos de tarefas |
| Cache de respostas | 100%*** | Média | Perguntas repetidas |

*custo, não tokens  
**custo  
***para perguntas já respondidas

---

## ⚡ Checklist de Otimização

```
Para CADA projeto/app com LLMs, verifique:

Sistema de Prompt:
□ Tem menos de 200 tokens?
□ Remove preâmbulos e educação excessiva?
□ Usa formato estruturado (XML, JSON) para dados?

Contexto:
□ Usa RAG em vez de full context?
□ Tem sliding window ou compressão?
□ Usa cache de prompt para contexto estático?

Modelos:
□ Usa o modelo mais barato que resolve a tarefa?
□ Tem model routing por tipo de pergunta?
□ Cacheia respostas para perguntas repetidas?

Memória:
□ Usa Mem0/Zep em vez de histórico completo?
□ Extrai fatos ao fim de cada sessão?
□ Tem handoff document entre sessões?
```

---

## 🔗 Recursos

- [Anthropic Prompt Caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- [OpenAI Token Counter](https://platform.openai.com/tokenizer)
- [Mem0 Token Efficiency](https://mem0.ai/blog/state-of-ai-agent-memory-2026)
- [LangChain Token Tracking](https://python.langchain.com/docs/modules/callbacks/token_counting)
- [← Guia do Obsidian](../14-step-by-step-guide.md)
- [← Visão Geral](../00-overview.md)
