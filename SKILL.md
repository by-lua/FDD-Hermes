---
name: fdd
description: >-
  Feature-Driven Development (FDD) para Hermes, com motor adaptativo
  inspirado no L-Spec PI: Discovery com classificação automática →
  [Discuss*] → Specify → [Clarify*] → [Design*] → Tasks → Execute
  (com Gate dentro) → State. Comandos via /xfdd.
  Compliance Gate BLOQUEANTE antes de qualquer edit.
trigger: xfdd,reverse,novo projeto,feature,bug,levantamento,monetização,produto,pricing,preço,plano,assinatura
metadata:
  author: Lua (Hermes Agent)
  version: 2.3.0
  based-on: l-spec PI v2 — sincronizado Hermes (agent-lsp, pipeline table, NUNCA rules, /xfdd commands, Compliance Gate bloqueante)
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
- State atualizado ao final

---

## Comandos (via /xfdd)

- `/xfdd new` → Projeto novo
- `/xfdd feature` → Nova feature em projeto existente
- `/xfdd reverse` → Projeto existente sem specs (survey)
- `reverse` (texto solto) → atalho para fluxo reverse

> Tudo via `/xfdd`. Comando antigo `/fdd` também funciona via alias.

---

## Pipeline canônico (igual ao L-Spec PI, adaptado)

```
Discovery → [Discuss*] → Specify → [Clarify*] → [Design*] → Tasks → Execute
```

```
┌────────────┬──────────────────────────────────────────────────────────┐
│ FASE       │ QUANDO RODA                                             │
├────────────┼──────────────────────────────────────────────────────────┤
│ Discovery  │ SEMPRE                                                  │
│ Discuss    │ OPCIONAL — só se área cinzenta/ambígua                  │
│ Specify    │ SEMPRE (OBRIGATÓRIO)                                    │
│ Clarify    │ OPCIONAL — só se ambiguidade nos requisitos             │
│ Design     │ OPCIONAL — só se necessidade arquitetural              │
│ Tasks      │ SEMPRE                                                  │
│ Execute    │ SEMPRE                                                  │
└────────────┴──────────────────────────────────────────────────────────┘
```

**NUNCA:**
- Pular fases obrigatórias
- Quick mode
- Auto-sizing

**SEMPRE:**
- Reler spec antes de implementar
- Autosave de estado em cada fase
- Gate antes de encerrar Execute
- **Passar no Compliance Gate antes de QUALQUER edit**

### Regra crítica

**Validate não é fase separada.**
A validação acontece no **Execute**, no passo de **GATE CHECK**.

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

Criar `.fdd/features/<feature>/spec.md` com:

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

Criar `.fdd/features/<feature>/tasks.md` com tarefas atômicas:

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
║  □  .fdd/features/<feature>/spec.md existe                     ║
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

**Regra operacional obrigatória:** durante o Execute, usar o **máximo de subagentes possível** (dentro dos limites da plataforma) para implementar, revisar e validar em paralelo. Só executar de forma sequencial quando houver dependência estrita entre etapas.

Ciclo por tarefa:

0. **COMPLIANCE GATE** — verificar checklist (spec existe, lida, arquivos listados, dentro do escopo, state atualizado). Se □ = false → para e pergunta.
1. **Plan** da tarefa
2. **RED** — teste falhando
3. **GREEN** — implementação mínima
4. **GATE CHECK** — executar validações
5. **Review** — checar aderência à spec
6. próxima tarefa

### Gate mínimo para concluir

Não pode encerrar sem evidência de:

- **Compliance Gate verificado** (todos os 5 □ = true) — **por cada tarefa**
- testes executados (comando + resultado)
- validação funcional da regra principal
- arquivos alterados
- confirmação de que não ficou item pedido pendente sem justificativa

---

## State (sempre atualizar)

Atualizar `.fdd/state.md` com:

- fase atual/status
- resumo objetivo
- arquivos alterados
- testes executados e resultado
- pendências e próximos passos

State é fonte de continuidade entre sessões.

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

## Pitfalls críticos

- Pular spec por "mudança pequena"
- Encerrar sem teste rodado
- Não atualizar state
- Derivar escopo quando a Lua pedir ajuste novo
- Presumir requisito sem confirmação em caso ambíguo
- **EDITAR sem passar no Compliance Gate — PARAR e verificar checklist primeiro**
- **Fingir que passou no gate sem verificar os5 itens**

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
