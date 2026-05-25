# AGENT-BUILDER.md

<!--
## Description:
Processo canônico para construção de Gemini Agents neste projeto.
Estende a estrutura de quatro pilares (Persona · Task · Context · Format)
com o ciclo de vida do Gemini Enterprise Agent Designer, adaptado para
focar em agrupamentos de Sub-Agents como Skills.

## Usage Note:
Use este documento como referência principal ao criar qualquer novo Agent.
Leia integralmente antes de escrever qualquer instrução.

## Attribution:
Autoria de João Mello em colaboração com Claude (Anthropic).
Estrutura do Agent Designer adaptada da documentação do Gemini Enterprise.
Modelo de Sub-Agent inspirado em padrões de arquitetura modular de skills.

## Licensing:
MIT License

Date: May 25, 2026
-->

---

## Propósito deste documento

Este documento é a **referência de processo principal** para construir Gemini Agents neste projeto.

Ele introduz uma escolha arquitetural deliberada: **agrupar capacidades relacionadas como Sub-Agents (Skills) dentro de um único Agent**, em vez de construir muitos Agents independentes e roteá-los dentro de um orquestrador genérico.

A hipótese que este processo foi desenhado para testar:

> **Um Agent focado com Sub-Agent Skills bem definidos é mais confiável, aprendível e sustentável do que um orquestrador único que acumula Agents independentes.**

Cada Agent construído com este processo é um dado de validação dessa hipótese. Registre suas observações em `## Post-Launch Notes` no arquivo de instrução de cada Agent.

---

## Ciclo de Construção em 3 Etapas

### Etapa 1 — Defina Identidade e Propósito

Antes de escrever qualquer instrução, responda estas três perguntas por escrito. Elas se tornam a descrição pública do Agent.

**1a. Nome**
Claro, específico ao papel, e instantaneamente reconhecível para alguém que nunca usou o Agent.

Bom: `Facilitador de Jornada do Cliente`
Ruim: `Agente PM 3` ou `Helper`

**1b. Goal Description (uma frase)**
O que este Agent ajuda o usuário a realizar? Esta é a resposta para "quando devo usar este Agent versus outro?"

Exemplo: `Ajuda PMs a construir mapas de jornada do cliente através de facilitação estruturada, do mapeamento de fricção até o plano de intervenção.`

**1c. Capability Boundary**
O que está explicitamente FORA do escopo? Isso protege o Agent contra expansão de escopo e o mantém focado.

Exemplo: `Este Agent não executa pesquisa de mercado, priorização de backlog, ou análise competitiva.`

---

### Etapa 2 — Desenhe a Arquitetura de Sub-Agent Skills

Esta é a parte estrutural central do processo. Em vez de escrever um grande bloco de instrução, decomponha as capacidades do Agent em **Sub-Agents discretos**, cada um responsável por uma skill única e bem definida.

#### O que é um Sub-Agent (Skill)?

Um Sub-Agent é um módulo de capacidade autocontido dentro do Agent. Ele tem:
- **Trigger** — quando o Agent principal ativa este skill?
- **Modo de persona** — a voz do Agent muda? Se não, escreva "mantém voz principal".
- **Escopo de task** — o que este skill produz?
- **Output contract** — como é o output quando o skill termina?

Sub-Agents **não são Gemini Agents separados**. São modos comportamentais declarados dentro do conjunto de instruções de um único Agent, ativados pelo Agent orquestrador com base no contexto do usuário.

#### Regras de Design de Sub-Agents

1. **Uma skill, um trabalho.** Cada Sub-Agent faz uma coisa bem. Se você escrever "e também" na descrição de task de um Sub-Agent, divida-o.
2. **Nomeie cada skill explicitamente.** Dê um rótulo que o Agent principal possa referenciar por nome (ex: `[Skill: Facilitação de Escopo]`).
3. **Defina a condição de ativação.** O Agent principal precisa saber exatamente quando invocar cada skill. Escreva a condição em linguagem simples.
4. **Defina o output contract.** O skill deve produzir um artefato ou formato de resposta específico e previsível. O Agent principal depende disso para continuar.
5. **Skills são composáveis.** Uma única sessão de usuário pode ativar múltiplos skills em sequência. Desenhe cada skill para que seu output possa ser consumido pelo próximo.

#### Template de Sub-Agent Map

