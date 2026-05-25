# PRD Creator — Multi-Step Agent

Sistema multi-agente para criação de Product Requirement Documents (PRDs) no Gemini. Desenvolvido para times de produto brasileiros com foco na entrega de documentação pronta para engenharia e design.

---

## Estrutura dos Agentes

```
gemini-agents/prd-creator/
├── orchestrator.md          ← Gem principal (configure este no Gemini)
├── sub-agent-intake.md      ← Sub-agente de descoberta
├── sub-agent-prd-writer.md  ← Sub-agente de escrita e montagem
└── README.md                ← Este arquivo
```

---

## Arquitetura

```
[PM] ←→ [Orchestrator]
              ↓
    [Intake Agent]          Fase 1: Descoberta
    Faz perguntas uma       (13 campos, um por vez)
    por vez, exibe PRD      Aceita "não sei" / "pular"
    parcial a cada turn     
              ↓
    [PRD Writer Agent]      Fase 2: Escrita
    Recebe contexto,        Refina linguagem, preenche
    monta PRD completo,     suposições, formata eventos
    aceita revisões         
              ↓
    [Orchestrator]          Fase 3: Revisão e Entrega
    Revisão final,          Confirma com PM, oferece
    4 opções de próx passo  4 opções de próximo passo
```

---

## Como Configurar no Gemini

### Opção A — Gem Único (recomendado para começar)

Configure um único Gem com o conteúdo do `orchestrator.md`. O Gemini simula os sub-agentes internamente dentro da mesma sessão.

**Nome do Gem:** `PRD Creator`

**Instruções:** Cole o conteúdo completo de `orchestrator.md`, seguido pelos prompts de sistema dos dois sub-agentes, separados por seções:

```
[CONTEÚDO DE orchestrator.md]

---

## Sub-Agente: Intake Agent
[CONTEÚDO DE sub-agent-intake.md]

---

## Sub-Agente: PRD Writer Agent  
[CONTEÚDO DE sub-agent-prd-writer.md]
```

### Opção B — Gems Separados (para uso com Gemini API + orquestração programática)

Configure três Gems distintos e conecte-os via API usando o Orchestrator como controlador:

1. `prd-orchestrator` — Gem de controle de fluxo
2. `prd-intake` — Gem de descoberta  
3. `prd-writer` — Gem de escrita

---

## Comportamento Esperado

### Sessão típica:

**Turno 1 — PM:**
> "Quero criar um PRD para uma feature de wishlist no app."

**Turno 1 — Agent:**
> 📋 **Fase 1 — Descoberta**  
> Ótimo! Vou te ajudar a construir o PRD da feature de Wishlist.  
>  
> Qual é o problema principal que essa feature resolve, e quem é o usuário afetado?

**Turno 2 — PM:**  
> "Usuários perdem produtos que querem comprar depois. São clientes recorrentes do app."

**Turno 2 — Agent:**
> [Exibe PRD parcial com contexto preenchido]  
> 📊 Progresso: 1/13 campos capturados  
>  
> Quais são os objetivos desta iniciativa para o negócio e para o usuário?

*[...continua por até 13 turnos de intake...]*

**Turno 14 — Agent:**
> ✅ Intake concluído — 11/13 campos capturados  
> [Exibe PRD completo com todos os campos]  
>  
> 📋 **Fase 2 — Escrita e Refinamento**  
> [PRD Writer refina e completa o documento]

**Turno final — Agent:**
> ✅ PRD finalizado — pronto para entrega  
> [Documento final completo]  
>  
> Qual o próximo passo?  
> 1. Refinar métricas e adicionar OKRs *(Recomendado)*  
> 2. Detalhar critérios de aceite em Gherkin  
> 3. Gerar briefing executivo para stakeholders  
> 4. Exportar como arquivo Markdown

---

## Referências

- Prompt base: adaptado do PRD Gem prompt (documento de texto em anexo)
- Exemplo de PRD: [POC] Review Social na PDP de Centauro (PDF em anexo)
- Framework: Persona · Task · Context · Format (Google Gemini Gems)
- Metodologia: PDF Loop do repositório `product-manager-prompts`

---

## Idioma

Todos os agentes respondem **exclusivamente em português brasileiro**, independentemente do idioma da entrada do usuário.
