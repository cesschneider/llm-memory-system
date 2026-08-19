# 🚀 Guia Prático: Memvid — Base de Conhecimento em Arquivo de Vídeo

**Nível:** Iniciante Python  
**Tempo de setup:** 5 minutos  
**Pré-requisitos:** Python 3.8+  
**GitHub:** https://github.com/Vortex-21/memvid  

---

## 🎯 O que você vai conseguir

- ✅ Transformar qualquer coleção de documentos em memória pesquisável
- ✅ Base de conhecimento inteira em **um único arquivo portável**
- ✅ Zero banco de dados, zero servidor — funciona offline
- ✅ Ideal para PDFs, artigos, documentação, livros

---

## 📦 Instalação

```bash
pip install memvid
```

Para suporte a PDFs:
```bash
pip install memvid PyMuPDF
```

---

## 🚀 Passo a Passo: Criar sua Primeira Base de Conhecimento

### Passo 1: Codificar conhecimento em vídeo

```python
from memvid import MemvidEncoder

encoder = MemvidEncoder()

# Adicionar texto diretamente
encoder.add_text("""
Python é uma linguagem de programação de alto nível, interpretada e
de propósito geral. Criada por Guido van Rossum e lançada em 1991,
Python enfatiza legibilidade de código e sintaxe limpa.
""")

# Adicionar múltiplos textos
textos = [
    "Machine Learning é um subcampo da IA que permite sistemas aprenderem com dados.",
    "Deep Learning usa redes neurais artificiais com múltiplas camadas.",
    "LLMs (Large Language Models) são modelos treinados em grandes volumes de texto.",
]
for texto in textos:
    encoder.add_text(texto)

# Adicionar de um arquivo PDF
encoder.add_pdf("meu_documento.pdf")

# Adicionar de uma lista de chunks
with open("artigo.txt", "r") as f:
    conteudo = f.read()
    # Divide em chunks de 500 caracteres automaticamente
    encoder.add_text(conteudo)

# Salvar como arquivo de vídeo (cria 2 arquivos)
encoder.build_video("base_conhecimento.mv2", "base_conhecimento.index")
print("✅ Base de conhecimento criada!")
print("   📹 base_conhecimento.mv2  — os dados codificados")
print("   📋 base_conhecimento.index — índice de busca semântica")
```

### Passo 2: Pesquisar na base de conhecimento

```python
from memvid import MemvidRetriever

# Carregar a base
retriever = MemvidRetriever("base_conhecimento.mv2", "base_conhecimento.index")

# Busca semântica simples
resultados = retriever.search("O que é machine learning?", top_k=3)

for chunk, score in resultados:
    print(f"[Relevância: {score:.3f}]")
    print(f"{chunk}\n")
```

### Passo 3: Conversar com sua base de conhecimento

```python
from memvid import MemvidChat

# Inicializar chat com a base
chat = MemvidChat(
    "base_conhecimento.mv2",
    "base_conhecimento.index",
    api_key="SUA-OPENAI-KEY"  # ou omitir para usar Ollama
)

# Chat interativo
chat.chat("Explique a diferença entre ML e Deep Learning")
```

---

## 📚 Caso de Uso 1: Base de Conhecimento Pessoal (Suas Notas do Obsidian)

```python
import os
from memvid import MemvidEncoder

encoder = MemvidEncoder()
vault_path = os.path.expanduser("~/Documents/MeuSegundoCerebro")

# Indexar todas as notas do Obsidian
arquivos_indexados = 0
for root, dirs, files in os.walk(vault_path):
    # Ignorar pastas de sistema
    dirs[:] = [d for d in dirs if not d.startswith('.')]
    for file in files:
        if file.endswith('.md'):
            caminho = os.path.join(root, file)
            try:
                with open(caminho, 'r', encoding='utf-8') as f:
                    conteudo = f.read()
                    if len(conteudo) > 50:  # ignorar notas vazias
                        encoder.add_text(conteudo)
                        arquivos_indexados += 1
            except Exception as e:
                print(f"Erro em {file}: {e}")

encoder.build_video("obsidian_vault.mv2", "obsidian_vault.index")
print(f"✅ {arquivos_indexados} notas indexadas!")
```

Agora pergunte qualquer coisa sobre suas notas:

```python
from memvid import MemvidRetriever
from openai import OpenAI

retriever = MemvidRetriever("obsidian_vault.mv2", "obsidian_vault.index")
client = OpenAI(api_key="SUA-KEY")

def perguntar_vault(pergunta: str) -> str:
    # Recuperar contexto relevante das notas
    chunks = retriever.search(pergunta, top_k=5)
    contexto = "\n\n---\n\n".join([chunk for chunk, score in chunks])

    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": f"Responda com base nestas notas do usuário:\n\n{contexto}"},
            {"role": "user", "content": pergunta}
        ]
    )
    return resp.choices[0].message.content

print(perguntar_vault("O que aprendi sobre Mem0?"))
print(perguntar_vault("Quais são meus projetos ativos?"))
```

