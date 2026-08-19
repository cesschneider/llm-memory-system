# 🚀 Guia Prático: claude-mem / cmem — Memória para Claude Code

**Nível:** Iniciante  
**Tempo de setup:** 5 minutos  
**Pré-requisitos:** Claude Code instalado, Node.js  
**Site:** https://cmem.ai  

---

## 🎯 O que você vai conseguir

- ✅ Claude Code que lembra projetos, decisões e contexto entre sessões
- ✅ Zero repetição de contexto ao iniciar novas sessões
- ✅ Até 71.5x menos tokens por sessão
- ✅ Funciona automaticamente via hooks — sem esforço manual

---

## 📋 Pré-requisitos

```bash
# Verificar se Node.js está instalado
node --version   # precisa ser v18+

# Verificar se Claude Code está instalado
claude --version

# Se não tiver Node.js: https://nodejs.org/download
```

---

## 📦 OPÇÃO A — Instalação Local (Grátis, 5 minutos)

### Passo 1: Instalar claude-mem

```bash
npx claude-mem install
```

Isso automaticamente:
- Instala o pacote globalmente
- Configura os hooks no `~/.claude/settings.json`
- Cria o diretório `~/.claude-mem/` para armazenar memórias

### Passo 2: Verificar a instalação

```bash
# Ver memórias salvas
claude-mem list

# Deve mostrar: "No memories found" (vault vazio no início)
```

### Passo 3: Usar com Claude Code

Simplesmente abra o Claude Code normalmente:

```bash
claude
```

A partir de agora, a cada sessão o claude-mem:
1. **Ao iniciar:** injeta memórias relevantes no contexto automaticamente
2. **Durante:** captura decisões e contexto importantes
3. **Ao encerrar:** salva o que vale lembrar para a próxima sessão

---

## 📦 OPÇÃO B — cmem Cloud (Sync entre máquinas, 10 minutos)

### Passo 1: Instalar e criar conta

```bash
npx claude-mem install
npx claude-mem login
```

Siga o fluxo no browser para criar conta em cmem.ai.

### Passo 2: Verificar sync

```bash
claude-mem status
# Deve mostrar: ✅ Connected to cmem cloud
```

Agora suas memórias sincronizam entre diferentes computadores.

---

## 🛠️ Comandos Essenciais

```bash
# Ver todas as memórias
claude-mem list

# Salvar uma memória manualmente
claude-mem store "Este projeto usa TypeScript strict mode e Bun como runtime"
claude-mem store "O cliente prefere commits semânticos (feat:, fix:, docs:)"
claude-mem store "Banco de dados: PostgreSQL 16 com Prisma ORM"

# Buscar memórias relevantes
claude-mem recall "configuração do banco de dados"
claude-mem recall "preferências do cliente"

# Deletar uma memória pelo ID
claude-mem list   # pegar o ID
claude-mem delete <ID>

# Limpar tudo
claude-mem clear

# Exportar memórias (backup)
claude-mem export > backup-memorias.json

# Importar memórias
claude-mem import backup-memorias.json
```

---

## 📂 Por Projeto: Memórias com Namespace

O claude-mem organiza memórias por projeto automaticamente baseado no diretório:

```bash
# Em ~/projetos/app-vendas/
claude-mem store "Stack: Next.js + Supabase + Tailwind"
# Salvo como: namespace = "app-vendas"

# Em ~/projetos/api-pagamentos/
claude-mem store "Stack: FastAPI + PostgreSQL + Redis"
# Salvo como: namespace = "api-pagamentos"

# Claude Code em cada pasta só vê as memórias do projeto certo
```

---

## 🔧 Configuração Manual dos Hooks (avançado)

Se quiser controle total, edite `~/.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "claude-mem recall --format=json --limit=10"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "claude-mem store --auto"
          }
        ]
      }
    ]
  }
}
```

---

## 🔄 Workflows Práticos

### Workflow 1: Onboarding de Projeto Novo

```bash
cd ~/projetos/meu-novo-projeto

# Salve o contexto do projeto logo no início
claude-mem store "Projeto: Sistema de agendamento online para clínica veterinária"
claude-mem store "Stack: React 19, Node.js, MongoDB, Docker"
claude-mem store "Deploy: AWS ECS via GitHub Actions"
claude-mem store "Padrão de código: ESLint Airbnb, Prettier, commits convencionais"
claude-mem store "Contato do cliente: Dr. Carlos - prefere atualizações às sextas"

# Inicie Claude Code — ele já sabe tudo isso
claude
```

### Workflow 2: Registrar Decisões Técnicas

Durante ou após uma sessão com Claude:

```bash
claude-mem store "DECISÃO 2026-08: Usamos JWT com refresh tokens, não sessões. Motivo: app mobile precisa de auth stateless"
claude-mem store "DECISÃO 2026-08: Schema do banco normalizado até 3NF, sem desnormalização prematura"
claude-mem store "BUG CONHECIDO: Upload de avatar falha para arquivos > 5MB. Ticket #234 aberto"
```

### Workflow 3: Resumo de Sessão

Ao encerrar uma sessão longa, peça ao Claude:

```
Antes de encerrar, liste as 5 coisas mais importantes que discutimos
hoje para eu salvar na memória do projeto.
```

Copie e salve:
```bash
claude-mem store "Implementamos o módulo de pagamento com Stripe. Webhook endpoint: /api/webhooks/stripe"
claude-mem store "Pendente: testes de integração do módulo de pagamento"
# etc.
```

---

## 💡 Boas Práticas

```bash
# ✅ BOM: memórias específicas e acionáveis
claude-mem store "Variável de ambiente DB_URL deve apontar para localhost:5432/dev em desenvolvimento"

# ❌ RUIM: memórias vagas
claude-mem store "banco de dados importante"

# ✅ BOM: incluir data em decisões
claude-mem store "[2026-08] Mudamos de REST para GraphQL. PR #89 tem a implementação"

# ✅ BOM: prefixar por categoria
claude-mem store "[ARCH] Usamos arquitetura hexagonal (ports & adapters)"
claude-mem store "[ENV] Node 22 LTS em produção, 20 LTS em staging"
claude-mem store "[CLIENT] Cliente aprova PRs às segundas-feiras"
```

---

## 🔗 Combinando com Graphiti (Avançado)

Para memória em grafo além de vetores simples:

```bash
# Instalar Graphiti MCP
pip install graphiti-mcp

# Adicionar ao claude_desktop_config.json
# (veja guia 23 para detalhes)
```

---

## 🔗 Recursos

- [GitHub claude-mem](https://github.com/thedotmack/claude-mem)
- [cmem.ai](https://cmem.ai)
- [Claude Code Docs](https://docs.anthropic.com/claude-code)
- [← Pesquisa sobre claude-mem](../08-claude-mem.md)
