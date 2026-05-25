# AGENT-BUILDER.md

<!--
## Description:
Processo canônico para construção de Gemini Agents neste projeto.
Define o ciclo de 4 etapas: Objetivo → Decomposição → Sub-Agents → Orquestrador.
Alinhado com o Agent Designer do Gemini Enterprise (documentação Google, fev/2025).

## Usage Note:
Use este documento como referência principal ao criar qualquer novo Agent.
Leia integralmente antes de escrever qualquer instrução.

## Attribution:
Autoria de João Mello em colaboração com Claude (Anthropic).
Processo baseado na documentação oficial do Gemini Enterprise Agent Designer.
Modelo de Sub-Agent alinhado ao fluxo nativo do Agent Designer (Add subagent).

## Licensing:
MIT License

Date: May 25, 2026
-->

---

## Propósito deste documento

Este documento é a **referência de processo principal** para construir Gemini Agents neste projeto.

Todo Agent criado aqui segue uma arquitetura de **orquestrador + sub-agents**: o agente principal não executa tarefas diretamente — ele entende a intenção do usuário e delega para sub-agents especializados, cada um responsável por uma ação específica e bem definida.

A hipótese que este processo foi desenhado para testar:

> **Um orquestrador que delega para sub-agents especializados é mais confiável, sustentável e fácil de evoluir do que um agente único que acumula todas as capacidades em um bloco monolítico de instruções.**

Cada Agent construído com este processo é um dado de validação dessa hipótese. Registre suas observações em `## Post-Launch Notes` no arquivo `instructions.md` de cada Agent.

---

## Como o Agent Designer do Gemini Enterprise funciona

Antes de criar, entenda o modelo nativo da plataforma:

- O **Agent Designer** permite criar agentes single-step (um único bloco de instruções) e **multi-step** (um agente principal com sub-agents).
- Em um agente multi-step, o **agente principal é o orquestrador**: ele recebe a mensagem do usuário, decide qual sub-agent ativar, e delega a execução.
- Cada **sub-agent** é uma entidade independente com seu próprio nome, descrição e instruções — configurado separadamente no canvas Flow via **"Add subagent"**.
- O orquestrador e cada sub-agent têm seus próprios campos: **Name**, **Description**, **Instructions**, e opcionalmente **Starter Prompts** (no orquestrador).

Este processo de 4 etapas foi desenhado para preparar todo o conteúdo necessário antes de você abrir o Agent Designer — chegando na plataforma com clareza sobre o que criar.

---

## Ciclo de Construção em 4 Etapas

```
Etapa 1          Etapa 2              Etapa 3               Etapa 4
Objetivo    →    Decomposição    →    Sub-Agents    →    Orquestrador
(O que?)         (Quais ações?)       (Como cada um        (Como o todo
                                       funciona?)            funciona?)
```

---

### Etapa 1 — Entenda o Objetivo do Agent

Antes de qualquer decisão técnica, responda três perguntas em linguagem simples. Estas respostas guiam todas as etapas seguintes.

**1a. Qual problema este Agent resolve?**
Descreva o problema do usuário, não a solução técnica.

Bom: `"PMs perdem tempo tentando estruturar uma jornada do cliente do zero — não sabem por onde começar nem qual escopo priorizar."`
Ruim: `"Preciso de um agente que gera mapas de jornada."`

**1b. O que o usuário consegue fazer com este Agent que não conseguia antes?**
Esta é a proposta de valor. Uma frase.

Exemplo: `"Com este Agent, um PM consegue sair de um problema vago para um mapa de jornada estruturado com plano de intervenção em uma única sessão."`

**1c. O que este Agent explicitamente NÃO faz?**
Defina o limite de escopo agora, antes de qualquer instrução. Isso evita que o Agent vire um canivete suíço.

Exemplo: `"Este Agent não realiza pesquisa de mercado, priorização de backlog ou análise competitiva."`

> **Sinal de alerta:** Se você não consegue responder 1b em uma frase, o objetivo ainda não está claro o suficiente. Refine antes de avançar.

---

### Etapa 2 — Decomponha em Ações Distintas

Com o objetivo claro, mapeie **todas as ações discretas** que precisam acontecer para que o Agent entregue o resultado prometido.

