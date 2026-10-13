# ⚽ Agente RAG: Regras do Futebol

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SEU-USUARIO/rag-regras-futebol/blob/main/rag_regras_futebol.ipynb)

Assistente que responde perguntas sobre as **Regras do Jogo** de futebol (IFAB, edição 2023/24, em português) usando **RAG (Retrieval-Augmented Generation)**.

Em vez de depender só do que o modelo de linguagem "lembra", o agente busca os trechos relevantes no documento oficial e responde com base neles, indicando a página de onde tirou a informação. Quando a pergunta não é coberta pelas regras, ele diz isso em vez de inventar uma resposta.

<!-- Adicione aqui um print de uma resposta do notebook -->

## Como funciona

```
PDF das Regras → divisão em trechos → embeddings → banco vetorial (Chroma)
                                                          │
Pergunta → busca dos trechos mais parecidos ──────────────┘
         → prompt com o contexto → LLM → resposta com a página de origem
```

1. **Carregamento:** o PDF oficial é baixado do site da IFAB e lido página a página.
2. **Divisão:** o texto é quebrado em trechos de 500 caracteres, com 100 de sobreposição para não cortar frases importantes.
3. **Indexação:** cada trecho vira um vetor (embedding) e é guardado no Chroma.
4. **Busca:** para cada pergunta, os 3 trechos semanticamente mais próximos são recuperados.
5. **Geração:** os trechos entram no prompt, que instrui o modelo a responder apenas com base neles, citar a página e admitir quando a informação não está nas regras.

## Tecnologias

| Componente | Ferramenta |
|---|---|
| Orquestração | LangChain (LCEL) |
| Modelo de linguagem | OpenAI `gpt-4o-mini` |
| Embeddings | OpenAI `text-embedding-3-small` |
| Banco vetorial | Chroma |
| Leitura do PDF | PyPDF |

## Exemplos de perguntas

| Pergunta | O que se espera |
|---|---|
| Um jogador pode usar a mão para marcar um gol? | Resposta baseada na Regra 12, com a página |
| O que caracteriza a posição de impedimento? | Resposta baseada na Regra 11, com a página |
| Qual é o número mínimo de jogadores para uma partida continuar? | Resposta baseada na Regra 3, com a página |
| Quem ganhou a Copa do Mundo de 2022? | O agente informa que isso não consta nas Regras do Jogo |

A última pergunta é proposital: ela verifica se o agente se limita ao documento em vez de usar o conhecimento geral do modelo.

## Como executar

### No Google Colab (recomendado)

1. Clique no botão **Abrir no Colab** acima.
2. No menu lateral, abra **Secrets** (🔑) e crie o segredo `OPENAI_API_KEY` com a sua chave da OpenAI.
3. Execute as células em ordem (*Ambiente de execução → Executar tudo*).

### Localmente

```bash
git clone https://github.com/SEU-USUARIO/rag-regras-futebol.git
cd rag-regras-futebol
python -m venv .venv
source .venv/bin/activate        # no Windows: .venv\Scripts\activate
pip install -r requirements.txt jupyter
export OPENAI_API_KEY="sua-chave"  # no Windows: set OPENAI_API_KEY=sua-chave
jupyter notebook rag_regras_futebol.ipynb
```

O PDF é baixado automaticamente na primeira execução. Se o download falhar, coloque o arquivo das Regras do Jogo 2023/24 em português na pasta do projeto com o nome `regras_futebol.pdf`.

## Limitações

- As respostas dependem da qualidade da busca: se os trechos recuperados não contiverem a resposta, o agente informa que não encontrou a informação, mesmo que ela exista em outra parte do documento.
- O conteúdo cobre apenas a edição 2023/24 das Regras do Jogo.
- O uso da API da OpenAI tem custo, embora baixo para este volume: a indexação completa do documento e algumas dezenas de perguntas custam bem menos de um dólar.

## Créditos

Projeto desenvolvido a partir de uma aula da [Alura](https://www.alura.com.br/) sobre RAG com LangChain. Em relação à versão original, foram feitas as seguintes mudanças:

- reescrita da cadeia no padrão atual do LangChain (LCEL), substituindo o `RetrievalQA`, que foi descontinuado;
- prompt próprio em português, com instrução para citar a página e não responder fora do documento;
- teste com pergunta fora do escopo, para validar que o agente não inventa respostas;
- correção da duplicação de trechos no banco vetorial ao reexecutar o notebook;
- download automático do PDF e leitura da chave sem expô-la no código.

O documento das Regras do Jogo pertence à [IFAB](https://www.theifab.com/) e não é distribuído neste repositório.
