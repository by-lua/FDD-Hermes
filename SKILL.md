---
name: fdd
description: >-
  Feature-Driven Development (FDD) para Hermes, com motor adaptativo
  inspirado no L-Spec PI: Discovery com classificação automática →
  [Discuss*] → Research → Specify → [Clarify*] → [Design*] → Tasks → Execute
  (com Gate dentro) → State. Comandos via /xfdd.
  Compliance Gate BLOQUEANTE antes de qualquer edit.
  Autosave OBRIGATÓRIO entre fases.
trigger: xfdd,reverse,novo projeto,feature,bug,levantamento,monetização,produto,pricing,preço,plano,assinatura
metadata:
  author: Lua (Hermes Agent)
  version: 2.6.0
  based-on: l-spec PI v2 — sincronizado Hermes (agent-lsp, pipeline table, NUNCA rules, /xfdd commands, Compliance Gate bloqueante, Autosave obrigatório entre fases, Artifact Enforcement)
  v2.6-changes:
    - "2026-06-05 — Research obrigatório (RPIV-PI insight): análise do codebase antes de Specify. Impede 'inventar' soluções que não existem."
---

# FDD — Feature-Driven Development (Hermes)

> Spec-first, rastreável, validado. Nada fora da spec. Comandos FDD preservados.

## Objetivo desta versão

Esta versão adapta o funcionamento do **L-Spec PI adaptivo** para o Hermes,
com **uma única simplificação intencional**:

- **Discovery de projeto novo reduzido** (mais curto)

Todo o resto segue a mesma disciplina de execução:

- Discovery adaptativo por tipo
- Clarify/Sanity quando houver ambiguidades
- Spec testável
- Design quando necessário
- Tasks rastreáveis
- Execute com **Gate de validação obrigatório**
- **Compliance Gate BLOQUEANTE antes de qualquer edit**
- **Autosave OBRIGATÓRIO entre fases**
- State atualizado ao final

---

## Comandos (via /xfdd)

- `/xfdd new` → Projeto novo
- `/xfdd feature` → Nova feature em projeto existente
- `/xfdd reverse` → Projeto existente sem specs (survey)
- `reverse` (texto solto) → atalho para fluxo reverse

> Tudo via `/xfdd`. Comando antigo `/fdd` também funciona via alias.

---

## Pipeline Adaptativo

```
Discovery → [Discuss*] → Research → Specify → [Clarify*] → [Design*] → Tasks → Execute
```

```
┌────────────┬──────────────────────────────────────────────────────────┐
│ FASE       │ QUANDO RODA                                             │
├────────────┼──────────────────────────────────────────────────────────┤
│ Discovery  │ SEMPRE                                                  │
│ Research   │ SEMPRE (OBRIGATÓRIO) — análise do codebase              │
│ Discuss    │ OPCIONAL — só se área cinzenta/ambígua                  │
│ Specify    │ SEMPRE (OBRIGATÓRIO)                                    │
│ Clarify    │ OPCIONAL — só se ambiguidade nos requisitos             │
│ Design     │ OPCIONAL — só se necessidade arquitetural              │
│ Tasks      │ SEMPRE                                                  │
│ Execute    │ SEMPRE                                                  │
└────────────┴──────────────────────────────────────────────────────────┘
```

**NUNCA:**
- Pular fases obrigatórias (especialmente Research!)
- Quick mode
- Auto-sizing
- Implementar sem analisar codebase primeiro

**SEMPRE:**
- Reler spec antes de implementar
- Autosave de estado em cada fase
- Gate antes de encerrar Execute
- **Passar no Compliance Gate antes de QUALQUER edit**

### Regra crítica

**Validate não é fase separada.**
A validação acontece no **Execute**, no passo de **GATE CHECK**.

### ⚠️ REGRA CRÍTICA — ARTIFACT ENFORCEMENT (RPIV-PI insight)

**"The artifact one writes is the next one's input."**

Inspirado no RPIV-PI pipeline — cada fase **PRODUZ um artifact** que a próxima **PRECISA**. Sem artifact, não avanza. Ver [references/rpiv-pi-artifact-enforcement.md](references/rpiv-pi-artifact-enforcement.md).

