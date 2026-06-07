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

1. Crie uma pasta na raiz com o nome da skill (kebab-case): `minha-skill/`.
2. Dentro, crie um `SKILL.md` com frontmatter:

   ```markdown
   ---
   name: minha-skill
   description: >-
     Descrição clara de QUANDO usar a skill (é isso que decide o acionamento).
   ---

   # Minha Skill
   ...
   ```

3. (Opcional) Coloque arquivos de apoio em `minha-skill/references/` e referencie-os
   no `SKILL.md` (carregamento sob demanda — _progressive disclosure_).
4. Adicione a skill na tabela acima, `git add`, `commit` e `push`.

> O nome da pasta e o `name` do frontmatter devem coincidir (kebab-case). A `description`
> é o que faz a skill ser sugerida na hora certa — capriche nos gatilhos.
