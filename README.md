# FDD-Hermes

**Feature-Driven Development v2.8.0** — Spec-first, rastreável, validado. Nada fora da spec.

Pipeline adaptativo inspirado no L-Spec PI, com Compliance Gate bloqueante e autosave obrigatório entre fases.

---

## O que é FDD

Metodologia de desenvolvimento onde toda mudança parte da **spec antes do código**. Cada fase produz um artifact que a próxima fase consome. Dois gates bloqueantes garantem disciplina:

- **Compliance Gate**: antes de cada edit — verifica spec, contexto, escopo
- **State Saved Gate**: entre fases — verifica state atualizado

---

## Pipeline

```
Discovery → [Discuss*] → Research → Specify → [Clarify*] → [Design*] → Tasks → Execute
```

| FASE | QUANDO |
|------|--------|
| Discovery | SEMPRE (17 perguntas em 6 fases para projeto novo) |
| Research | SEMPRE — análise do codebase, NÃO perguntas ao usuário |
| Discuss | OPCIONAL — só se área cinzenta/ambígua |
| Specify | SEMPRE |
| Clarify | OPCIONAL — só se ambiguidade nos requisitos |
| Design | OPCIONAL — só se necessidade arquitetural |
| Tasks | SEMPRE |
| Execute | SEMPRE |

---

## Gates Bloqueantes

### Compliance Gate (antes de cada tarefa no Execute)

```
□ spec.md existe
□ spec.md foi lida e compreendida
□ design.md existe e foi lido (se houver)
□ Arquivos a editar estão listados
□ Mudança NÃO foge do escopo
□ State atualizado na última fase

→ Se qualquer □ = false → BLOQUEIA. Não edita.
```

### State Saved Gate (entre fases)

```
□ .specs/project/STATE.md existe
□ Última fase registrada
□ Pendências atualizadas

→ Se qualquer □ = false → SALVA ANTES de iniciar nova fase.
```

---

## Artifact Enforcement

Cada fase **PRODUZ** um artifact que a próxima **PRECISA**. Sem artifact, bloqueia.

| Pipeline | Produz | Feedsa |
|----------|--------|--------|
| Discovery | `.specs/project/STATE.md` | Research |
| Research | `.specs/features/[name]/research.md` | Specify |
| Specify | `.specs/features/[name]/spec.md` | Design ou Tasks |
| Design | `.specs/features/[name]/design.md` | Tasks |
| Tasks | `.specs/features/[name]/tasks.md` | Execute |

---

## NUNCA Rules

- ❌ Pular fases obrigatórias (especialmente Research!)
- ❌ Quick mode / auto-sizing
- ❌ Implementar sem analisar codebase primeiro
- ❌ Editar sem passar no Compliance Gate
- ❌ Não salvar `.specs/project/STATE.md` entre fases
- ❌ Editar em `~/.hermes/profiles/luna/skills/` (não é git)

---

## Qual Comando Usar

### Single Entry Point — `/xfdd`

O comando `/xfdd` é o **único ponto de entrada**. Ele auto-detecta o tipo de projeto e executa o pipeline completo.

**Não precisa escolher comando por fase.** O sistema avança automaticamente.

| Cenário | Comando | O que acontece |
|---------|---------|----------------|
| Projeto novo | `/xfdd new` | Discovery completo (17 perguntas em 6 fases) → Research → Specify → ... |
| Feature em projeto existente | `/xfdd feature` | Discovery focado → Research → Specify → ... |
| Bug em projeto existente | `/xfdd corrigir bug de...` | Discovery curto (3 perguntas) → Research → Tasks → Execute |
| Projeto existente sem specs | `/xfdd reverse` | **Reverse mode** — survey do código → spec gerada automaticamente |

> `/fdd` também funciona como alias de `/xfdd`.

### Reverse — Quando Usar

Use `/xfdd reverse` quando:

- Projeto existe mas **não tem `.specs/`**
- Precisa entender estrutura antes de mexer
- Vai接手 projeto de outra pessoa

O reverse escaneia o código, gera `.specs/project/` e `.specs/features/[name]/` automaticamente.

---

## Comandos

| Comando | Descrição |
|---------|-----------|
| `/xfdd new` | Projeto novo (17 perguntas em 6 fases) |
| `/xfdd feature` | Nova feature em projeto existente |
| `/xfdd reverse` | Projeto existente sem specs (survey do código) |

> `/fdd` também funciona como alias.

---

## Hermes Tools

**Code navigation via `agent-lsp` MCP** (66 tools LSP):

- `list_symbols`, `find_symbol` — estrutura do arquivo
- `go_to_definition`, `find_references`, `find_callers` — navegação contextual
- `inspect_symbol` — tipo, docs, signature
- `blast_radius` — impacto a montante antes de editar
- `get_diagnostics` — erros/warnings

**Bash grep/find** apenas para output pipeado ou casos pontuais não cobertos pelo LSP.

> Não usar `pi-cymbal` — `agent-lsp` é o primary.

---

## Git Workflow — Onde Editar

```
~/.pi/agent/git/github.com/by-lua/<REPO>/   ← CLONE GIT (edit here)
~/.hermes/profiles/luna/skills/              ← LOCAL ONLY (don't edit)
```

Edição no clone com `.git/` → commit → push. Skills local não é git.

---

## Estrutura de Projeto

```
.specs/
├── project/
│   ├── STATE.md      # Estado do projeto (obrigatório)
│   ├── PROJECT.md    # Docs do projeto
│   └── ROADMAP.md    # Roadmap
└── features/
    └── [name]/
        ├── research.md  # Research output
        ├── spec.md      # Spec testável
        ├── design.md    # Design (se aplicável)
        └── tasks.md     # Tarefas atômicas
```

> `.specs/` deve ser versionado no Git. Não versionar `.env`.

---

## Propagação de Skill

Após update no repo, copiar para todos os perfis:

```bash
cp ~/.pi/agent/git/github.com/by-lua/FDD-Hermes/SKILL.md \
   /root/.hermes/skills/software-development/fdd/SKILL.md

cp ~/.pi/agent/git/github.com/by-lua/FDD-Hermes/SKILL.md \
   /root/.hermes/profiles/kira/skills/software-development/fdd/SKILL.md

cp ~/.pi/agent/git/github.com/by-lua/FDD-Hermes/SKILL.md \
   /root/.hermes/profiles/kira2/skills/software-development/fdd/SKILL.md
```

---

## Repo

🔗 [https://github.com/by-lua/FDD-Hermes](https://github.com/by-lua/FDD-Hermes)