```
╔══════════════════════════════════════════════════════════════════════╗
║  GATE: ARTIFACT CHECK — antes de iniciar próxima fase               ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  Pipeline → Produz → Feeds → Próxima                               ║
║                                                                      ║
║  Discovery  → .fdd/state.md           → Research                   ║
║  Research   → features/[name]/research.md → Specify                ║
║  Specify    → features/[name]/spec.md  → Tasks                     ║
║  Tasks      → features/[name]/tasks.md → Execute                  ║
║  Execute    → working-tree changes     → Validate                   ║
║                                                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║  ⚠️  Se artifact da fase anterior NÃO EXISTE → BLOQUEIA.            ║
║  ⚠️  Não pergunta. Não pula. Não implementa.                        ║
║  ⚠️  Cria o artifact primeiro, depois avanza.                       ║
╚══════════════════════════════════════════════════════════════════════╝
```

**Exemplo de Bloqueio:**

```
[ENFORCED] ✗ Execute BLOQUEADO

Artifact necessário: features/minha-feature/tasks.md (de Tasks)
Artifact encontrado: ❌ NÃO EXISTE

⚠️  Execute não pode rodar sem tasks.md.
⚠️  Execute /xfdd tasks primeiro para criar o artifact.
```

```
[ENFORCED] ✗ Tasks BLOQUEADO

Artifact necessário: features/minha-feature/spec.md (de Specify)
Artifact encontrado: ❌ NÃO EXISTE

⚠️  Tasks não pode rodar sem spec.md.
⚠️  Execute /xfdd specify primeiro para criar o artifact.
```

---

## Hermes Tools — Code Navigation

**Code Navigation — NUNCA use bash grep/find para navegação de código**

O Hermes já tem `agent-lsp` configurado como MCP server (66 tools LSP). Use SOMENTE:

**Estrutura do repo:**
- `mcp_agent_lsp_list_symbols` → symbols de um arquivo (classes, funções, métodos)
- `mcp_agent_lsp_find_symbol` → encontrar símbolo pelo nome

**Navegação contextual:**
- `mcp_agent_lsp_go_to_definition` → ir para definição de símbolo
- `mcp_agent_lsp_find_references` → todas as referências a um símbolo
- `mcp_agent_lsp_find_callers` → o que chama esta função
- `mcp_agent_lsp_inspect_symbol` → tipo, docs, signature de um símbolo

**Impacto antes de editar:**
- `mcp_agent_lsp_blast_radius` → impacto a montante (upstream callers)
- `mcp_agent_lsp_get_diagnostics` → erros/warnings do arquivo

**Se agent-lsp não disponível ou linguagem não suportada → usa tools nativas:**
- `read_file`, `search_files`, `terminal` com ls/cd
- NÃO BLOQUEIA — continua o fluxo normalmente

**Bash grep/find SOLO quando:**
- Precisa de output pipeado para outro comando shell
- Casos pontuais que LSP não cobre

---

## ⚠️ CRÍTICO — Discovery é PERGUNTAS PARA O USUÁRIO

**Discovery NÃO é eu escrevendo respostas sozinho.**

O agente faz PERGUNTAS → usuário responde → agente salva em state.md → avança para próxima fase.

**NUNCA:**
- ❌ Inventar respostas de Discovery sem perguntar ao usuário
- ❌ Pular para implementação durante Discovery
- ❌ Despejar todas perguntas de uma vez (uma fase por vez, confirmar antes de continuar)

**SEMPRE:**
- ✓ Perguntar uma fase, esperar confirmação, continuar
- ✓ Salvar resposta do usuário no state.md
- ✓ Avançar para Research após confirmar Discovery completo

**Exemplo de erro:**
```
❌ "O projeto é um sistema de checkout..." (eu inventei)
✓ "Qual o objetivo do projeto?" (perguntei ao usuário)
```

---

## Discovery adaptativo

### Passo 0 — Classificar tipo (IMEDIATO ao carregar skill)

Ao carregar a skill, classifique o que a Lua descreveu:

| Palavra-chave na mensagem | Tipo detectado | Fluxo |
|---------------------------|---------------|-------|
| "bug", "erro", "não funciona", "falha", "crash" | **Bug** | Fluxo 4 |
| "novo projeto", "criar", "começar" | **Projeto novo** | Fluxo 1 |
| "reverse", "survey", "sem spec", "mapear" | **Reverse** | Fluxo 2 |
| "feature", "melhorar", "adicionar", "nova função" | **Feature** | Fluxo 3 |
| "pequeno", "ajuste", "mudar isso", "corrigir só" | **Mudança pequena** | Fluxo 5 |
| Nenhuma acima | **Perguntar** | "Você quer criar projeto,feature,fix bug ou reverter projeto existente?" |

