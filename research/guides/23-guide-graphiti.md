# 🚀 Guia Prático: Graphiti — Grafo de Conhecimento Temporal

**Nível:** Avançado  
**Tempo de setup:** 30–45 minutos  
**Pré-requisitos:** Python 3.10+, Docker  
**GitHub:** https://github.com/getzep/graphiti  

---

## 🎯 O que você vai conseguir

- ✅ Grafo de conhecimento onde cada fato tem validade temporal
- ✅ Fatos antigos invalidados automaticamente quando substituídos
- ✅ Integração via MCP com Claude e Cursor
- ✅ Memória que sabe a diferença entre "era verdade" e "é verdade agora"

---

## 📦 Instalação

```bash
pip install graphiti-core
```

---

## 🐳 Passo 1: Subir o Banco de Grafos

### Opção A: FalkorDB (mais leve, recomendado para começar)

```bash
docker run -d \
  --name falkordb \
  -p 6379:6379 \
  falkordb/falkordb:latest

# Verificar
docker ps | grep falkordb  # deve aparecer "Up"
```

### Opção B: Neo4j (mais poderoso, para produção)

```bash
docker run -d \
  --name neo4j \
  -e NEO4J_AUTH=neo4j/senha123 \
  -p 7474:7474 \
  -p 7687:7687 \
  -v neo4j_data:/data \
  neo4j:5-community

# Interface web: http://localhost:7474
# Login: neo4j / senha123
```

---

## 🚀 Exemplo 1: Começando com Graphiti

```python
import asyncio
from graphiti_core import Graphiti
from graphiti_core.nodes import EpisodeType
from datetime import datetime, timezone

async def main():
    # Conectar ao FalkorDB
    graphiti = Graphiti(
        neo4j_uri="bolt://localhost:6379",   # FalkorDB usa protocolo Bolt
        neo4j_user="",                        # FalkorDB: sem autenticação por padrão
        neo4j_password="",
        openai_api_key="SUA-OPENAI-KEY",
    )

    # Criar índices (só na primeira vez)
    await graphiti.build_indices_and_constraints()

    # ── ADICIONAR conhecimento ──
    agora = datetime.now(timezone.utc)

    await graphiti.add_episode(
        name="episodio_001",
        episode_body="Alice é gerente de projetos na empresa TechBr.",
        source_description="Perfil do LinkedIn",
        reference_time=agora,
        source=EpisodeType.message,
    )

    # Adicionar fato que atualiza o anterior
    await graphiti.add_episode(
        name="episodio_002",
        episode_body="Alice foi promovida a Diretora de Tecnologia na TechBr em agosto de 2026.",
        source_description="Anúncio interno",
        reference_time=agora,
        source=EpisodeType.message,
    )
    # Graphiti invalida automaticamente a relação "gerente de projetos"
    # e cria a nova relação "Diretora de Tecnologia" com a data correta

    # ── BUSCAR no grafo ──
    resultados = await graphiti.search("Qual é o cargo atual de Alice?")
    print("\n🧠 Resultados da busca:")
    for r in resultados:
        print(f"  ✓ {r.fact}")
        print(f"    Válido desde: {r.valid_at}")

asyncio.run(main())
```

---

## 🚀 Exemplo 2: Graphiti com Neo4j (Produção)

```python
import asyncio
from graphiti_core import Graphiti
from graphiti_core.nodes import EpisodeType
from datetime import datetime, timezone

async def main():
    # Conectar ao Neo4j
    graphiti = Graphiti(
        neo4j_uri="bolt://localhost:7687",
        neo4j_user="neo4j",
        neo4j_password="senha123",
        openai_api_key="SUA-OPENAI-KEY",
    )

    await graphiti.build_indices_and_constraints()

    # ── CRM com histórico temporal ──
    historico_cliente = [
        ("2024-01", "Carlos Silva assinou o plano Básico por R$99/mês"),
        ("2024-06", "Carlos fez upgrade para o plano Pro por R$299/mês"),
        ("2025-01", "Carlos migrou para o plano Enterprise por R$999/mês"),
        ("2026-08", "Carlos solicitou downgrade para o plano Pro por corte de budget"),
    ]

    for data, evento in historico_cliente:
        ref_time = datetime.fromisoformat(f"{data}-01T00:00:00+00:00")
        await graphiti.add_episode(
            name=f"crm_carlos_{data}",
            episode_body=evento,
            source_description="CRM sistema",
            reference_time=ref_time,
            source=EpisodeType.message,
        )

    # Buscar: qual é o plano ATUAL?
    resultados = await graphiti.search("Qual é o plano atual do Carlos Silva?")
    for r in resultados:
        print(f"Plano atual: {r.fact}")
    # Retorna: Pro (o mais recente e válido agora)

asyncio.run(main())
```

---

## 🔌 Integração MCP com Claude Desktop

Esta é a forma mais acessível de usar Graphiti sem código:

### Passo 1: Instalar servidor MCP

```bash
pip install graphiti-mcp
```

### Passo 2: Configurar Claude Desktop

