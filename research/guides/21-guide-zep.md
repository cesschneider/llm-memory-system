# 🚀 Guia Prático: Zep — Memória Temporal para Agentes Enterprise

**Nível:** Intermediário  
**Tempo de setup:** 15–20 minutos  
**Pré-requisitos:** Python 3.9+  
**Site:** https://www.getzep.com  
**Docs:** https://help.getzep.com  

---

## 🎯 O que você vai conseguir

- ✅ Memória que entende **quando** os fatos mudam (ex: "mudou de emprego")
- ✅ Sub-200ms de latência de recuperação
- ✅ Grafo de conhecimento temporal com invalidação automática de fatos antigos
- ✅ Pronto para produção: SOC 2 Type II, escalável para milhões de usuários

---

## 📦 Instalação

```bash
pip install zep-cloud
```

---

## 🔑 Passo 1: Criar conta e obter API Key

1. Acesse **https://www.getzep.com**
2. Clique em **"Get Started Free"**
3. Crie sua conta
4. Vá em **Settings → API Keys → Create Key**
5. Salve a chave (começa com `z_...`)

---

## 🚀 Exemplo 1: Memória Básica em 5 Minutos

```python
from zep_cloud.client import Zep
from zep_cloud.types import Message

# Conectar ao Zep Cloud
client = Zep(api_key="z_SUA_CHAVE_AQUI")

# ── CRIAR USUÁRIO ──
USER_ID = "usuario_maria"
try:
    client.user.add(
        user_id=USER_ID,
        email="maria@exemplo.com",
        first_name="Maria",
        last_name="Silva"
    )
except Exception:
    pass  # usuário já existe

# ── CRIAR SESSÃO (conversa) ──
SESSION_ID = "sessao_001"
client.memory.add_session(
    session_id=SESSION_ID,
    user_id=USER_ID
)

# ── ADICIONAR MENSAGENS ──
client.memory.add(SESSION_ID, messages=[
    Message(
        role="user",
        role_type="user",
        content="Meu nome é Maria e trabalho como gerente de produto na TechCorp."
    ),
    Message(
        role="assistant",
        role_type="assistant",
        content="Olá Maria! Entendido que você é PM na TechCorp."
    ),
    Message(
        role="user",
        role_type="user",
        content="Acabei de ser promovida a Diretora de Produto!"
    ),
])
print("✅ Mensagens adicionadas!")

# ── BUSCAR FATOS RELEVANTES ──
import time; time.sleep(2)  # aguardar processamento do grafo

fatos = client.memory.search_sessions(
    user_id=USER_ID,
    text="Qual é o cargo atual de Maria?",
    search_scope="facts",
    limit=5
)

print("\n🧠 Fatos sobre Maria:")
for resultado in fatos.results:
    print(f"  ✓ {resultado.fact}")
    # Zep vai mostrar "Diretora de Produto" como cargo atual
    # e invalidar automaticamente "Gerente de Produto"
```

---

## 🚀 Exemplo 2: Agente com Memória Temporal

```python
from zep_cloud.client import Zep
from zep_cloud.types import Message
from anthropic import Anthropic
import uuid

zep = Zep(api_key="z_SUA_CHAVE")
claude = Anthropic(api_key="SUA-ANTHROPIC-KEY")

USER_ID = "usuario_001"

# Setup inicial (executar uma vez por usuário)
try:
    zep.user.add(user_id=USER_ID, first_name="Usuário")
except:
    pass

def nova_sessao() -> str:
    session_id = f"sess_{uuid.uuid4().hex[:8]}"
    zep.memory.add_session(session_id=session_id, user_id=USER_ID)
    return session_id

def chat_com_memoria(mensagem: str, session_id: str) -> str:
    # 1. Recuperar memória relevante do Zep
    try:
        memoria = zep.memory.get(session_id)
        contexto_zep = memoria.context or ""
    except:
        contexto_zep = ""

    # 2. Buscar fatos específicos para a pergunta
    try:
        fatos = zep.memory.search_sessions(
            user_id=USER_ID,
            text=mensagem,
            search_scope="facts",
            limit=5
        )
        fatos_texto = "\n".join([f"- {f.fact}" for f in fatos.results])
    except:
        fatos_texto = ""

    # 3. Montar contexto completo
    sistema = "Você é um assistente pessoal com memória de longo prazo."
    if contexto_zep or fatos_texto:
        sistema += f"\n\nO que sei sobre o usuário:\n{fatos_texto}"
        if contexto_zep:
            sistema += f"\n\nContexto recente:\n{contexto_zep}"

    # 4. Chamar Claude
    resp = claude.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        system=sistema,
        messages=[{"role": "user", "content": mensagem}]
    )
    resposta = resp.content[0].text

    # 5. Salvar interação no Zep
    zep.memory.add(session_id, messages=[
        Message(role="user", role_type="user", content=mensagem),
        Message(role="assistant", role_type="assistant", content=resposta)
    ])

    return resposta

# Uso
session = nova_sessao()
print(chat_com_memoria("Moro em Curitiba e trabalho com finanças.", session))
print(chat_com_memoria("Mudei para Porto Alegre mês passado!", session))
print(chat_com_memoria("Em qual cidade estou morando?", session))
# Zep sabe que a cidade atual é Porto Alegre (Curitiba está invalidada)
```