#### Como decompor

Liste cada ação que o Agent precisa realizar, na ordem natural de execução. Use verbos de ação. Seja granular.

**Exemplo — Agent de Jornada do Cliente:**
1. Receber o contexto inicial do usuário
2. Identificar se o escopo está definido ou não
3. Facilitar a definição de persona e momento doloroso
4. Facilitar a escolha do escopo de jornada
5. Gerar o mapa de jornada em formato de tabela
6. Identificar os principais pontos de fricção
7. Gerar recomendações de intervenção
8. Listar assumptions a validar
9. Apresentar opções de próximo passo

#### Agrupe ações em clusters coesos

Depois de listar, agrupe as ações em clusters onde cada grupo representa uma **responsabilidade única e coesa**. Cada cluster vai se tornar um sub-agent.

**Regras de agrupamento:**
- Ações do mesmo cluster compartilham contexto e dependem umas das outras
- Ações de clusters diferentes podem ser executadas de forma independente
- Se um cluster tiver mais de 4–5 ações, considere dividir em dois
- Se um cluster tiver apenas 1 ação simples, considere fundir com outro

**Exemplo — clusters do Agent de Jornada:**

| Cluster | Ações incluídas | Nome do Sub-Agent |
|---------|----------------|-------------------|
| A | 1, 2, 3, 4 | Facilitador de Escopo |
| B | 5, 6 | Gerador de Jornada |
| C | 7, 8, 9 | Planejador de Intervenção |

> **Princípio central:** Se você precisar escrever "e também" na descrição de um cluster, ele precisa ser dividido.

---

### Etapa 3 — Escreva as Instruções de Cada Sub-Agent

Cada cluster identificado na Etapa 2 se torna um sub-agent independente. Escreva as instruções de cada um usando os **quatro pilares**: Persona · Task · Context · Format.

#### O que é um Sub-Agent neste contexto

No Agent Designer do Gemini Enterprise, cada sub-agent é configurado separadamente com seu próprio conjunto de campos. Aqui você prepara o conteúdo desses campos antes de entrar na plataforma.

**Campos que você vai preencher no Agent Designer para cada sub-agent:**
- **Name** — nome claro e específico à função
- **Description** — uma frase descrevendo o que este sub-agent faz
- **Instructions** — o bloco completo de instruções (Persona · Task · Context · Format)

#### Template de Instrução para Sub-Agent

```
# Sub-Agent: [Nome]

## Metadados (para o Agent Designer)
- Name: [Nome do sub-agent — claro e específico]
- Description: [Uma frase: o que este sub-agent faz e quando é ativado]

## Persona
[Qual especialista este sub-agent encarna? Qual é a voz e o tom?
Como ele responde quando recebe uma solicitação?]

## Task
[O que este sub-agent produz? Seja específico sobre o entregável.
Liste as ações que ele executa, na ordem.]

## Context
Idioma: Sempre responda em português brasileiro, independentemente do idioma usado pelo usuário.

Input esperado: [O que este sub-agent recebe do orquestrador ou do usuário?]

Regras de operação:
- [Regra 1 — comportamento específico deste sub-agent]
- [Regra 2]
- [Regra 3]

Contexto ausente: [Se o input necessário estiver faltando, o que o sub-agent deve fazer?
Máximo de 2 perguntas antes de prosseguir com assumptions rotuladas.]

Limites: [O que este sub-agent NÃO faz. Redireciona para quem?]

## Format
Output contract: [Descrição precisa do formato de saída — estrutura, seções, extensão.]

[Template de output, se aplicável:]
---
[seção 1]
[seção 2]
[seção 3]
---
```

#### Regras de escrita para sub-agents

1. **Instruções autocontidas.** Cada sub-agent deve funcionar sem depender de memória de outras sessões ou de outros sub-agents. Inclua no Context tudo que ele precisa saber.
2. **Output contract preciso.** O orquestrador depende de um output previsível para continuar o fluxo. Defina formato, seções e extensão esperada.
3. **Limites explícitos.** Diga ao sub-agent o que ele não deve fazer e para onde redirecionar se o usuário pedir algo fora do escopo.
4. **Sem comentários `<!-- -->`** no texto final — toda lógica deve estar em texto visível e executável.

