# Claude Code — Estudos de Skills

Repositório de estudo prático sobre **Skills no Claude Code**: criar, testar, organizar e entender seu funcionamento.

> **Ensine uma vez. Reutilize sempre que a tarefa aparecer.**

Exemplos de uso: code review, pull request, commit messages, documentação, debugging, geração de testes, padrões de equipe.

---

## O que é uma Skill

Um conjunto reutilizável de instruções que ensina o Claude Code a executar uma tarefa específica.

## Estrutura

Mínima:
```text
skill-name/
└── SKILL.md
```

Com arquivos auxiliares (recursos usados pela Skill principal):
```text
skill-name/
├── SKILL.md
├── checklist.md
├── examples.md
└── templates/
    └── template.md
```

## `SKILL.md`

Duas partes: **frontmatter** e **instruções**.

```markdown
---
name: code-review
description: Reviews code for bugs, security issues, tests, quality, and maintainability.
---

# Code Review

When reviewing code:
1. Check for bugs.
2. Check security issues.
3. Check tests.
4. Check code quality.
5. Check maintainability.

Do not modify the code unless explicitly requested.
```

- `name`: identifica a Skill.
- `description`: usada pelo Claude para fazer o **matching** entre o pedido do usuário e as Skills disponíveis (por intenção/significado, não por palavras exatas).

## Matching e carregamento sob demanda

1. Claude Code inicia e descobre as Skills, carregando apenas `name` + `description`.
2. O usuário faz um pedido.
3. Claude compara semanticamente o pedido com as descriptions.
4. Se houver match, pede confirmação e só então carrega o conteúdo completo da Skill para executar.

Isso evita carregar todas as Skills por completo em toda conversa.

## Personal × Project Skills

| Tipo | Localização | Escopo |
|---|---|---|
| Personal Skill | `~/.claude/skills/` | Todos os projetos do usuário |
| Project Skill | `.claude/skills/` (dentro do projeto) | Projeto atual (versionável com Git) |

## Prioridade quando há Skills com o mesmo nome

```text
Enterprise > Personal > Project > Plugins
```

## Ciclo de vida

**Criar:**
```bash
mkdir -p ~/.claude/skills/pr-description
touch ~/.claude/skills/pr-description/SKILL.md
```
ou
```bash
mkdir ~/.claude
mkdir .claude/skills
```

**Testar:** reiniciar o Claude Code → fazer um pedido relacionado → verificar se a Skill é identificada e aplicada.

**Atualizar:** editar o `SKILL.md` → reiniciar o Claude Code.

**Remover:** apagar o diretório da Skill → reiniciar o Claude Code.
```bash
rm -rf ~/.claude/skills/pr-description
```

## Prioridade quando há Skills com o mesmo nome

1. Empresarial — configurações gerenciadas, prioridade máxima
2. Pessoal — ~/.claude/skills
3. Projeto — .claude/skills (dentro de um repositório)
4. Plugins — plugins instalados, prioridade mais baixa

## CLAUDE.md × Skills × Slash Commands

| Recurso | Finalidade | Ativação |
|---|---|---|
| `CLAUDE.md` | Regras gerais | Automática |
| Skill | Tarefa específica | Automática, quando relevante |
| Slash Command | Comando específico | Manual |

## Pontos-chave

- Skills podem ser acionadas automaticamente, sem precisar de slash command.
- Arquivos auxiliares (checklist, examples, templates) — fazem parte de uma única Skill.
- A `description` é decisiva para o matching correto.
- Personal (`~/.claude/skills/`) e Project (`.claude/skills/`) são escopos diferentes.

---

## Checklist de estudo

- [ ] Criar uma Skill do zero (`SKILL.md`, frontmatter, `name`, `description`)
- [ ] Adicionar arquivos auxiliares
- [ ] Testar a Skill e observar o semantic matching
- [ ] Diferenciar Personal Skill de Project Skill
- [ ] Memorizar a hierarquia de prioridade
- [ ] Atualizar e remover uma Skill

**Objetivo final:** ser capaz de criar uma Skill do zero, organizar seus arquivos, explicar como o Claude Code a descobre/seleciona, testá-la e entender como conflitos entre Skills são resolvidos.