**Se projeto existe** → detectar `.fdd/state.md` e usar como base antes de perguntar.

### Fluxos por tipo

#### 1) Projeto novo (`/xfdd new`) — 6 perguntas

Perguntas obrigatórias (reduzidas):

1. **Objetivo** — o que o projeto entrega? (1 frase)
2. **Público/problema** — para quem e qual dor resolve?
3. **MVP** — mínimo que precisa funcionar para lançar
4. **Stack/restrições** — linguagem/framework/banco e limites técnicos
5. **Deploy alvo** — onde vai rodar (Coolify, VPS, etc.)
6. **Fora de escopo agora**

> Após essas 6, segue direto para Specify.

#### 2) Projeto existente sem `.fdd/` (Reverse primeiro)

- Executar reverse para levantar estrutura e features reais
- Gerar base `.fdd/` necessária para continuar o pipeline
- Só depois discovery reduzido da demanda atual

#### 3) Nova feature em projeto existente (`/xfdd feature`)

- O que muda (1 frase)
- Onde toca (arquivos/módulos)
- Regra principal esperada
- Impacto/risco conhecido
- Fora de escopo

#### 4) Bug

- Comportamento atual
- Comportamento esperado
- Como reproduzir
- Área provável do código
- Regressão (sim/não)
- Fora de escopo do fix

#### 5) Mudança pequena

Discovery ultra curto, sem pular pipeline:

- O que ajustar
- Onde ajustar
- Como validar que ficou certo

---

## Sanity / Clarify (gate de ambiguidade)

Se houver dúvida de escopo, regra, prioridade, trade-off ou comportamento:

- **Parar e perguntar**
- Registrar decisão antes de escrever/alterar código

Sem clareza, não avança.

---

## Specify (obrigatório)

Criar `features/<feature>/spec.md` com:

- Objetivo
- Requisitos funcionais numerados e testáveis
- Fora de escopo
- Critérios de aceite

Formato de requisito:

- `WHEN [ação/evento] THEN sistema DEVE [resposta]`

### Rastreio obrigatório

Todo pedido explícito da Lua deve virar item rastreável (`R1`, `R2`, `R3...`) e
ser mapeado para requisito(s) da spec.

---

## Design (quando necessário)

Obrigatório quando:

- múltiplos módulos
- impacto arquitetural
- integração externa
- risco de regressão relevante

Saída esperada: decisões técnicas, componentes afetados, estratégia de erro,
impacto em dados/contratos.

---

## Tasks (obrigatório)

Criar `features/<feature>/tasks.md` com tarefas atômicas:

- o que fazer
- onde fazer
- dependências
- critério de pronto
- teste associado

**Tools por tarefa:**
- MCP: `agent-lsp` (code navigation: go_to_definition, find_references, list_symbols, find_callers, blast_radius, get_diagnostics)
- NUNCA use bash grep/find para navegação de código
- Se agent-lsp não disponível → usa tools nativas (read_file, search_files), NÃO BLOQUEIA

Se escopo mudar no meio, pausar e **replanejar tasks**.

---

## Execute (obrigatório, com Gate)

### COMPLIANCE GATE — ANTES DE QUALQUER EDIT

**Este gate é BLOQUEANTE.** Roda **antes de cada tarefa** do ciclo de implementação. Não é uma vez só no início do Execute — é por tarefa.

```
╔══════════════════════════════════════════════════════════════════╗
║  GATE: COMPLIANCE CHECK — antes de cada tarefa                 ║
╠══════════════════════════════════════════════════════════════════╣
║  □  features/<feature>/spec.md existe                     ║
║  □  spec.md foi lida e compreendida ("Contexto lido")         ║
║  □  Arquivos a editar estão listados na spec ou tasks          ║
║  □  Mudança proposta NÃO foge do escopo da spec                ║
║  □  .fdd/state.md foi atualizado na última fase                ║
╠══════════════════════════════════════════════════════════════════╣
║ ⚠️  Se ANY □ = false → BLOQUEIA. Não edita. Pergunta.        ║
╚══════════════════════════════════════════════════════════════════╝
```

