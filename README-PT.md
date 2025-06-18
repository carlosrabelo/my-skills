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
| `github-repo-editor` | manual | Gera comandos `gh repo edit` com descrição e tópicos derivados automaticamente para um repositório |
| `gitignore-skeleton` | agent | Estrutura padrão de `.gitignore`: ferramentas de IA, segredos, tipo de projeto (Go/Python/Node), editores, SO. Cria do zero ou atualiza arquivos existentes. |

### Go

| Skill | Modo | Descrição |
|-------|------|-----------|
| `go-skeleton` | agent | Layout padrão de projetos Go: flat root com `go.mod`, Makefile único, `cmd/`, `internal/`, scripts `make/`. Cria do zero ou reorganiza projetos existentes. |

### Python

| Skill | Modo | Descrição |
|-------|------|-----------|
| `python-skeleton` | agent | Layout padrão de projetos Python: flat root, `pyproject.toml`, `.venv`, Makefile único, scripts `make/`. Cria do zero ou reorganiza projetos existentes. |

### Tooling

| Skill | Modo | Descrição |
|-------|------|-----------|
| `makefile-skeleton` | agent | Estrutura padrão de Makefile: linhas de abertura, ajuda autodocumentada, targets padrão, padrão delegate-to-make/. Cria do zero ou atualiza arquivos existentes. |

### Documentação

| Skill | Modo | Descrição |
|-------|------|-----------|
| `readme-skeleton` | agent | Estrutura padrão de README e sincronização bilíngue: ordem de seções, formato Highlights, seção Development, tradução README-PT.md. Cria do zero ou atualiza arquivos existentes. |

### Projeto

| Skill | Modo | Descrição |
|-------|------|-----------|
| `monorepo-skeleton` | agent | Layout padrão de monorepo multi-linguagem: raízes por componente, Makefile orquestrador na raiz, convenções de nomes consistentes. Cria do zero ou reorganiza repositórios existentes. |

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

As skills de agente são ativadas automaticamente quando o Claude detecta contexto relevante — por exemplo, `go-skeleton` é ativada quando você está trabalhando em um projeto Go e pergunta sobre organização.

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
