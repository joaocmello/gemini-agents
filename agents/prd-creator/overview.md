# PRD Creator — Overview

## Etapa 1: Objetivo do Agent

**Problema que resolve:**
Product managers precisam criar PRDs a partir de documentos de discovery, pesquisas, briefings e comunicações — mas o fluxo tradicional exige que o PM preencha campo por campo em resposta a perguntas do agente. Isso inverte a carga de trabalho: o PM faz o trabalho que o agente deveria fazer.

**Proposta de valor:**
O PRD Creator lê qualquer documento ou contexto fornecido, infere o máximo possível, e entrega um rascunho inicial completo para o PM revisar — em vez de fazer perguntas antes de produzir qualquer coisa.

**Fluxo esperado:**
1. PM fornece um documento (oportunidades, discovery, briefing, PRD parcial, etc.)
2. Agent analisa, infere e produz um rascunho completo com marcadores de confiança
3. PM valida o rascunho em bloco
4. Agent refina seção por seção com perguntas pontuais (uma por vez)
5. Documento final entregue em Markdown

**Usuário-alvo:** Product Managers de times de produto digital, que entregam PRDs para times de engenharia e design iniciarem RFC, Design Docs e implementação.

---

## Etapa 2: Decomposição em Sub-Agent Skills

O agent opera com **três skills internas**, invisíveis ao usuário. O orquestrador (`instructions.md`) decide qual skill ativar com base no estado da sessão.

### Skill 1 — Document Analyzer
**Responsabilidade:** Ler o documento ou contexto fornecido e classificar cada campo do template PRD em três categorias:
- ✅ **Inferível com alta confiança** — extraído diretamente do material
- 🟡 **Inferível com suposição razoável** — deduzido a partir de padrões e contexto
- 🔴 **Ausente** — não há base no material; será perguntado depois

**Output:** Mapa interno de confiança por campo (não exibido ao usuário diretamente).

### Skill 2 — Draft Builder
**Responsabilidade:** Com base no mapa do Document Analyzer, preencher todas as seções do PRD. Seções ✅ são preenchidas normalmente. Seções 🟡 recebem o conteúdo inferido com o marcador explícito `🟡 [Suposição: ...]`. Seções 🔴 recebem `🔴 [A preencher]` e são listadas ao final para refinamento.

**Output:** PRD completo em Markdown, com marcadores visíveis para o PM.

### Skill 3 — Refinement Loop
**Responsabilidade:** Para cada campo marcado como 🟡 ou 🔴, fazer uma pergunta por vez. Atualizar o documento a cada resposta. Encerrar quando o PM aprovar ou sinalizar que quer parar.

**Output:** PRD revisado com os campos atualizados.

### Skill 4 — Analytics Event Planner
**Responsabilidade:** Gerar o plano completo de eventos de analytics para a feature, seguindo a taxonomia canônica da equipe. Percorre o fluxo do usuário descrito no PRD, identifica todos os pontos de rastreamento (impressões e interações), aplica as regras de seleção de evento corretas (`view_content`, `select_content`, `view_creative`, `select_creative`, `ab_[feature]`), e define as propriedades padronizadas e customizadas para cada evento.

**Ativação:** Sob demanda — acionado quando o PM escolhe a opção 1 do próximo passo ao final da sessão, envia documentação de eventos, ou solicita o plano de instrumentação explicitamente.

**Output:** Tabela de eventos organizada por etapa/tela, com propriedades padronizadas, marcadores de confiança (✅/🟡/🔴) e lista de itens para alinhamento com o time de analytics.

---

## Regra de Transição entre Skills

```
[PM envia documento ou contexto]
        ↓
[Skill 1: Document Analyzer] → mapa de confiança interno
        ↓
[Skill 2: Draft Builder] → PRD completo com marcadores
        ↓
[PM valida rascunho em bloco]
        ↓
[Skill 3: Refinement Loop] → uma pergunta por vez para 🟡 e 🔴
        ↓
[Orquestrador] → entrega final + 4 próximos passos
        ↓ (sob demanda — opção 1 do próximo passo)
[Skill 4: Analytics Event Planner] → tabela de eventos por etapa/tela
```

Se o PM não fornecer nenhum documento, o orquestrador faz uma única pergunta de intake antes de acionar a Skill 2 com mínimo contexto.

---

## Post-Launch Notes

| Data | Observação |
|------|------------|
| — | A ser preenchido após uso em produção |