**Verificar cada item explicitamente.** Não presumir. Ler os arquivos, conferir.

**Se qualquer □ = false:**
- Parar imediatamente
- Reportar qual item falhou
- Perguntar à Lua o que fazer antes de prosseguir
- **Não fazer nenhum edit até resolver**

**Se todos □ = true:**
- Prosseguir com ciclo de implementação

---

**Proibidos:**
- ❌ Quick mode — implementar completo, sem atalhos
- ❌ Auto-sizing — não redimensionar sem aprovação
- ❌ Pular tarefas — implementar todas listadas
- ❌ Sair sem evidência de teste rodado
- ❌ Editar sem passar no Compliance Gate
- ❌ Não salvar `.fdd/state.md` — autosave é obrigatório, não opcional

**Regra operacional obrigatória:** durante o Execute, usar o **máximo de subagentes possível** (dentro dos limites da plataforma) para implementar, revisar e validar em paralelo. Só executar de forma sequencial quando houver dependência estrita entre etapas.

Ciclo por tarefa:

0. **COMPLIANCE GATE** — verificar checklist (spec existe, lida, arquivos listados, dentro do escopo, state atualizado). Se □ = false → para e pergunta.
1. **Plan** da tarefa
2. **RED** — teste falhando
3. **GREEN** — implementação mínima
4. **GATE CHECK** — executar validações
5. **Review** — checar aderência à spec
6. **AUTOSAVE** — salvar `.fdd/state.md` com T[X] completo (OBRIGATÓRIO)
7. próxima tarefa

### Gate mínimo para concluir

Não pode encerrar sem evidência de:

- **Compliance Gate verificado** (todos os 5 □ = true) — **por cada tarefa**
- **Autosave feito** (`.fdd/state.md` atualizado com T[X] completo)
- testes executados (comando + resultado)
- validação funcional da regra principal
- arquivos alterados
- confirmação de que não ficou item pedido pendente sem justificativa

---

## State (sempre atualizar) — OBRIGATÓRIO

**State saving NÃO é opcional.** É parte do pipeline, não uma nota pessoal.

### GATE: Estado Salvo Entre Fases

**Antes de iniciar qualquer fase nova, verificar se a fase anterior salvou `.fdd/state.md`:**

```
╔══════════════════════════════════════════════════════════════════╗
║  GATE: STATE SAVED — antes de iniciar nova fase                 ║
╠══════════════════════════════════════════════════════════════════╣
║  □  .fdd/state.md existe                                        ║
║  □  Última fase registrada (ex: "Tarefas T1-T3 completas")     ║
║  □  Pendências atualizadas                                      ║
║  □  Commits feitos (se aplicável)                              ║
╠══════════════════════════════════════════════════════════════════╣
║ ⚠️  Se ANY □ = false → SALVA ANTES de iniciar nova fase.       ║
╚══════════════════════════════════════════════════════════════════╝
```

**Se qualquer □ = false:**
- Parar
- Salvar `.fdd/state.md` com informações da fase atual
- Só então prosseguir para próxima fase

---

### Passo Explícito de Save (em cada fase)

**Ao final de cada fase, seguir este checklist:**

```
[Estado da Fase: <nome>]
□ .fdd/state.md atualizado com:
  - Fase atual e status
  - Tarefas completadas
  - Pendências
  - Arquivos alterados
  - Commits (se houver)
□ Próxima fase definida
→ Fase seguinte pode começar
```

---

### Conteúdo do state.md

Atualizar `.fdd/state.md` com:

- fase atual/status
- resumo objetivo
- arquivos alterados
- testes executados e resultado
- pendências e próximos passos

**State é fonte de continuidade entre sessões.**

---

## Research — Análise do Codebase (NÃO perguntas ao usuário)

**Research é OBRIGATÓRIO. Não pula. Não pergunta usuário.**

Research analiza o **código existente**, não faz perguntas para o usuário.

| SEM Research | COM Research |
|--------------|--------------|
| "Invventa" soluções que não existem | Vê como código similar foi feito |
| Spec pede algo impossível no codebase | Spec sabe o que é possível |
| Ignora patterns e convenções | Respeita arquitetura existente |
| Código conflita com código antigo | Integra corretamente |