---

## 📄 Caso de Uso 2: Q&A sobre Documentação Técnica

```python
from memvid import MemvidEncoder, MemvidRetriever

# Indexar documentação
encoder = MemvidEncoder()

# Pode adicionar URLs também (com requests)
import requests

urls = [
    "https://docs.python.org/3/tutorial/index.html",
    # adicione mais URLs
]

for url in urls:
    try:
        resp = requests.get(url, timeout=10)
        encoder.add_text(resp.text)
    except:
        pass

# Adicionar PDFs de manuais
for pdf in ["manual_v1.pdf", "api_reference.pdf"]:
    try:
        encoder.add_pdf(pdf)
    except:
        pass

encoder.build_video("documentacao.mv2", "documentacao.index")

# Consultar
retriever = MemvidRetriever("documentacao.mv2", "documentacao.index")
resultados = retriever.search("Como instalar e configurar o ambiente?", top_k=4)
for chunk, score in resultados:
    print(f"[{score:.2f}] {chunk[:200]}...\n")
```

---

## 🏢 Caso de Uso 3: Base de Conhecimento da Empresa (Compartilhável)

```python
# Líder técnico executa uma vez:
from memvid import MemvidEncoder

encoder = MemvidEncoder()

# Adicionar todos os documentos da empresa
documentos = [
    ("processos/onboarding.md", "text"),
    ("processos/deploy.md", "text"),
    ("docs/arquitetura.pdf", "pdf"),
    ("docs/api-reference.pdf", "pdf"),
    ("wiki/glossario.md", "text"),
]

for caminho, tipo in documentos:
    if tipo == "pdf":
        encoder.add_pdf(caminho)
    else:
        with open(caminho) as f:
            encoder.add_text(f.read())

# Gera 2 arquivos para distribuir ao time
encoder.build_video("conhecimento_empresa.mv2", "conhecimento_empresa.index")

# Compartilhe os 2 arquivos via Google Drive, S3, etc.
# Qualquer membro com Python pode usar instantaneamente:
```

```python
# Qualquer colaborador usa assim:
from memvid import MemvidRetriever

retriever = MemvidRetriever("conhecimento_empresa.mv2", "conhecimento_empresa.index")

resultado = retriever.search("Como fazer deploy em produção?", top_k=3)
for chunk, score in resultado:
    print(chunk)
```

---

## ⚡ Dicas de Performance

```python
# Para bases grandes: aumentar top_k e filtrar por score
resultados = retriever.search(query, top_k=10)
bons = [(chunk, score) for chunk, score in resultados if score > 0.6]

# Para documentos muito grandes: chunking manual
def chunkar_texto(texto: str, tamanho: int = 800, overlap: int = 100):
    chunks = []
    inicio = 0
    while inicio < len(texto):
        fim = inicio + tamanho
        chunks.append(texto[inicio:fim])
        inicio = fim - overlap
    return chunks

# Adicionar por chunks em vez de texto completo
for chunk in chunkar_texto(documento_grande):
    encoder.add_text(chunk)
```

---

## 🔄 Atualizar a Base de Conhecimento

```python
# Memvid é otimizado para leitura — para atualizar, reindexe:
encoder = MemvidEncoder()
# ... adicione todos os documentos novamente ...
encoder.build_video("base_conhecimento.mv2", "base_conhecimento.index")
# Sobrescreve os arquivos anteriores
```

> 💡 **Dica:** para bases que mudam com frequência, prefira Mem0 ou Zep. Memvid brilha em bases estáticas (documentação, livros, manuais).

---

## 📊 Comparativo: Quando Usar Memvid vs. Alternativas

| Situação | Use |
|----------|-----|
| Documentação estática que não muda | ✅ Memvid |
| Compartilhar base com alguém sem infra | ✅ Memvid |
| Precisa funcionar offline | ✅ Memvid |
| Memória dinâmica de conversas | ❌ Use Mem0 |
| Fatos que mudam com frequência | ❌ Use Zep |
| Equipe grande atualizando constantemente | ❌ Use Zep/Cognee |

---

## 🔗 Recursos

- [GitHub](https://github.com/Vortex-21/memvid)
- [PyPI](https://pypi.org/project/memvid/)
- [← Pesquisa sobre Memvid](../10-memvid.md)