#### Exemplo — Sub-Agent: Facilitador de Escopo

```
# Sub-Agent: Facilitador de Escopo

## Metadados
- Name: Facilitador de Escopo
- Description: Guia o usuário para definir persona, momento doloroso e escopo
  de jornada antes de qualquer geração de conteúdo.

## Persona
Você é um consultor estratégico de produto com experiência em facilitação
de discovery. Seu estilo é colaborativo, curioso e direto. Você nunca gera
um mapa sem antes entender claramente quem é o usuário e qual é o problema.
Quando recebe uma solicitação vaga, você faz perguntas cirúrgicas — uma
por vez — até ter o contexto mínimo necessário.

## Task
Guiar o usuário através de uma sequência de decisões para definir:
1. A persona-alvo da jornada
2. O momento doloroso ou oportunidade central
3. O escopo da jornada (quais etapas cobrir)

Ao final, produza um resumo das 3 decisões tomadas para ser usado pelo
próximo sub-agent.

## Context
Idioma: Sempre responda em português brasileiro, independentemente do
idioma usado pelo usuário.

Input esperado: Qualquer descrição inicial do usuário sobre o que ele
quer mapear — pode ser vaga ou detalhada.

Regras de operação:
- Faça uma pergunta por vez. Nunca agrupe múltiplas perguntas.
- Colete contexto mínimo primeiro, depois proponha opções.
- Em cada decisão, ofereça exatamente 3 opções. A recomendada sempre
  primeiro, com uma linha de justificativa. Adicione tradução de negócio
  para cada opção.
- Aceite respostas numéricas (1, 2, 3) ou direção personalizada.
- Mostre progresso após cada resposta: "Progresso: 2/3 decisões tomadas."

Contexto ausente: Se o usuário não mencionar nem uma persona nem um
problema, pergunte pelo problema primeiro. Máximo de 2 perguntas antes
de propor opções com base em assumptions rotuladas.

Limites: Este sub-agent não gera mapas de jornada nem planos de
intervenção. Ao concluir as 3 decisões, entrega o resumo e indica
que o próximo passo é a geração da jornada.

## Format
Output contract: Bloco de texto estruturado com as 3 decisões, pronto
para ser consumido pelo sub-agent Gerador de Jornada.

---
**Resumo de Escopo — Jornada do Cliente**

- **Persona:** [nome/perfil definido]
- **Momento doloroso / Oportunidade:** [descrição da situação central]
- **Escopo da jornada:** [etapas que serão mapeadas]

*Pronto para geração do mapa. Confirma ou quer ajustar algum ponto?*
---
```

---

### Etapa 4 — Escreva as Instruções do Orquestrador

O orquestrador é o agente principal. Ele é o primeiro ponto de contato com o usuário e o responsável por entender a intenção e delegar para o sub-agent correto.

**O orquestrador não executa tarefas.** Ele:
- Recebe a mensagem do usuário
- Identifica a intenção
- Ativa o sub-agent adequado
- Mantém a coerência da experiência ao longo da sessão

#### Campos do Orquestrador no Agent Designer

| Campo | O que colocar |
|-------|--------------|
| **Name** | Nome público do Agent — claro e orientado ao usuário |
| **Description** | Uma frase: o que este Agent faz e para quem |
| **Instructions** | Bloco completo (Persona · Task · Context · Format) |
| **Starter Prompts** | 4 prompts que ativam fluxos diferentes |

#### Template de Instrução para o Orquestrador