**Research pergunta ao CÓDIGO, não ao usuário:**
- "Onde está implementada essa feature?"
- "Quem chama essa função?"
- "Qual pattern esse código usa?"

**Output:** `features/[name]/research.md`

**Sem artifact = BLOQUEIA Specify.**

---

## Reverse (`/xfdd reverse`)

Para projeto existente sem specs:

1. verificar `.fdd/state.md` (se existir, usar como base)
2. scan de estrutura
3. análise de módulos/rotas/models
4. agrupamento por feature
5. geração de specs por feature
6. mapa geral
7. state atualizado

Se houver models ORM, validar consistência com schema real quando aplicável.

---

## Regras fixas

1. Nada fora da spec
2. Sempre reler spec antes de implementar
3. **Preflight obrigatório de contexto antes de codar:** ler a pasta `.fdd/` inteira (no mínimo `state.md`, `map.md` se existir, specs e tasks da feature-alvo) e responder com "Contexto lido" + lista dos arquivos lidos + resumo em 3 bullets do entendimento. Sem isso, não implementar. **Isto alimenta o item □ "Contexto lido" do Compliance Gate.**
4. **Compliance Gate é BLOQUEANTE — ANTES de qualquer edit**, verificar checklist (spec existe, lida, arquivos listados, dentro do escopo, state atualizado). Se qualquer □ = false → para e pergunta.
5. **A pasta `.fdd/` deve ser versionada no Git e subir para o repositório remoto.** Não ignorar `.fdd/` no `.gitignore`.
6. **Nunca versionar nem subir `.env` (ou qualquer arquivo de segredo).** Garantir `.env*` no `.gitignore` (mantendo apenas exemplos como `.env.example`).
7. **Usar o máximo de subagentes possível em todo o fluxo FDD** (dentro dos limites da plataforma), priorizando paralelismo; só executar sequencial quando houver dependência estrita.
8. Sem pular fases obrigatórias
9. Validate é dentro do Execute (Gate)
10. Toda solicitação explícita da Lua entra no rastreio (`R#`)
11. Se escopo mudar, replanejar antes de continuar
12. State sempre atualizado
13. Se não há evidência de teste/validação, não está concluído
14. **NUNCA editar arquivo sem ter passado no Compliance Gate**

---

## Formato de resposta operacional

Ao final de cada bloco de trabalho, responder **curto**:

```
[Discovery | Specify | Tasks | Execute] ✓
Feito: ...
Próximo: ...
```

Se bloqueado ou pendente:

```
[Bloqueado] — razão
Aguardando: ...
```

**Compliance Gate — formato de resposta (por tarefa):**

```
[GATE: Compliance Check] ✓
□ spec.md existe
□ Contexto lido
□ Arquivos listados
□ Dentro do escopo
□ State atualizado
→ Prosseguindo para ciclo de implementação
```

Se qualquer □ = false:

```
[GATE: Compliance Check] ✗ BLOQUEADO
□ [falhou] — especifique o item
□ [falhou] — especifique o item
Aguardando: decisão da Lua antes de editar
```

**Não** escrever parágrafos longos. Bullet points curtos. Resposta máxima 5-6 linhas.

---

## Enforcement Pattern (CRÍTICO)

**O fluxo só funciona se AMBOS os gates estão ativos:**

1. **Compliance Gate** — antes de cada edit (verifica spec, contexto, escopo)
2. **State Saving Gate** — entre fases (verifica state atualizado)

**Um sem o outro não funciona:**
- Só Compliance Gate → agente pode "passar" mas state fica pendente
- Só State Saving → agente pode salvar mas editar fora da spec

**Teste de funcionamento:** o cenário Bug neste session provou: T-ID-5 foi bloqueado porque state não foi atualizado após T-ID-4. O Compliance Gate chamou atenção, mas o root cause era o State Saving Gate.

**Resumo:** Ambos são obrigatórios. Compliance Gate sem State Saving = state fica para trás. State Saving sem Compliance Gate = edits fora da spec.

---

## Pitfalls críticos

