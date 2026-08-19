# 🚀 Guia Prático: Cognee — Memória em Grafo de Conhecimento

**Nível:** Intermediário  
**Tempo de setup:** 15–20 minutos  
**Pré-requisitos:** Python 3.10+, uv ou pip  
**GitHub:** https://github.com/topoteretes/cognee  

---

## 🎯 O que você vai conseguir

- ✅ IA que entende **relacionamentos** entre conceitos, não apenas fatos isolados
- ✅ Ingerir PDFs, sites, código e textos em um grafo de conhecimento unificado
- ✅ Integração com Claude e Cursor via MCP em 5 minutos
- ✅ 100% self-hosted — dados nunca saem do seu computador

---

## 📦 Instalação

```bash
# Recomendado: usar uv (mais rápido)
pip install uv
uv pip install cognee

# Ou pip tradicional
pip install cognee
```

---

## 🔑 Configuração de Variáveis de Ambiente

Crie um arquivo `.env` na pasta do seu projeto:

```bash
# Provedor de LLM (escolha um)
OPENAI_API_KEY=sk-sua-chave-openai
# OU
ANTHROPIC_API_KEY=sk-ant-sua-chave-claude

# Banco de dados (SQLite por padrão — sem configuração extra)
# DB_PROVIDER=sqlite   ← padrão, não precisa configurar

# Para Neo4j (produção):
# GRAPH_DATABASE_PROVIDER=neo4j
# GRAPH_DATABASE_URL=bolt://localhost:7687
# GRAPH_DATABASE_USERNAME=neo4j
# GRAPH_DATABASE_PASSWORD=sua-senha
```

---

## 🚀 Exemplo Mínimo: 5 Minutos para Funcionar

```python
import cognee
import asyncio

async def main():
    # Resetar estado (primeira execução)
    await cognee.prune.prune_data()
    await cognee.prune.prune_system(metadata=True)

    # ── ADICIONAR CONHECIMENTO ──
    # Textos simples
    await cognee.add("A empresa foi fundada em 2010 por Maria Silva.")
    await cognee.add("Maria Silva é CEO e formada em Engenharia pela USP.")
    await cognee.add("A empresa tem 150 funcionários e sede em São Paulo.")

    # ── PROCESSAR (constrói o grafo) ──
    await cognee.cognify()
    print("✅ Grafo de conhecimento construído!")

    # ── BUSCAR ──
    resultados = await cognee.search(
        query_text="Quem fundou a empresa e qual é sua formação?",
        query_type="INSIGHTS"
    )
    for r in resultados:
        print(f"🧠 {r}")

asyncio.run(main())
```

---

## 📄 Ingerir Documentos (PDFs, Sites, Código)

```python
import cognee
import asyncio

async def ingerir_documentos():
    await cognee.prune.prune_data()
    await cognee.prune.prune_system(metadata=True)

    # Arquivo de texto
    await cognee.add("path/to/documento.txt")

    # PDF (requer: pip install pypdf)
    await cognee.add("relatorio_anual.pdf")

    # Diretório inteiro
    await cognee.add("./docs/")

    # URL
    await cognee.add("https://exemplo.com/artigo")

    # Texto com metadados
    await cognee.add(
        "FastAPI é um framework Python moderno para APIs REST.",
        dataset_name="tech_stack"  # organiza por dataset
    )

    # Processar tudo
    await cognee.cognify()
    print("✅ Todos os documentos processados!")

asyncio.run(ingerir_documentos())
```

---

## 🔍 Tipos de Busca

```python
import cognee
import asyncio

async def exemplos_busca():
    # INSIGHTS: resposta sintetizada da IA
    r1 = await cognee.search("Qual é a arquitetura do sistema?", query_type="INSIGHTS")

    # CHUNKS: trechos brutos dos documentos
    r2 = await cognee.search("autenticação JWT", query_type="CHUNKS")

    # GRAPH_COMPLETION: navegação pelo grafo de relacionamentos
    r3 = await cognee.search("relacionamentos entre módulos", query_type="GRAPH_COMPLETION")

    # SUMMARIES: resumos de alto nível
    r4 = await cognee.search("visão geral do projeto", query_type="SUMMARIES")

    for r in r1:
        print(f"[INSIGHT] {r}")

asyncio.run(exemplos_busca())
```

