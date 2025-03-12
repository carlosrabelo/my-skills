# my-skills

Uma biblioteca pessoal de skills reutilizáveis do Claude Code para workflows de desenvolvimento de software.

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

Este repositório contém um conjunto curado de skills do [Claude Code](https://docs.anthropic.com/en/docs/claude-code) que codificam boas práticas de desenvolvimento de software. Cada skill é um guia detalhado que o Claude invoca durante conversas para fornecer assistência consistente e de alta qualidade em diferentes tarefas e projetos.

As skills cobrem estrutura de projetos Go e Python, workflows Git, documentação de código e ferramentas GitHub — funcionando como uma base de conhecimento compartilhada que pode ser instalada em qualquer projeto via `sync-skills`.

## Skills

### Git & GitHub

| Skill | Modo | Descrição |
|-------|------|-----------|
| `git-commit-suggest` | manual | Analisa as mudanças no repositório e sugere comandos `git add` + `git commit` no formato Conventional Commits |
| `git-workflow-go` | manual | Guia para organizar commits atômicos, histórico limpo e workflows em equipe em projetos Go |
| `github-repo-editor` | manual | Gera comandos `gh repo edit` com descrição e tópicos derivados automaticamente para um repositório |

### Go

| Skill | Modo | Descrição |
|-------|------|-----------|
| `go-project-structure` | agent | Layout padrão de projetos Go: `src/` com `go.mod`, hierarquia dupla de Makefile, `cmd/`, `internal/`, scripts `run/` |
| `go-reorganize-refactor` | agent | Padrões passo a passo para refatorar código Go existente na estrutura padrão de projetos |
| `go-commenting-en` | agent | Guia para escrever comentários claros e idiomáticos em inglês e documentação godoc em código Go |

### Python

| Skill | Modo | Descrição |
|-------|------|-----------|
| `python-project-structure` | agent | Layout padrão de projetos Python: layout `src/`, `pyproject.toml`, `.venv`, hierarquia de Makefile, scripts `run/` |

### Documentação

| Skill | Modo | Descrição |
|-------|------|-----------|
| `readme-bilingual-sync` | agent | Mantém `README.md` (inglês) e `README-PT.md` (português) sincronizados e atualizados |

### Meta

| Skill | Modo | Descrição |
|-------|------|-----------|
| `sync-skills` | manual | Copia todas as skills deste repositório para `~/.claude/skills/` para uso global |

**Modos**:
- `manual` — invocado explicitamente pelo usuário (ex: `/git-commit-suggest`)
- `agent` — invocado automaticamente pelo Claude quando as condições de ativação são atendidas

## Pré-requisitos

- CLI do [Claude Code](https://docs.anthropic.com/en/docs/claude-code) instalada
- Diretório de skills em `~/.claude/skills/` (criado automaticamente pelo Claude Code)

## Instalação

Clone o repositório e execute o skill de sincronização para instalar todas as skills globalmente:

```bash
git clone https://github.com/carlosrabelo/my-skills
cd my-skills
```

Em seguida, dentro de uma sessão do Claude Code neste diretório, execute:

```
/sync-skills
```

Isso copia todas as skills para `~/.claude/skills/`, tornando-as disponíveis em todos os projetos.

### Instalação manual

Para instalar uma skill individualmente:

```bash
cp -r git-commit-suggest ~/.claude/skills/
```

## Uso

### Invocando uma skill manual

Em qualquer sessão do Claude Code, digite o nome da skill como um comando slash:

```
/git-commit-suggest
/github-repo-editor
/sync-skills
```

### Skills de agente

As skills de agente são ativadas automaticamente quando o Claude detecta contexto relevante — por exemplo, `go-project-structure` é ativada quando você está trabalhando em um projeto Go e pergunta sobre organização.

### Atualizando skills

Após modificar skills neste repositório, execute `/sync-skills` novamente dentro de uma sessão do Claude Code para propagar as mudanças para `~/.claude/skills/`.

## Estrutura do Projeto

```
my-skills/
├── .claude/
│   └── skills/
│       └── sync-skills/        ← Skill meta interna (não sincronizada)
│           └── SKILL.md
├── git-commit-suggest/
│   └── SKILL.md
├── git-workflow-go/
│   └── SKILL.md
├── github-repo-editor/
│   └── SKILL.md
├── go-commenting-en/
│   └── SKILL.md
├── go-project-structure/
│   └── SKILL.md
├── go-reorganize-refactor/
│   └── SKILL.md
├── python-project-structure/
│   └── SKILL.md
├── readme-bilingual-sync/
│   └── SKILL.md
├── README.md
└── README-PT.md
```

Cada skill vive em seu próprio diretório contendo um único arquivo `SKILL.md` com a definição completa da skill, instruções e exemplos.

## Adicionando Novas Skills

1. Crie um novo diretório com o nome da skill (use kebab-case):
   ```bash
   mkdir minha-nova-skill
   ```

2. Crie `minha-nova-skill/SKILL.md` com a definição da skill. No mínimo inclua:
   - Frontmatter com `name`, `description`, `category`, `mode`, `shared`
   - Uma seção de visão geral explicando quando usar a skill
   - Instruções detalhadas e exemplos

3. Execute `/sync-skills` em uma sessão do Claude Code para instalá-la globalmente.

## Contribuindo

1. Faça um fork do repositório
2. Crie uma branch de feature: `git checkout -b feat/nome-da-nova-skill`
3. Adicione ou atualize a skill seguindo a estrutura acima
4. Faça o commit usando Conventional Commits: `git commit -m "feat: add nome-da-nova-skill skill"`
5. Faça o push e abra um Pull Request

## Licença

MIT