- **Discovery: NÃO inventar respostas** — fazer perguntas ao usuário, esperar resposta, salvar
- **Research: NÃO perguntar ao usuário** — analisar o código existente
- Pular spec por "mudança pequena"
- Encerrar sem teste rodado
- Não atualizar state
- Derivar escopo quando a Lua pedir ajuste novo
- Presumir requisito sem confirmação em caso ambíguo
- **EDITAR sem passar no Compliance Gate — PARAR e verificar checklist primeiro**
- **Fingir que passou no gate sem verificar os 5 itens**
- **EDITAR NO DIRETÓRIO ERRADO — editar SEMPRE no clone git (ex: ~/.pi/agent/git/github.com/by-lua/<REPO>/), nunca em ~/.hermes/profiles/luna/skills/ que não é git e não vai pro remote**
- **Compliance Gate sem State Saving — state fica pendente entre fases**
- **NÃO existe pasta `fixes/`** — bugs e ajustes usam prefixo `fix-`/`bug-` em `features/`
- **Feature existente = atualizar docs e continuar** — não reiniciar pipeline

---

## Referências de operação

- `references/skill-rollout-global-perfis.md` — checklist para sincronizar atualização da FDD entre global e perfis e validar consistência.
- `references/platform-tooling-mapping.md` — mapeamento PI vs Hermes para tools de navegação de código (pi-cymbal vs agent-lsp). Sempre consultar antes de instruir tools em skills.
- `references/compliance-gate-pattern.md` — **PADRÃO OBRIGATÓRIO**: Compliance Gate bloqueante inspirado em opensquad/aiox-core/agentic-os. Verificar antes de toda implementação.

## Compatibilidade e transição

- Comandos via `/xfdd` (aliases `/fdd` também funcionam)
- Fluxo interno segue disciplina L-Spec PI adaptiva
- Discovery de projeto novo: 6 perguntas (enxuto)
- Hermes tools: `agent-lsp` (não pi-cymbal)
- Compliance Gate: bloqueante antes de cada tarefa (5 itens)

---

## Git Workflow — Onde editar

**⚠️ CRÍTICO: Editar NO clone git, não no skills local.**

O diretório `~/.hermes/profiles/luna/skills/` **NÃO é git**. Patches ali não sobem pro remote.

Estrutura correta:

```
~/.pi/agent/git/github.com/by-lua/<REPO>/   ← CLONE GIT (edit here)
  └── .git/
  └── SKILL.md
  └── skills/

~/.hermes/profiles/luna/skills/              ← LOCAL ONLY (don't edit)
  └── software-development/fdd/SKILL.md
```

**Ao trabalhar em projetos FDD:**
1. Clone repo se não existir: `git clone https://github.com/by-lua/<REPO>.git`
2. Editar NO clone (diretório com `.git/`)
3. Commitar no clone
4. Push via signet secret exec

## Propagação de Skill — Após atualizar master (VPS)

**Quando a skill FDD for atualizada (nova versão no repo `by-lua/FDD-Hermes`), propagar para TODOS os perfis:**

```
Perfis com FDD (verificar todos):
- /root/.hermes/skills/software-development/fdd/          (global)
- /root/.hermes/profiles/luna/skills/software-development/fdd/
- /root/.hermes/profiles/kira/skills/software-development/fdd/
- /root/.hermes/profiles/kira2/skills/software-development/fdd/
```

**Após atualizar o repo Git:**
```bash
# Copiar do clone git para todos os perfis
cp /root/.hermes/profiles/luna/home/.pi/agent/git/github.com/by-lua/FDD-Hermes/SKILL.md \
   /root/.hermes/skills/software-development/fdd/SKILL.md

cp /root/.hermes/profiles/luna/home/.pi/agent/git/github.com/by-lua/FDD-Hermes/SKILL.md \
   /root/.hermes/profiles/kira/skills/software-development/fdd/SKILL.md

cp /root/.hermes/profiles/luna/home/.pi/agent/git/github.com/by-lua/FDD-Hermes/SKILL.md \
   /root/.hermes/profiles/kira2/skills/software-development/fdd/SKILL.md
```

**Verificar versões:**
```bash
grep -m1 "version:" /root/.hermes/skills/software-development/fdd/SKILL.md
grep -m1 "version:" /root/.hermes/profiles/luna/skills/software-development/fdd/SKILL.md
grep -m1 "version:" /root/.hermes/profiles/kira/skills/software-development/fdd/SKILL.md
grep -m1 "version:" /root/.hermes/profiles/kira2/skills/software-development/fdd/SKILL.md
```

**Regra:** após qualquer update na FDD, verificar e propagar para todos os perfis antes de considerar a tarefa concluída.