---

## 🔧 Self-Hosted (Open Source, Grátis)

Para quem quer rodar o Zep localmente:

```bash
# Clonar e subir com Docker Compose
git clone https://github.com/getzep/zep.git
cd zep
docker-compose up -d

# Verificar que está rodando
curl http://localhost:8000/healthz
# {"status": "ok"}
```

```python
from zep_cloud.client import Zep

# Apontar para instância local
client = Zep(
    api_key="qualquer-coisa",  # self-hosted não valida a chave
    base_url="http://localhost:8000"
)
```

---

## 🔌 Integração com LangChain

```python
from langchain_community.memory import ZepChatMessageHistory
from langchain.chains import ConversationChain
from langchain_openai import ChatOpenAI

# Usar Zep como backend de memória no LangChain
history = ZepChatMessageHistory(
    session_id="sess_langchain_001",
    url="https://api.getzep.com",
    api_key="z_SUA_CHAVE"
)

# Adicionar ao chain normalmente
llm = ChatOpenAI(model="gpt-4o-mini")
chain = ConversationChain(
    llm=llm,
    verbose=True,
    memory=history
)

chain.predict(input="Meu nome é Pedro e sou arquiteto.")
chain.predict(input="Qual é o meu nome?")
```

---

## 🔄 Workflow: CRM com Memória Temporal

```python
from zep_cloud.client import Zep
from zep_cloud.types import Message
import datetime

zep = Zep(api_key="z_SUA_CHAVE")

def registrar_interacao_cliente(
    cliente_id: str,
    nota: str,
    tipo: str = "nota"
) -> None:
    """Registra uma interação com o cliente."""
    session_id = f"crm_{cliente_id}_{datetime.date.today().isoformat()}"

    try:
        zep.memory.add_session(session_id=session_id, user_id=cliente_id)
    except:
        pass

    zep.memory.add(session_id, messages=[
        Message(role="user", role_type="user", content=f"[{tipo.upper()}] {nota}")
    ])

def obter_contexto_cliente(cliente_id: str, pergunta: str) -> str:
    """Busca contexto relevante sobre um cliente."""
    fatos = zep.memory.search_sessions(
        user_id=cliente_id,
        text=pergunta,
        search_scope="facts",
        limit=10
    )
    return "\n".join([f"• {f.fact}" for f in fatos.results])

# Uso
registrar_interacao_cliente("cliente_abc", "João assinou plano Pro em jan/2026", "contrato")
registrar_interacao_cliente("cliente_abc", "João fez upgrade para Enterprise em ago/2026", "upgrade")
registrar_interacao_cliente("cliente_abc", "João solicitou desconto por renovação anual", "negociacao")

contexto = obter_contexto_cliente("cliente_abc", "Qual é o plano atual do cliente?")
print(contexto)
# Vai mostrar "Enterprise" como plano atual (Pro foi invalidado pelo upgrade)
```

---

## ⚡ Dicas de Otimização

```python
# 1. Use search_scope para buscar só o que precisa
fatos = zep.memory.search_sessions(
    user_id=USER_ID,
    text=query,
    search_scope="facts",   # só fatos extraídos (mais preciso)
    # search_scope="messages"  # mensagens brutas (mais abrangente)
    limit=5  # só os 5 mais relevantes
)

# 2. Use sessions por contexto (não uma sessão para tudo)
# Sessão por conversa = extração de memória mais precisa
# Sessão única = contexto misturado, memória degradada

# 3. Filtre por score de relevância
bons_fatos = [f for f in fatos.results if f.score > 0.7]
```

---

## 🔗 Recursos

- [Site Oficial](https://www.getzep.com)
- [Documentação](https://help.getzep.com)
- [GitHub](https://github.com/getzep/zep)
- [LangChain Integration](https://python.langchain.com/docs/integrations/memory/zep_memory/)
- [← Pesquisa sobre Zep](../02-zep.md)