```
# Orquestrador: [Nome do Agent]

## Persona
[Qual é a identidade deste Agent para o usuário? Qual expertise ele
representa? Como ele se apresenta quando cumprimentado? O que ele diz
quando o usuário pergunta "o que você faz?"

Inclua: papel, voz, tom, e uma frase de boas-vindas com exemplo de uso.]

## Task
Este Agent orquestra um fluxo de [N] etapas para ajudar o usuário a
[objetivo definido na Etapa 1b].

Ele não executa tarefas diretamente. Ele identifica a intenção do usuário
e delega para o sub-agent correspondente:

| Intenção do usuário | Sub-Agent ativado |
|---------------------|------------------|
| [Condição de ativação] | [Nome do Sub-Agent] |
| [Condição de ativação] | [Nome do Sub-Agent] |
| [Condição de ativação] | [Nome do Sub-Agent] |

## Context
Idioma: Sempre responda em português brasileiro, independentemente do
idioma usado pelo usuário.

Comportamento de boas-vindas: Quando o usuário abrir a sessão sem contexto,
apresente-se em 2–3 linhas e ofereça um exemplo concreto de uso. Não liste
todas as capacidades — mostre o fluxo mais comum.

Regras de orquestração:
1. Sempre identifique a intenção antes de delegar. Se a intenção não
   estiver clara, faça uma única pergunta de clarificação.
2. Delegue para apenas um sub-agent por vez.
3. Ao receber o output de um sub-agent, apresente-o ao usuário e pergunte
   se deseja continuar para a próxima etapa ou ajustar algo.
4. Mantenha o histórico de decisões da sessão visível para o usuário.
5. Se o usuário pedir algo fora do escopo do Agent, reconheça e redirecione
   sem julgamento. Não tente executar o que está além do escopo.

Fluxo padrão da sessão:
[Descreva a sequência esperada de ativação dos sub-agents para o caso de uso principal.]

Exemplo:
1. Usuário descreve o problema → Orquestrador ativa [Sub-Agent A]
2. [Sub-Agent A] entrega resumo → Orquestrador apresenta e confirma
3. Usuário confirma → Orquestrador ativa [Sub-Agent B]
4. [Sub-Agent B] entrega output → Orquestrador apresenta e oferece próximos passos

Limites do Agent: [O que este Agent não faz — repita o definido na Etapa 1c.]

## Format
O orquestrador não tem um formato de output fixo — ele adapta a apresentação
ao conteúdo entregue por cada sub-agent.

Regras de apresentação:
- Ao apresentar o output de um sub-agent, adicione uma linha de contexto
  antes ("Aqui está o resultado da etapa X:") e uma pergunta de continuidade
  depois ("Quer prosseguir para [próxima etapa] ou ajustar algo?").
- Mantenha respostas do orquestrador curtas. O conteúdo fica nos sub-agents.
- Encerre toda sessão com: decisões tomadas, assumptions a validar, e
  exatamente 3 opções de próximo passo (opção 1 = Recomendada).
```

#### Starter Prompts do Orquestrador

Todo orquestrador deve ter **4 Starter Prompts** — frases clicáveis que ensinam o usuário a interagir antes de qualquer conversa.

**Regras:**
1. Cada prompt ativa um fluxo diferente (idealmente um sub-agent diferente).
2. Escreva em primeira pessoa do usuário.
3. Inclua contexto suficiente para que o orquestrador saiba qual sub-agent ativar.
4. Evite prompts genéricos como "Como funciona?" ou "Pode me ajudar?".

**Template:**
```
Starter Prompt 1: [Fluxo completo — caso de uso mais comum]
Starter Prompt 2: [Ativa um sub-agent específico diretamente]
Starter Prompt 3: [Usuário sem contexto — testa o comportamento de boas-vindas]
Starter Prompt 4: [Caso de borda — contexto parcial ou pedido incomum]
```

---

## Estrutura de Arquivos por Agent

Cada Agent criado neste projeto deve ter sua própria pasta em `agents/` com a seguinte estrutura:

```
agents/
└── [nome-do-agent]/
    ├── overview.md          # Etapas 1 e 2: objetivo, ações e clusters
    ├── subagent-[nome].md   # Etapa 3: instrução de cada sub-agent (um arquivo por sub-agent)
    └── instructions.md      # Etapa 4: instrução do orquestrador + Starter Prompts
```

O arquivo `instructions.md` é o que vai para o campo **Instructions** do agente principal no Agent Designer. Os arquivos `subagent-*.md` vão para o campo **Instructions** de cada sub-agent configurado via "Add subagent".

---

## Checklist de Qualidade

Antes de publicar qualquer Agent no Agent Designer, verifique:

**Etapa 1 — Objetivo**
- [ ] O problema do usuário está descrito (não a solução técnica)
- [ ] A proposta de valor está em uma frase
- [ ] O limite de escopo está definido explicitamente

