# PRD Creator

**Versão:** v2.0 — Modo Proativo
**Última atualização:** Mai/2026

Agent de criação de Product Requirement Documents que analisa documentos de discovery e entrega um rascunho completo antes de fazer qualquer pergunta.

---

## O que mudou na v2.0

A versão anterior fazia perguntas campo a campo antes de produzir qualquer documento. O PM fazia o trabalho que o agente deveria fazer.

A v2.0 inverte o fluxo:

| Antes (v1) | Agora (v2) |
|-----------|------------|
| Agent pergunta → PM responde → document cresce | PM entrega material → Agent analisa → rascunho completo |
| 13 perguntas antes de ver qualquer documento | Rascunho completo na primeira resposta |
| PM define cada campo | Agent infere, PM valida e corrige |
| Campos em branco se PM não souber | Campos preenchidos com suposições sinalizadas |

---

## Estrutura dos Arquivos

```
agents/prd-creator/
├── overview.md                    ← Objetivo e decomposição (AGENT-BUILDER etapas 1–2)
├── subagent-document-analyzer.md  ← Skill de leitura e classificação do material
├── subagent-draft-builder.md      ← Skill de montagem do rascunho completo
├── subagent-refinement-loop.md    ← Skill de refinamento seção por seção
├── instructions.md                ← Orquestrador principal (configure este no Gemini)
└── README.md                      ← Este arquivo
```

---

## Arquitetura de Skills

```
[PM envia documento]
        ↓
[Document Analyzer]     Classifica cada campo: ✅ inferido / 🟡 suposição / 🔴 ausente
        ↓
[Draft Builder]         Preenche todas as seções com marcadores de confiança
        ↓              Apresenta rascunho completo → PM valida em bloco
[Refinement Loop]       Uma pergunta por vez para 🟡 e 🔴
        ↓
[Orquestrador]          Documento final + 4 próximos passos
```

---

## Como Configurar no Gemini

### Opção A — Gem Único (recomendado)

1. Crie um novo Gem no Gemini com o nome **"PRD Creator"**
2. No campo **Instructions**, cole o conteúdo de `instructions.md`
3. Logo abaixo, adicione uma seção separada com os três sub-agents:

```
[CONTEÚDO DE instructions.md]

---

## Sub-Agent Skill: Document Analyzer
[CONTEÚDO DE subagent-document-analyzer.md]

---

## Sub-Agent Skill: Draft Builder
[CONTEÚDO DE subagent-draft-builder.md]

---

## Sub-Agent Skill: Refinement Loop
[CONTEÚDO DE subagent-refinement-loop.md]
```

4. Configure os **Starter Prompts** sugeridos:
   - `Tenho um documento de discovery ou oportunidades — quero transformar em PRD`
   - `Quero criar um PRD do zero para uma nova feature`
   - `Tenho um PRD incompleto — quero preencher as lacunas`

### Opção B — Gems Separados (para implementação via API)

Configure 4 Gems distintos:
- `prd-orchestrator` → `instructions.md`
- `prd-document-analyzer` → `subagent-document-analyzer.md`
- `prd-draft-builder` → `subagent-draft-builder.md`
- `prd-refinement-loop` → `subagent-refinement-loop.md`

Conecte via Gemini API passando o estado entre eles.

---

## Comportamento Esperado

### Sessão com documento (fluxo padrão)

**PM:** *(cola documento de oportunidades da página de produto)*

**Agent:**
> 📄 Rascunho inicial do PRD gerado a partir do material fornecido.
>
> Analisei o documento e preenchi o máximo possível...
>
> [PRD completo com marcadores ✅ 🟡 🔴]
>
> **O rascunho está no caminho certo?**

**PM:** Sim, pode refinar.

**Agent:**
> 📝 Métricas de Sucesso — campo 1 de 4 pendentes
>
> O documento menciona aumentar a taxa de conversão, mas não especifica uma meta. Qual é o número que o time está mirando para considerar essa feature um sucesso?
>
> 💡 Sugestão baseada no material: +5% na taxa de conversão em 90 dias 🟡

---

### Sessão sem documento (fallback)

**PM:** Quero criar um PRD do zero para uma nova feature

**Agent:**
> Qual é a feature e o problema principal que ela resolve? Me dê o máximo de contexto que tiver e eu monto o primeiro rascunho.

**PM:** *(descreve a feature)*

**Agent:**
> 📄 Rascunho inicial do PRD...
> [segue o fluxo padrão]

---

## Marcadores de Confiança

| Marcador | Significado | Ação do PM |
|----------|-------------|------------|
| ✅ (sem marcador) | Extraído diretamente do material | Revisar se necessário |
| 🟡 Suposição: [...] | Inferido com raciocínio documentado | Confirmar ou corrigir |
| 🔴 [A preencher] | Ausente no material | Fornecer no refinamento |