```
## Sub-Agent Map

### [Skill: Nome do Skill]
- Trigger: [Quando este skill é ativado — o que o usuário disse ou fez?]
- Persona mode: [Há mudança de tom ou voz? Se não, escreva "mantém voz principal".]
- Task: [O que este skill produz?]
- Output contract: [Como é o output? Formato, seções, extensão esperada.]
- Hands off to: [Nome do próximo skill, ou "encerramento" se for o último.]

### [Skill: Nome do Skill]
...
```

#### Exemplo de Sub-Agent Map (Agent de Jornada do Cliente)

```
## Sub-Agent Map

### [Skill: Facilitação de Escopo]
- Trigger: Usuário inicia sessão ou não tem escopo definido.
- Persona mode: Mantém voz principal (consultor estratégico colaborativo).
- Task: Guiar o usuário para definir persona, momento doloroso, e escopo de jornada.
- Output contract: Resumo de 3 decisões — persona, escopo, e foco de resultado.
- Hands off to: [Skill: Geração de Jornada]

### [Skill: Geração de Jornada]
- Trigger: Escopo definido (saída do Skill anterior confirmada).
- Persona mode: Mantém voz principal.
- Task: Produzir o mapa de jornada completo em formato de tabela Markdown canônico.
- Output contract: Tabela com linhas fixas × colunas de etapas + Top 3 pontos de fricção.
- Hands off to: [Skill: Plano de Intervenção]

### [Skill: Plano de Intervenção]
- Trigger: Tabela de jornada gerada e confirmada pelo usuário.
- Persona mode: Mantém voz principal.
- Task: Gerar recomendações de intervenção e assumptions a validar.
- Output contract: Lista numerada de intervenções + bullets de assumptions + 3 opções de próximo passo.
- Hands off to: Encerramento.
```

---

### Etapa 3 — Escreva o Conjunto de Instruções com os Quatro Pilares

Com o Sub-Agent Map definido, traduza a arquitetura em texto de instrução do Agent usando os quatro pilares: **Persona · Task · Context · Format**.

O Sub-Agent Map fica na seção **Context** como tabela de roteamento explícita.

#### Template de Conjunto de Instruções

```
## Persona
[Define o papel que o Gemini desempenha. Qual expertise ele incorpora?
Qual é a voz e a atitude? O que ele faz quando cumprimentado ou quando
o usuário pergunta o que ele pode fazer?]

## Task
[Declare o entregável principal claramente. O que este Agent cria ou
facilita? Liste os Sub-Agents ativos como subtarefas explícitas.]

Subtarefas (Skills ativos):
- [Skill: Nome] — [uma linha descrevendo o que produz]
- [Skill: Nome] — [uma linha descrevendo o que produz]
- [Skill: Nome] — [uma linha descrevendo o que produz]

## Context
Idioma: Sempre responda em português brasileiro, independentemente do idioma usado pelo usuário.

Comportamento de boas-vindas: [O que o Agent diz quando o usuário abre
a sessão sem contexto? Inclua uma frase curta de orientação e um exemplo de uso.]

Sub-Agent Routing:
[Cole aqui o Sub-Agent Map completo da Etapa 2.]

Regras de facilitação (remova se o Agent for modo execução pura):
1. Faça uma pergunta por vez. Nunca agrupe múltiplas perguntas.
2. Aplique inversão de esforço: colete contexto mínimo primeiro, depois proponha opções.
3. Em cada decisão, ofereça exatamente 3 opções. A recomendada sempre primeiro, com justificativa.
4. Fraseie as opções na linguagem da persona primeiro; adicione tradução de negócio depois.
5. Aceite `1`, `2`, `3`, `1 e 3`, ou direção personalizada.
6. Mostre progresso depois de cada resposta (ex: `Progresso: 2/4 inputs capturados`).
7. Encerre toda sessão com: decisões tomadas, assumptions a validar, 3 opções de próximo passo.

Limites de escopo:
- [O que este Agent NÃO faz. Seja explícito.]
- Se o usuário pedir algo fora do escopo, o Agent reconhece e redireciona sem julgamento.

Contexto ausente:
- Se a informação necessária para ativar um Skill estiver faltando, faça no máximo
  3 perguntas (uma por vez) antes de prosseguir com assumptions explicitamente rotuladas.

## Format
[Descreva a estrutura de output de cada Skill com precisão. Inclua templates
de seção, exemplos de formatação, orientação de extensão e qualquer template
canônico a preservar.]

### Output do [Skill: Nome]
[Template específico deste skill]

### Output do [Skill: Nome]
[Template específico deste skill]

### Encerramento padrão
Toda sessão encerra com exatamente N opções de próximo passo numeradas de 1 a N,
com a opção 1 marcada como Recomendada.
```

---

## Starter Prompts (UX de Entrada)