Edite `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "graphiti": {
      "command": "python",
      "args": ["-m", "graphiti_mcp.server"],
      "env": {
        "NEO4J_URI": "bolt://localhost:7687",
        "NEO4J_USER": "neo4j",
        "NEO4J_PASSWORD": "senha123",
        "OPENAI_API_KEY": "SUA-OPENAI-KEY",
        "GROUP_ID": "meu_grafo"
      }
    }
  }
}
```

### Passo 3: Usar no Claude Desktop

Reinicie o Claude Desktop. Agora você pode:

```
"Adiciona ao grafo: João trabalha como DevOps na Nuvem SA desde jan/2026"
"Atualiza: João mudou de empresa para CloudBr em agosto de 2026"
"O que você sabe sobre a trajetória profissional do João?"
"Quais fatos sobre João ainda são válidos atualmente?"
```

---

## 🔧 Busca Avançada no Grafo

```python
import asyncio
from graphiti_core import Graphiti
from graphiti_core.search.search_config_recipes import (
    NODE_HYBRID_SEARCH_RRF,
    EDGE_HYBRID_SEARCH_RRF,
)

async def busca_avancada():
    graphiti = Graphiti(
        neo4j_uri="bolt://localhost:7687",
        neo4j_user="neo4j",
        neo4j_password="senha123",
        openai_api_key="SUA-OPENAI-KEY",
    )

    # Busca híbrida: vetorial + grafo estrutural
    # Retorna arestas (relacionamentos) relevantes
    arestas = await graphiti.search(
        query="histórico de empregos do usuário",
        config=EDGE_HYBRID_SEARCH_RRF,  # ranking por relevância combinada
        num_results=10,
    )

    print("=== RELACIONAMENTOS ENCONTRADOS ===")
    for aresta in arestas:
        print(f"\n🔗 {aresta.name}")
        print(f"   Fato: {aresta.fact}")
        print(f"   Válido em: {aresta.valid_at}")
        if aresta.invalid_at:
            print(f"   Invalidado em: {aresta.invalid_at} ← fato antigo")

    # Busca por nós (entidades)
    nos = await graphiti.search(
        query="empresas onde trabalhou",
        config=NODE_HYBRID_SEARCH_RRF,
    )
    print("\n=== ENTIDADES ENCONTRADAS ===")
    for no in nos:
        print(f"📍 {no.name}: {no.summary}")

asyncio.run(busca_avancada())
```

---

## 🔄 Workflow: Base de Conhecimento com Histórico Completo

```python
import asyncio
from graphiti_core import Graphiti
from graphiti_core.nodes import EpisodeType
from datetime import datetime, timezone

async def registrar_decisao(
    graphiti: Graphiti,
    decisao: str,
    contexto: str = "decisão técnica"
):
    """Registra uma decisão no grafo com timestamp."""
    await graphiti.add_episode(
        name=f"decisao_{datetime.now().strftime('%Y%m%d_%H%M')}",
        episode_body=decisao,
        source_description=contexto,
        reference_time=datetime.now(timezone.utc),
        source=EpisodeType.message,
    )
    print(f"✅ Decisão registrada: {decisao[:60]}...")

async def main():
    g = Graphiti(
        neo4j_uri="bolt://localhost:7687",
        neo4j_user="neo4j",
        neo4j_password="senha123",
        openai_api_key="SUA-OPENAI-KEY",
    )
    await g.build_indices_and_constraints()

    # Registrar decisões arquiteturais ao longo do tempo
    await registrar_decisao(g, "Usaremos REST API com FastAPI", "reunião de arquitetura")
    await registrar_decisao(g, "Migramos para GraphQL com Strawberry para melhor DX", "revisão técnica Q2")
    await registrar_decisao(g, "Revertemos para REST — GraphQL adicionou complexidade desnecessária", "retrospectiva Q3")

    # Consultar o histórico de decisões
    historico = await g.search("histórico de decisões sobre API")
    for h in historico:
        data = h.valid_at.strftime("%Y-%m") if h.valid_at else "?"
        invalido = f" → SUBSTITUÍDA em {h.invalid_at.strftime('%Y-%m')}" if h.invalid_at else " (ATUAL)"
        print(f"[{data}] {h.fact}{invalido}")

asyncio.run(main())
```

---

## ⚡ Dicas de Performance

```python
# Defina group_id para isolar grafos por projeto/usuário
await graphiti.add_episode(..., group_id="projeto_alpha")
resultados = await graphiti.search("...", group_ids=["projeto_alpha"])

# Para grandes volumes: processar episódios em batch
episodios = [...]
tasks = [graphiti.add_episode(**ep) for ep in episodios]
await asyncio.gather(*tasks)  # processa em paralelo
```

---

## 🔗 Recursos

- [GitHub](https://github.com/getzep/graphiti)
- [Documentação](https://help.getzep.com/graphiti)
- [MCP Setup](https://github.com/getzep/graphiti/tree/main/mcp_server)
- [← Pesquisa sobre Graphiti](../09-graphiti.md)
