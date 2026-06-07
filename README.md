# skills

Coleção de [Claude Code Skills](https://docs.claude.com/en/docs/claude-code/skills)
pessoais. Cada subpasta é **uma skill** (uma pasta com um `SKILL.md` na raiz dela).

Este repositório mapeia diretamente a pasta de skills do Claude Code
(`~/.claude/skills`), então o que está versionado aqui é detectado automaticamente.

## Skills disponíveis

| Skill                      | O que faz                                                          |
| -------------------------- | ----------------------------------------------------------------- |
| [`cleancode`](./cleancode) | Padrões de Clean Code TS/JS — React, Next, Nest, testes, arquitetura |

## Instalação

Clone o repositório **na própria pasta de skills** do Claude Code:

```bash
# se a pasta ainda não existir
git clone git@github.com:romeroalmeida/skills.git ~/.claude/skills

# se ~/.claude/skills já existir (anexa o repo ao conteúdo atual)
cd ~/.claude/skills
git init && git remote add origin git@github.com:romeroalmeida/skills.git
git fetch && git checkout -t origin/main
```

Windows (PowerShell): troque `~/.claude/skills` por `$HOME\.claude\skills`.

O Claude Code passa a detectar todas as skills automaticamente.

## Como adicionar uma nova skill

Resumo: crie uma pasta na raiz (`minha-skill/`) com um `SKILL.md` (frontmatter `name` +
`description` + instruções), adicione a linha na tabela acima e dê `git push`.

📖 **Passo a passo completo, formato do `SKILL.md`, boas práticas de `description` e
validação:** veja **[CONTRIBUTING.md](./CONTRIBUTING.md)**.

> O nome da pasta e o `name` do frontmatter devem coincidir (kebab-case). A `description`
> é o que faz a skill ser sugerida na hora certa — capriche nos gatilhos.
