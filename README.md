# FDD — Feature-Driven Development (Hermes)

> Spec-first, rastreável, validado. Nada fora da spec. Comandos via `/xfdd`.

## Objetivo

**L-Spec PI adaptado para o Hermes** — mesmo pipeline sequencial fixo com gates bloqueantes.

Serve para projetos novos E existentes. Tudo começa com spec, nunca com código.

- Discovery adaptativo por tipo
- Research obrigatório (análise do codebase)
- Specify testável
- Tasks rastreáveis
- Execute com **Compliance Gate bloqueante**
- **Autosave OBRIGATÓRIO entre fases**
- State atualizado ao final

**Single entry point:** `/xfdd [request]`

---

## Comandos

| Comando | Quando usar |
|---------|-------------|
| `/xfdd new` | Projeto novo |
| `/xfdd feature` | Feature em projeto existente |
| `/xfdd reverse` | Código existente sem specs |
| `/xfdd bug` | Bug a corrigir |

> `/fdd` também funciona como alias.

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
│ Discuss*   │ OPCIONAL — só se área cinzenta/ambígua                  │
│ Specify    │ SEMPRE (OBRIGATÓRIO)                                    │
│ Clarify*   │ OPCIONAL — só se ambiguidade nos requisitos             │
│ Design*    │ OPCIONAL — só se necessidade arquitetural              │
│ Tasks      │ SEMPRE                                                  │
│ Execute    │ SEMPRE                                                  │
└────────────┴──────────────────────────────────────────────────────────┘

(*) OPCIONAL — só executa se necessário
```

---

## Gates Bloqueantes

### Compliance Gate (antes de QUALQUER edit)

```
┌────────────────────────────────────────────────────────────────┐
│ COMPLIANCE GATE CHECKLIST                                      │
├────────────────────────────────────────────────────────────────┤
│ □ spec.md existe                                                │
│ □ spec.md foi lida e compreendida                              │
│ □ design.md existe e foi lido (se houver)                       │
│ □ Arquivos a editar estão listados                              │
│ □ Mudança NÃO foge do escopo                                    │
│ □ State atualizado na última fase                               │
└────────────────────────────────────────────────────────────────┘

→ Se qualquer □ = false → BLOQUEIA. Não edita.
```

### State Saved Gate (entre fases)

```
┌────────────────────────────────────────────────────────────────┐
│ STATE SAVED GATE                                               │
├────────────────────────────────────────────────────────────────┤
│ □ .specs/project/STATE.md existe                               │
│ □ Última fase registrada                                        │
│ □ Pendências atualizadas                                       │
└────────────────────────────────────────────────────────────────┘

→ Se qualquer □ = false → SALVA ANTES de iniciar nova fase.
```

**AMBOS os gates são obrigatórios.**

---

## Artifact Enforcement

**"The artifact one writes is the next one's input."**

```
Discovery  → .specs/project/STATE.md  → Research
Research   → .specs/features/[name]/research.md → Specify
Specify    → .specs/features/[name]/spec.md  → Design (opcional)
Design     → .specs/features/[name]/design.md → Tasks (se existir)
Tasks      → .specs/features/[name]/tasks.md → Execute
Execute    → working-tree changes     → Validate
```

**Sem artifact da fase anterior → Execute bloqueado.**

---

## Discovery adaptativo

| Tipo | Fluxo | Perguntas |
|------|-------|-----------|
| Projeto novo | `/xfdd new` | 17 perguntas em 6 fases |
| Feature | `/xfdd feature` | 5 perguntas focadas |
| Bug | `/xfdd bug` | 6 perguntas curtas |
| Reverse | `/xfdd reverse` | scan automático |

**Discovery é PERGUNTAS PARA O USUÁRIO — nunca inventar respostas.**

---

## Research — Análise do Codebase

**OBRIGATÓRIO. Não pula. Não pergunta ao usuário.**

Pesquisa o **código existente** — onde está implementada a feature, quem chama essa função, qual pattern o código usa.

Output: `.specs/features/[name]/research.md`

**Sem research = BLOQUEIA Specify.**

---

## Specify (obrigatório)

Criar `.specs/features/<feature>/spec.md` com:
- Requisitos funcionais numerados e testáveis
- Critérios de aceite

Formato:
```
WHEN [ação/evento] THEN sistema DEVE [resposta]
```

---

## Tasks (obrigatório)

Criar `.specs/features/<feature>/tasks.md` com tarefas atômicas:
- o que fazer
- onde fazer
- dependências
- critério de pronto

---

## Execute (com Gate)

Ciclo por tarefa:
0. **COMPLIANCE GATE** — verificar checklist (por tarefa, não uma vez só)
1. **Plan** da tarefa
2. **RED** — teste falhando
3. **GREEN** — implementação mínima
4. **GATE CHECK** — executar validações
5. **Review** — checar aderência à spec
6. **AUTOSAVE** — salvar `.specs/project/STATE.md` (OBRIGATÓRIO)

---

## State (sempre atualizar) — OBRIGATÓRIO

**State saving NÃO é opcional.**

Ao final de cada fase:
```
□ .specs/project/STATE.md atualizado
□ Última fase registrada
□ Pendências atualizadas
□ Commits feitos (se aplicável)
```

---

## Regras fixas

1. Nada fora da spec
2. Sempre reler spec antes de implementar
3. **Preflight obrigatório:** ler `.specs/` inteiro antes de codar → responder "Contexto lido"
4. **Compliance Gate é BLOQUEANTE** — antes de qualquer edit, verificar 6 itens
5. **`.specs/` deve ser versionado no Git** — subir para remote
6. **Nunca versionar `.env`** — manter apenas `.env.example`
7. **Usar máximo de subagentes** — paralelismo sempre que possível
8. Sem pular fases obrigatórias
9. Validate é dentro do Execute (não é fase separada)
10. Toda solicitação da Lua vira item rastreável (`R#`)
11. Se escopo mudar, replanejar antes de continuar
12. State sempre atualizado
13. Se não há evidência de teste/validação, não está concluído