---

## 🔌 Integração via MCP com Claude Desktop / Cursor

Esta é a forma mais fácil para usuários não técnicos usarem o Cognee:

### Passo 1: Instalar o servidor MCP

```bash
pip install cognee-mcp
# ou
uvx cognee-mcp  # sem instalação permanente
```

### Passo 2: Configurar Claude Desktop

Edite `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "cognee": {
      "command": "uvx",
      "args": ["cognee-mcp"],
      "env": {
        "OPENAI_API_KEY": "sk-sua-chave",
        "LLM_API_KEY": "sk-sua-chave",
        "GRAPH_DATABASE_PROVIDER": "networkx",
        "DB_PROVIDER": "sqlite"
      }
    }
  }
}
```

### Passo 3: Reiniciar o Claude Desktop e usar

Agora no Claude Desktop você pode:

```
"Adicione ao meu grafo de conhecimento: [texto ou arquivo]"
"O que você sabe sobre [tema]?"
"Quais são as conexões entre [conceito A] e [conceito B]?"
"Mostre o que está no meu grafo sobre [empresa/projeto]"
```

---

## 🏢 Caso de Uso: Base de Conhecimento da Empresa

```python
import cognee
import asyncio
import os

async def criar_base_empresa():
    # Configurar dataset por departamento
    departamentos = {
        "rh": ["docs/rh/politicas.pdf", "docs/rh/beneficios.md"],
        "tech": ["docs/tech/arquitetura.md", "docs/tech/runbooks/"],
        "produto": ["docs/produto/roadmap.md", "docs/produto/specs/"],
    }

    for dept, fontes in departamentos.items():
        for fonte in fontes:
            await cognee.add(fonte, dataset_name=dept)

    await cognee.cognify()
    print("✅ Base de conhecimento da empresa criada!")

async def consultar(pergunta: str, dept: str = None):
    resultados = await cognee.search(
        query_text=pergunta,
        query_type="INSIGHTS"
    )
    return resultados

async def main():
    await criar_base_empresa()

    # Consultas práticas
    r = await consultar("Qual é o processo de onboarding para novos engenheiros?")
    for resultado in r:
        print(resultado)

asyncio.run(main())
```

---

## 🔄 Workflow Prático: Pesquisa Acumulativa

```python
import cognee
import asyncio

# Esse padrão permite ir adicionando conhecimento ao longo do tempo
# SEM resetar o grafo a cada vez

async def adicionar_ao_grafo(conteudo: str, dataset: str = "geral"):
    """Adiciona ao grafo existente sem apagar o anterior."""
    await cognee.add(conteudo, dataset_name=dataset)
    await cognee.cognify()  # reprocessa com o novo conteúdo integrado
    print(f"✅ Adicionado ao dataset '{dataset}'")

async def main():
    # Semana 1: adiciona contexto inicial
    await adicionar_ao_grafo("Projeto Alpha usa microsserviços com Docker", "alpha")

    # Semana 2: adiciona mais contexto — o grafo cresce
    await adicionar_ao_grafo("Alpha migrou para Kubernetes em agosto de 2026", "alpha")

    # Consulta reflete o conhecimento acumulado
    r = await cognee.search("Como evoluiu a infraestrutura do Projeto Alpha?", "INSIGHTS")
    for resultado in r:
        print(resultado)

asyncio.run(main())
```

---

## ⚡ Otimização de Tokens com Cognee

```python
# Busca precisa = menos tokens no contexto final
# Use CHUNKS para recuperar só o trecho exato necessário
chunks = await cognee.search(pergunta, query_type="CHUNKS")
contexto_minimal = "\n".join([str(c) for c in chunks[:2]])  # só 2 chunks

# Passe só o contexto minimal para o LLM
# Em vez de passar o documento inteiro
```

---

## 🔗 Recursos

- [GitHub](https://github.com/topoteretes/cognee)
- [Documentação](https://docs.cognee.ai)
- [MCP Setup](https://docs.cognee.ai/mcp)
- [← Pesquisa sobre Cognee](../06-cognee.md)
