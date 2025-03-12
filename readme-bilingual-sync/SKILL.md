---
name: readme-bilingual-sync
description: Keep README.md (English) and README-PT.md (Portuguese) synchronized and up-to-date. Use whenever project changes, features are added, or documentation needs updating. Ensures both versions have identical structure and information, translated appropriately.
mode: agent
category: documentation
shared: true
---

# Bilingual README Sync

Maintain synchronized and up-to-date README files in both English and Portuguese.

## Overview

This skill ensures your project has comprehensive bilingual documentation where README.md (English) is the source and README-PT.md (Portuguese) is a complete translation. Both files maintain identical structure and organization, with only the language differing.

## When to Use

Use this skill whenever you:
- Start a new project (create both README files)
- Add new features or change project behavior
- Update project structure or requirements
- Change installation or usage instructions
- Add new dependencies or change tech stack
- Update project status or roadmap
- Fix documentation errors or inconsistencies
- Want to review if documentation is current

## README Structure

Both files should follow this identical structure:

```markdown
# Project Name

One-line description of what the project does.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Development](#development)
- [Testing](#testing)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Overview

Comprehensive description of:
- What the project does
- Problem it solves
- Why it exists

## Features

List of key features:
- Feature 1
- Feature 2
- Feature 3

## Prerequisites

What's needed before using:
- Requirement 1 (version X.Y+)
- Requirement 2 (version A.B+)
- Requirement 3

## Installation

Step-by-step installation instructions:

### From Source
```bash
git clone https://github.com/username/project
cd project
make build
./run/install.sh
```

### From Binary
Download from releases page.

## Quick Start

Minimal example to get running in < 5 minutes:

```bash
project init
project run
# Output shows success
```

## Usage

Detailed usage examples:

### Basic Usage
```bash
project command --flag value
```

### Advanced Usage
```bash
project command --advanced-flag value --other-flag
```

## Project Structure

Explanation of directory organization:

```
project/
├── bin/           ← Compiled binaries
├── cfg/           ← Configuration
├── src/           ← Source code
│   ├── cmd/       ← Commands
│   └── internal/  ← Internal packages
├── run/           ← Scripts
└── README.md      ← This file
```

(Only if project structure matters to users)

## Development

How to set up development environment:

### Setup
```bash
git clone https://github.com/username/project
cd project
make test
```

### Building
```bash
make build
./bin/project --help
```

### Running Tests
```bash
make test
```

## Testing

Test coverage and testing strategy:

- Unit tests: `src/*_test.go`
- Run with: `make test`
- Coverage: Run `go test -cover ./...`

## Configuration

Configuration options if applicable:

```yaml
# cfg/config.yaml
log_level: info
port: 8080
database: postgres://localhost/db
```

## Troubleshooting

Common issues and solutions:

### Issue: "command not found"
**Solution**: Ensure `make install` ran successfully and binary is in PATH.

### Issue: "connection refused"
**Solution**: Check if service is running: `make run`

## Contributing

How others can contribute:

1. Fork repository
2. Create feature branch: `git checkout -b feature/name`
3. Commit changes: `git commit -m "feat: description"`
4. Push to branch: `git push origin feature/name`
5. Open Pull Request

## License

License information and link to LICENSE file.

---

## Notes

- Bilingual documentation ensures global accessibility
- Both versions should be equally detailed
- Update both when making any documentation change
- Keep structure identical for easy comparison
```

---

## Synchronization Workflow

### When Documentation Changes

**1. Update README.md first** (English source)
```bash
nano README.md
# Make changes in English
git diff README.md  # Review changes
```

**2. Update README-PT.md** (Portuguese translation)
```bash
nano README-PT.md
# Translate and apply same changes
git diff README-PT.md  # Review changes
```

**3. Verify both are synchronized**
```bash
# Check structure is identical
grep "^#" README.md | wc -l
grep "^#" README-PT.md | wc -l
# Should be the same number

# Check both have same sections
grep "^## " README.md
grep "^## " README-PT.md
# Should have identical sections
```

**4. Commit together**
```bash
git add README.md README-PT.md
git commit -m "docs: update documentation for feature X"
```

---

## Best Practices

### 1. Structure Parity
Both files must have:
- ✅ Identical section headers
- ✅ Identical section order
- ✅ Same code examples (don't translate code!)
- ✅ Same links and references

### 2. Translation Quality
- ✅ Use clear, professional Portuguese
- ✅ Maintain tone from English version
- ✅ Don't translate technical terms unnecessarily
- ✅ Keep sentences shorter for clarity

### 3. Maintenance
- ✅ Update both files in same commit
- ✅ Review for accuracy before committing
- ✅ Keep code examples identical
- ✅ Use same variable/function names

### 4. What NOT to Translate

❌ Don't translate:
- Variable names: `config.yaml` → `config.yaml` (not `config.yml`)
- Function names: `make build` → `make build`
- Command output: Show actual output, not translated
- File paths: `/home/user` → `/home/user`
- Technical terms: `Git`, `CLI`, `API`, `database`

✅ Do translate:
- Descriptions and explanations
- Instructions and steps
- Section headers
- Comments in examples (optionally)

---

## Example: Adding New Feature Documentation

### In README.md (English)

```markdown
## New Feature: Validation

The new validation system checks input before processing:

```bash
project validate --input file.yaml
```

Results in:
```
✓ Configuration valid
✓ All requirements met
```
```

### In README-PT.md (Portuguese)

```markdown
## Novo Recurso: Validação

O novo sistema de validação verifica a entrada antes do processamento:

```bash
project validate --input file.yaml
```

Resultando em:
```
✓ Configuração válida
✓ Todos os requisitos atendidos
```
```

**Note**: Code and output are identical, only descriptions are translated.

---

## Synchronization Checklist

When updating documentation:

- [ ] Edit section in README.md (English)
- [ ] Review changes in README.md
- [ ] Edit corresponding section in README-PT.md (Portuguese)
- [ ] Verify both have same structure
- [ ] Check code examples are identical
- [ ] Verify links work in both files
- [ ] Review Portuguese translation quality
- [ ] Commit both files together
- [ ] Check diff shows expected changes

---

## Common Sections Reference

### Features (English)
```markdown
## Features

- Advanced detection system
- Multi-platform support
- Configurable options
```

### Recursos (Portuguese)
```markdown
## Recursos

- Sistema de detecção avançado
- Suporte multi-plataforma
- Opções configuráveis
```

---

## Tools for Comparison

To ensure files stay synchronized:

```bash
# Show differences between files
diff <(sed 's/português/English/g' README-PT.md) README.md

# Count sections to ensure they match
grep "^## " README.md | wc -l
grep "^## " README-PT.md | wc -l

# Check for structure differences
diff <(grep "^[#]" README.md) <(grep "^[#]" README-PT.md)
```

---

## File Naming Conventions

- **README.md** — English version (primary)
- **README-PT.md** — Portuguese version (translation)

Not:
- ❌ README-en.md
- ❌ README-english.md
- ❌ README_PT.md
- ❌ README.pt.md

---

## Documentation Update Priorities

Update in this order:

1. **High Priority** — Update immediately:
   - Installation instructions
   - Usage changes
   - Breaking changes
   - Security issues

2. **Medium Priority** — Update soon:
   - New features
   - Configuration options
   - Examples
   - Troubleshooting

3. **Low Priority** — Update when convenient:
   - Typo fixes
   - Grammar improvements
   - Link updates
   - Minor clarifications

---

## Integration with git-commit-suggest

When committing documentation updates:

```bash
# Analyze changes before committing
# Should show both README files modified

# Suggested commit
git commit -m "docs: add validation feature documentation" \
  -m "Adds documentation for new validation system in both English and Portuguese"

# Or for translation fixes
git commit -m "docs: fix Portuguese translation" \
  -m "Corrects grammar and terminology in README-PT.md"
```

---

## Related Skills

- **readme-bilingual-sync** — This skill (self-reference for new projects)
- **go-project-structure** — For documenting project structure
- **git-commit-suggest** — For committing documentation changes

---

## Changelog

- **v1.0** (2025-03-11) — Initial version
  - Workflow for keeping READMEs synchronized
  - Best practices for bilingual documentation
  - Checklist for updates
  - Translation guidelines
  - Common section examples