**Etapa 2 — Decomposição**
- [ ] Todas as ações necessárias estão listadas
- [ ] As ações estão agrupadas em clusters coesos
- [ ] Cada cluster tem uma responsabilidade única (sem "e também")
- [ ] O número de sub-agents está definido

**Etapa 3 — Sub-Agents**
- [ ] Cada sub-agent tem: Name, Description e Instructions completas
- [ ] Instruções seguem os 4 pilares (Persona · Task · Context · Format)
- [ ] Regra de idioma presente em cada sub-agent: "Sempre responda em português brasileiro..."
- [ ] Output contract está definido com precisão
- [ ] Limites estão explícitos (o que o sub-agent não faz)
- [ ] Contexto ausente tem tratamento definido (máx. 2 perguntas)
- [ ] Nenhum bloco `<!-- -->` no texto final

**Etapa 4 — Orquestrador**
- [ ] Regra de idioma presente: "Sempre responda em português brasileiro..."
- [ ] Persona inclui comportamento de boas-vindas
- [ ] Tabela de delegação lista todos os sub-agents com condições de ativação
- [ ] Fluxo padrão da sessão está descrito
- [ ] Limites do Agent estão explícitos
- [ ] Encerramento padrão definido (decisões + assumptions + 3 próximos passos)
- [ ] 4 Starter Prompts configurados, cada um ativando um fluxo diferente

---

## Mapeamento: product-manager-prompts → gemini-agents

Use esta tabela ao adaptar assets do repositório fonte para este formato.

| Elemento do PM Prompt | → Destino no Agent Builder |
|----------------------|---------------------------|
| AI role e voz | → **Persona** do orquestrador ou sub-agent |
| Entregável principal | → **Task** do sub-agent responsável |
| Regras de facilitação, PDF Loop | → **Context** do sub-agent facilitador |
| Required Context Keys | → **Context** (Input esperado) de cada sub-agent |
| Missing Context Rule | → **Context** (Contexto ausente) — máx. 2 perguntas |
| Output Format com template | → **Format** (Output contract) do sub-agent |
| Final Step options | → **Format** do orquestrador (Encerramento padrão) |
| Múltiplas seções de output | → Decomposição em sub-agents distintos (Etapa 2) |
| Attribution / Licensing | → Omitir das instruções; preservar no `overview.md` |

---

## Decisão de Design: Orquestrador + Sub-Agents vs. Agente Monolítico

| Dimensão | Orquestrador + Sub-Agents | Agente Monolítico |
|----------|--------------------------|-------------------|
| Previsibilidade de output | Alta — cada sub-agent tem output contract fixo | Variável — depende do contexto acumulado |
| Facilidade de manutenção | Alta — edite um sub-agent sem afetar os outros | Baixa — mudanças em uma parte cascateiam |
| Clareza de responsabilidade | Alta — cada sub-agent tem uma função clara | Baixa — tudo misturado em um bloco |
| Evolução incremental | Alta — adicione sub-agents sem reescrever o todo | Baixa — qualquer adição exige revisão completa |
| Risco de deriva de escopo | Baixo — limites explícitos por sub-agent | Alto — o agente acumula comportamentos com o tempo |
| Curva de aprendizado inicial | Moderada — requer planejamento antes de escrever | Baixa — começa rápido, mas escala mal |

**Como registrar observações:** Ao encerrar uma sessão de uso com um Agent construído neste modelo, adicione uma entrada em `## Post-Launch Notes` no `instructions.md` do Agent com:
- Data
- Qual sub-agent foi mais ativado
- Onde a delegação falhou (se falhou)
- Se os output contracts foram respeitados
- Comparação subjetiva com a abordagem de agente monolítico equivalente

---

## Arquivos Relacionados

| Arquivo | Função |
|---------|--------|
| `README.md` | Visão geral do repositório e como usar |
| `agents/README.md` | Catálogo de todos os Agents disponíveis |
| `agents/[nome]/overview.md` | Objetivo e decomposição de cada Agent (Etapas 1–2) |
| `agents/[nome]/subagent-*.md` | Instruções de cada sub-agent (Etapa 3) |
| `agents/[nome]/instructions.md` | Instrução do orquestrador (Etapa 4) |
