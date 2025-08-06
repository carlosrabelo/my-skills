# my-skills

Uma biblioteca pessoal de skills reutilizáveis para workflows de desenvolvimento de software (Claude Code, Cursor, OpenCode, Gemini CLI, Antigravity, Qwen).

## Índice

- [Visão Geral](#visão-geral)
- [Skills](#skills)
- [Pré-requisitos](#pré-requisitos)
- [Instalação](#instalação)
- [Uso](#uso)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Adicionando Novas Skills](#adicionando-novas-skills)
- [Contribuindo](#contribuindo)
- [Licença](#licença)

## Visão Geral

Este repositório contém um conjunto curado de skills que codificam boas práticas de desenvolvimento de software. São usadas por múltiplos runtimes de agentes (Claude Code, Cursor e outros CLIs). Cada skill é um guia detalhado que o agente usa para fornecer assistência consistente em diferentes tarefas e projetos.

As skills cobrem estrutura de projetos Go, C++ e Python, workflows Git, documentação de código e ferramentas GitHub — como base de conhecimento instalável globalmente via **sync-skills** (seis destinos; pastas inexistentes são ignoradas).

## Skills

### Git & GitHub

| Skill | Modo | Descrição |
|-------|------|-----------|
| `git-commit-suggest` | manual | Analisa as mudanças no repositório e sugere comandos `git add` + `git commit` no formato Conventional Commits |
| `github-repo-editor` | manual | Gera comandos `gh repo edit` com descrição e tópicos derivados automaticamente para um repositório |
| `gitignore-skeleton` | agent | Estrutura padrão de `.gitignore`: segredos, tipo de projeto (Go/C++/Python/Node), editores, SO. Cria do zero ou atualiza arquivos existentes. |

### C++

| Skill | Modo | Descrição |
|-------|------|-----------|
| `cpp-skeleton` | agent | Layout C++ padrão: Makefile orquestrador na raiz + `projectname/` (`src/`, `tests/`, `lib/`), scripts `.make/`, g++, Catch2. Cria do zero ou reorganiza projetos existentes. |

### Go

| Skill | Modo | Descrição |
|-------|------|-----------|
| `go-skeleton` | agent | Layout padrão de projetos Go: flat root com `go.mod`, Makefile único, `cmd/`, `internal/`, scripts `.make/`. Cria do zero ou reorganiza projetos existentes. |

### Python

| Skill | Modo | Descrição |
|-------|------|-----------|
| `python-skeleton` | agent | Layout padrão de projetos Python: flat root, `pyproject.toml`, `.venv`, Makefile único, scripts `.make/`. Cria do zero ou reorganiza projetos existentes. |

### Tooling

| Skill | Modo | Descrição |
|-------|------|-----------|
| `makefile-skeleton` | agent | Estrutura padrão de Makefile: linhas de abertura, ajuda autodocumentada, targets padrão, delegação para scripts em `.make/`. Cria do zero ou atualiza arquivos existentes. |

### Documentação

| Skill | Modo | Descrição |
|-------|------|-----------|
| `readme-skeleton` | agent | Estrutura padrão de README e sincronização bilíngue: ordem de seções, formato Highlights, seção Development, tradução README-PT.md. Cria do zero ou atualiza arquivos existentes. |

### Projeto

| Skill | Modo | Descrição |
|-------|------|-----------|
| `esp32-skeleton` | agent | Layout ESP32/PlatformIO: PlatformIO, `src/`, scripts `.make/`. |
| `monorepo-skeleton` | agent | Layout padrão de monorepo multi-linguagem: raízes por componente, Makefile orquestrador na raiz, convenções de nomes consistentes. |
| `skeleton-scaffold` | agent | Orquestrador central: detecta o stack, aplica um skeleton de linguagem/plataforma e em seguida gitignore-skeleton e readme-skeleton. Matriz de deteção: só em [skeleton-scaffold/SKILL.md](skeleton-scaffold/SKILL.md). |

### Meta

| Skill | Modo | Descrição |
|-------|------|-----------|
| `sync-skills` | manual | Copia todas as skills públicas deste repositório para cada diretório global configurado (veja [Instalação](#instalação)) |

**Modos**:
- `manual` — invocado explicitamente pelo usuário (ex: `/git-commit-suggest` no Claude Code)
- `agent` — invocado automaticamente quando o agente detecta contexto relevante

## Pré-requisitos

- Pelo menos um diretório de destino (o script de sync **não cria** pastas que não existam):
  - [Claude Code](https://docs.anthropic.com/en/docs/claude-code): `~/.claude/skills/`
  - Cursor: `~/.cursor/skills/` ([Agent Skills](https://cursor.com/docs))
  - OpenCode: `~/.config/opencode/skills/`
  - Gemini / Antigravity / Qwen: `~/.gemini/skills/`, `~/.gemini/antigravity/skills/`, `~/.qwen/skills/`
- **Nota (Cursor):** não instale skills pessoais em `~/.cursor/skills-cursor/` — esse caminho é reservado às skills embutidas do Cursor.

## Instalação

Clone o repositório e execute o skill de sincronização para instalar todas as skills globalmente:

```bash
git clone https://github.com/carlosrabelo/my-skills
cd my-skills
```

Em seguida, execute o fluxo **sync-skills** a partir deste repositório (por exemplo `/sync-skills` no Claude Code, ou o bloco bash em `.claude/skills/sync-skills/SKILL.md` / `.cursor/skills/sync-skills/SKILL.md`).

Isso copia cada skill **pública** (diretórios na raiz com `SKILL.md`) para cada destino **existente**: `~/.claude/skills/`, `~/.config/opencode/skills/`, `~/.gemini/skills/`, `~/.gemini/antigravity/skills/`, `~/.qwen/skills/` e `~/.cursor/skills/`. Se `~/.cursor/` existir mas `~/.cursor/skills/` não, o sync-skills cria essa pasta primeiro; outros destinos em falta são ignorados.

### Instalação manual

Para instalar uma skill individualmente (exemplos):

```bash
cp -r git-commit-suggest ~/.claude/skills/
cp -r git-commit-suggest ~/.cursor/skills/
```

## Uso

### Invocando uma skill manual

No Claude Code, use comandos slash (por exemplo `/git-commit-suggest`). No Cursor, invoque conforme a UI de Agent Skills ou ao nomear a skill quando fizer sentido.

### Skills de agente

As skills de agente são carregadas pelo contexto quando o agente identifica correspondência — por exemplo, **skeleton-scaffold** ao detectar tipo de projeto ou fazer scaffold, ou um **\*-skeleton** específico (ex.: `go-skeleton`) quando a tarefa já é só desse stack.

### Atualizando skills

Após modificar skills neste repositório, execute **sync-skills** de novo para atualizar todos os destinos existentes.

## Estrutura do Projeto

```
my-skills/
├── .claude/
│   └── skills/
│       ├── audit-skills/
│       ├── check-sync/
│       └── sync-skills/
│           └── SKILL.md
├── .cursor/
│   └── skills/
│       ├── audit-skills/
│       ├── check-sync/
│       └── sync-skills/
├── cpp-skeleton/
│   └── SKILL.md
├── esp32-skeleton/
│   └── SKILL.md
├── git-commit-suggest/
│   └── SKILL.md
├── github-repo-editor/
│   └── SKILL.md
├── gitignore-skeleton/
│   ├── SKILL.md
│   └── references/
├── go-skeleton/
│   ├── SKILL.md
│   └── references/
├── makefile-skeleton/
│   ├── SKILL.md
│   └── references/
├── monorepo-skeleton/
│   ├── SKILL.md
│   └── references/
├── python-skeleton/
│   ├── SKILL.md
│   └── references/
├── readme-skeleton/
│   ├── SKILL.md
│   └── references/
├── skeleton-scaffold/
│   └── SKILL.md
├── README.md
└── README-PT.md
```

Cada skill vive em seu próprio diretório. Skills com subdiretório `references/` dividem seu conteúdo em múltiplos arquivos — `SKILL.md` é o ponto de entrada carregado automaticamente, enquanto os arquivos em `references/` são carregados sob demanda.

## Adicionando Novas Skills

1. Crie um novo diretório com o nome da skill (use kebab-case):
   ```bash
   mkdir minha-nova-skill
   ```

2. Crie `minha-nova-skill/SKILL.md` com a definição da skill. No mínimo inclua:
   - Frontmatter com `name`, `description`, `category`, `mode`, `shared`
   - Para Cursor: `description` ≤ 1024 caracteres; com `mode: manual`, inclua `disable-model-invocation: true`
   - Uma seção de visão geral explicando quando usar a skill
   - Instruções detalhadas e exemplos

3. Se a skill for um novo **layout de linguagem ou plataforma**, adicione uma linha à tabela de deteção em **`skeleton-scaffold/SKILL.md`** (fonte única) e uma linha ao inventário em **[AGENTS.md](AGENTS.md)** se for skill pública sincronizada.

4. Execute **sync-skills** para instalá-la em todos os diretórios globais configurados.

## Contribuindo

1. Faça um fork do repositório
2. Crie uma branch de feature: `git checkout -b feat/nome-da-nova-skill`
3. Adicione ou atualize a skill seguindo a estrutura acima
4. Faça o commit usando Conventional Commits: `git commit -m "feat: add nome-da-nova-skill skill"`
5. Faça o push e abra um Pull Request

## Licença

MIT
