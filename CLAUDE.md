# Claude Code Instructions — gemini-agents

## Repositories

- **gemini-agents** (`c:\Users\joaom\OneDrive\Documents\Code\gemini-agents`) — active repo where agents are built and committed
- **product-manager-prompts** (`c:\Users\joaom\OneDrive\Documents\Code\product-manager-prompts`) — read-only source of PM frameworks, prompts, and facilitation patterns to draw content from when building agents

---

## Agent Creation & Modification Workflow

Follow these 5 steps every time the user asks to create or change an agent.

### Step 1 — Understand the request
- Identify the agent's purpose, audience, and expected behavior
- Read `AGENT-BUILDER.md` in this repo for the canonical 4-stage build cycle: **Objetivo → Decomposição → Sub-Agents → Orquestrador**

### Step 2 — Source content from product-manager-prompts
- Search the `product-manager-prompts` repo for prompts, frameworks, or facilitation patterns relevant to the agent's task
- Map found content to the 4-pillar model (Persona · Task · Context · Format)
- Note which source files were consulted so the agent's README can reference them

### Step 3 — Design the agent architecture
Apply the 4-pillar model to every instruction file:
- **Persona** — role, expertise, and behavioral traits
- **Task** — what the agent does, step by step
- **Context** — rules, constraints, language, orchestration logic
- **Format** — output structure, templates, transition signals

File and folder conventions:
- Agent folder: `agents/{kebab-case-name}/`
- Required files: `orchestrator.md`, `sub-agent-{name}.md` (one per sub-agent), `README.md`
- Language: **Portuguese (Brazilian)** unless the user explicitly requests otherwise
- The orchestrator must never expose the internal sub-agent architecture to the end user

### Step 4 — Validate with the user
- Present all draft files for review before writing them to disk, or write them and clearly indicate they are drafts
- Iterate on feedback; do not commit until the user signals approval

### Step 5 — Commit and push
- Stage only the files inside `agents/{agent-name}/`
- Commit message format: `Add {agent-name} agent — {one-line description}` or `Update {agent-name} agent — {one-line description}`
- **Always confirm with the user before pushing** to the remote

---

## Quality Checklist

Before marking an agent as done, verify:

- [ ] Each sub-agent has a single, clearly bounded responsibility
- [ ] Output contracts between agents are explicitly defined in the orchestrator
- [ ] All instruction files follow the Persona · Task · Context · Format structure
- [ ] Language is consistently Portuguese (Brazilian) throughout
- [ ] README.md covers: architecture diagram, configuration options (single Gem vs. separate Gems), and an example session flow
- [ ] Orchestrator hides the internal sub-agent architecture from the user
- [ ] Starter prompts (4 clicker examples) are defined for the orchestrator
- [ ] The agent has been reviewed against the full checklist in `AGENT-BUILDER.md`

---

## Reference

- Canonical process: `AGENT-BUILDER.md`
- Existing agent example: `agents/prd-creator/`
- Agent catalog: `agents/README`