---

## Enforcement Pattern

**AMBOS gates obrigatórios:**

```
        ANTES DE CADA FASE:
┌──────────────────┐    ┌──────────────────┐
│ Compliance Gate  │ AND │ State Saved Gate │
│ (6 itens)        │     │ (verificação)    │
└────────┬─────────┘    └────────┬─────────┘
         │                       │
         └───────────┬───────────┘
                     ▼
            ┌──────────────┐
            │ AVANÇAR?    │
            └──────────────┘

AMBOS gates devem passar para prosseguir.
Se um falhar → BLOQUEIA e resolve antes.
```

---

## Git Workflow

### Config (FAZER PRIMEIRO)
```bash
git config --local user.name "by-lua"
git config --local user.email "noelia.assis@by-lua.com"
```

### Onde editar
**⚠️ Editar NO clone git, não no skills local.**

```
~/.pi/agent/git/github.com/by-lua/<REPO>/   ← CLONE GIT (edit here)
~/.hermes/profiles/luna/skills/              ← LOCAL ONLY (don't edit)
```

### Push rejeitado
```bash
git pull origin main --rebase && git push origin main
```

### Author format

| Formato | GitHub mostra |
|---------|--------------|
| `by-lua <noelia.assis@by-lua.com>` | ✅ Avatar + nome linkado |
| `by-lua/BY-LUA - NOELIA.ASSIS <...>` | ❌ Texto cru, sem link |

---

## Hermes Tools — Code Navigation

**NUNCA use bash grep/find para navegação de código.**

Usar SOMENTE `agent-lsp` (MCP server, 66 tools):

| Tarefa | Use | Not |
|--------|-----|-----|
| Ver estrutura | `list_symbols` | Read manual |
| Encontrar símbolo | `find_symbol` | Grep |
| Todas referências | `find_references` | Grep name |
| Entender símbolo | `inspect_symbol` | Read file |
| O que chama função | `find_callers` | Grep |
| Ver impacto antes de editar | `blast_radius` | - |
| Substituir corpo de função | `replace_symbol_body` | Text match |

**Se agent-lsp não disponível → usa tools nativas** (`read_file`, `search_files`), **NÃO BLOQUEIA**.

---

## NUNCA Rules

**NUNCA:**
- Pular fases obrigatórias (especialmente Research!)
- Quick mode
- Auto-sizing
- Implementar sem analisar codebase primeiro
- Editar sem passar no Compliance Gate
- Fingir que passou no gate sem verificar os 6 itens
- Não salvar `.specs/project/STATE.md` — autosave é obrigatório

**SEMPRE:**
- Reler spec antes de implementar
- Autosave de estado em cada fase
- Gate antes de encerrar Execute
- Passar no Compliance Gate antes de QUALQUER edit

---

## Pitfalls Críticos

- **Discovery: NÃO inventar respostas** — fazer perguntas ao usuário
- **Research: NÃO perguntar ao usuário** — analisar o código existente
- Pular spec por "mudança pequena"
- Encerrar sem teste rodado
- Não atualizar state
- **EDITAR sem passar no Compliance Gate — PARAR e verificar checklist primeiro**
- **EDITAR NO DIRETÓRIO ERRADO** — editar SEMPRE no clone git
- **Compliance Gate sem State Saving** — state fica pendente entre fases
- **NÃO existe pasta `fixes/`** — bugs usam prefixo `fix-` em `features/`
- **Feature existente = atualizar docs e continuar** — não reiniciar pipeline

---

## Formato de resposta operacional

Ao final de cada bloco:

```
[Discovery | Specify | Tasks | Execute] ✓
Feito: ...
Próximo: ...
```

Se bloqueado:

```
[Bloqueado] — razão
Aguardando: ...
```

**Compliance Gate — formato (por tarefa):**

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
Aguardando: decisão da Lua antes de editar
```

---

## Referências de operação

- `references/compliance-gate-pattern.md` — padrão obrigatório de Compliance Gate
- `references/platform-tooling-mapping.md` — mapeamento PI vs Hermes
- `references/pi-packages-reference-only.md` — alerta: packages fake
- `references/npm-publish-pi-packages.md` — como publicar packages npm

---

## Créditos

Base conceitual inspirada no **L-Spec PI** (by-lua) com evolução para o ecossistema Hermes por **by-lua**.