Todo Agent deve ter **4 Starter Prompts** — frases clicáveis que aparecem na tela inicial do Gemini e ensinam o usuário a interagir antes de qualquer conversa.

### Regras para Starter Prompts

1. Cada prompt deve ativar um fluxo diferente (idealmente um Sub-Agent diferente).
2. Escreva em primeira pessoa do usuário: "Quero mapear...", "Me ajuda a...", "Cria um..."
3. Inclua contexto suficiente para que o Agent saiba qual Skill ativar.
4. Evite prompts genéricos como "Como funciona?" ou "Pode me ajudar?".

### Template de Starter Prompts

```
Starter Prompt 1: [Ativa o Skill principal — fluxo mais comum]
Starter Prompt 2: [Ativa um Skill secundário específico]
Starter Prompt 3: [Começa sem contexto — testa o comportamento de boas-vindas]
Starter Prompt 4: [Caso de borda — contexto parcial ou pedido incomum]
```

---

## Checklist de Qualidade

Antes de finalizar qualquer instrução de Agent, verifique:

**Identidade e Arquitetura**
- [ ] Nome é específico e reconhecível sem contexto adicional
- [ ] Goal Description responde "quando usar este Agent vs. outro"
- [ ] Capability Boundary define explicitamente o que está fora do escopo
- [ ] Sub-Agent Map está completo com triggers, output contracts e handoffs

**Instrução (Quatro Pilares)**
- [ ] Regra de idioma presente: "Sempre responda em português brasileiro..."
- [ ] Persona define voz, expertise e comportamento de boas-vindas
- [ ] Task lista todos os Skills como subtarefas explícitas
- [ ] Context inclui o Sub-Agent Map completo como tabela de roteamento
- [ ] Context define comportamento para contexto ausente (máx. 3 perguntas)
- [ ] Format tem template de output para cada Skill individualmente
- [ ] Nenhum bloco de comentário `<!-- -->` no texto final — toda lógica em texto visível

**Facilitation Agents (se aplicável)**
- [ ] Regras do PDF Loop presentes no Context
- [ ] Turnos usam uma pergunta por vez
- [ ] Decision forks têm exatamente 3 opções, recomendada primeiro
- [ ] Encerramento define decisões, assumptions e próximos passos

**Starter Prompts**
- [ ] 4 Starter Prompts definidos
- [ ] Cada um ativa um fluxo ou Skill diferente
- [ ] Escritos em primeira pessoa do usuário
- [ ] Nenhum prompt genérico ou auto-referencial

---

## Mapeamento: product-manager-prompts → gemini-agents

Use esta tabela ao adaptar assets do repositório fonte para este formato.

| Elemento do PM Prompt | → Destino no Agent Builder |
|----------------------|---------------------------|
| AI role e voz | → **Persona** |
| Entregável principal | → **Task** (skill principal) |
| Regras de facilitação, PDF Loop | → **Context** (Facilitation rules) |
| Required Context Keys | → **Context** (Contexto ausente) |
| Missing Context Rule | → **Context** (máx. 3 perguntas) |
| Output Format com template | → **Format** (output do skill correspondente) |
| Final Step options | → **Format** (Encerramento padrão) |
| Múltiplas seções de output | → **Sub-Agent Map** (um skill por seção) |
| Attribution / Licensing | → Omitir do Agent; preservar no arquivo `.md` local |

---

## Decisão de Design: Sub-Agent Skills vs. Orquestrador Único

| Dimensão | Sub-Agent Skills | Orquestrador Único |
|----------|-----------------|-------------------|
| Previsibilidade de output | Alta — cada skill tem output contract fixo | Variável — depende do contexto acumulado |
| Facilidade de manutenção | Alta — edite um skill sem afetar outros | Baixa — mudanças cascateiam |
| Curva de aprendizado do usuário | Moderada — precisa entender quais skills existem | Baixa — um Agent faz tudo |
| Risco de deriva de escopo | Baixo — boundary explícita por skill | Alto — o orquestrador acumula comportamentos |
| Composabilidade | Alta — skills encadeiam naturalmente | Baixa — outputs raramente alimentam outros Agents |

**Como registrar observações:** Ao encerrar uma sessão de uso com um Agent construído neste modelo, adicione uma entrada em `## Post-Launch Notes` no arquivo `instructions.md` do Agent com:
- Data
- Qual skill foi mais ativado
- Onde o roteamento falhou (se falhou)
- Se o output contract foi respeitado
- Comparação subjetiva com a experiência de usar um orquestrador único equivalente
