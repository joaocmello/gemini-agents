# Catálogo de Agents

Este diretório contém todos os Agents construídos com o processo definido em [`AGENT-BUILDER.md`](../AGENT-BUILDER.md).

---

## Como está organizado

Cada Agent tem sua própria pasta com um arquivo `instructions.md`:

```
agents/
└── [nome-do-agent]/
    └── instructions.md
```

O `instructions.md` contém:
- Identidade e propósito do Agent
- Sub-Agent Map completo
- Instrução com os quatro pilares (Persona · Task · Context · Format)
- Starter Prompts
- Post-Launch Notes (atualizado após uso real)

---

## Agents disponíveis

| Agent | Tipo | Skills | Status |
|-------|------|--------|--------|
| *(nenhum ainda)* | — | — | — |

> Adicione uma linha nesta tabela ao criar cada novo Agent.
> **Tipo:** `Execution` (output direto) ou `Facilitation` (PDF Loop).

---

## Criando um novo Agent

1. Leia o [`AGENT-BUILDER.md`](../AGENT-BUILDER.md) integralmente.
2. Crie uma pasta com o nome do Agent em kebab-case: `nome-do-agent/`
3. Crie o arquivo `instructions.md` dentro da pasta seguindo o template do AGENT-BUILDER.
4. Registre o Agent na tabela acima.
5. Configure o Agent no Gemini colando o conteúdo do `instructions.md`.
