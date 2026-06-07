# Como criar e adicionar skills

Guia para criar uma nova skill neste repositório (que mapeia `~/.claude/skills`).
Se você só quer o resumo, veja o [TL;DR](#tldr) no final.

## 1. O que é uma skill

Uma skill é uma **pasta** com um arquivo **`SKILL.md`** na raiz dela. O `SKILL.md` tem
um cabeçalho YAML (_frontmatter_) com `name` e `description`, seguido do conteúdo em
Markdown. O Claude Code lê a `description` de todas as skills e **carrega o conteúdo
sob demanda**, quando a tarefa combina com a descrição.

```
minha-skill/
├── SKILL.md            ← obrigatório (frontmatter + instruções)
└── references/         ← opcional (arquivos carregados sob demanda)
    └── exemplo.md
```

## 2. Estrutura de uma skill no repo

Cada skill é **uma pasta na raiz** deste repositório:

```
skills/                 ← raiz do repo (= ~/.claude/skills)
├── cleancode/
│   └── SKILL.md
└── minha-skill/        ← sua nova skill
    └── SKILL.md
```

> **Regra:** o nome da pasta e o campo `name` do frontmatter devem ser **iguais** e em
> **kebab-case** (`minha-skill`, não `MinhaSkill` nem `minha_skill`).

## 3. O frontmatter (cabeçalho YAML)

Só dois campos são obrigatórios: `name` e `description`.

```markdown
---
name: minha-skill
description: >-
  Descreva O QUE a skill faz e, principalmente, QUANDO usá-la. Esta frase é o
  único sinal que o Claude usa para decidir acionar a skill — inclua os gatilhos
  (tecnologias, verbos, situações). Ex.: "Use ao escrever queries Prisma/SQL...".
---
```

Regras e dicas:

- **`name`**: minúsculas, kebab-case, único no repo, igual ao nome da pasta. Máx. ~64 chars.
- **`description`**: o item mais importante. É o que faz a skill ser sugerida na hora certa.
  - Comece por quando/para quê: _"Use quando…"_, _"Consulte antes de…"_.
  - Cite **gatilhos concretos**: nomes de libs/frameworks, tipos de tarefa, verbos
    (criar, revisar, refatorar, depurar, otimizar).
  - Escreva em 3ª pessoa, de forma objetiva. Pode usar `>-` para quebrar em várias linhas.
  - Evite descrição genérica demais ("ajuda com código") — não aciona bem e atrapalha
    outras skills.
- Campos opcionais comuns: `metadata` (ex.: `version`), `allowed-tools` (restringe
  ferramentas). Não invente campos que o runtime não entende.

### Exemplos de boas `description`

✅ `Use ao escrever ou revisar queries com Prisma/Drizzle — modelagem de schema, migrations, índices, N+1, transações.`
✅ `Padrões de mensagens de commit e fluxo Git (Conventional Commits). Consulte antes de commitar ou abrir PR.`
❌ `Skill de banco de dados.` (vago, sem gatilho, não aciona)

## 4. O corpo do SKILL.md

Depois do frontmatter, escreva instruções **acionáveis** — não teoria. Boa estrutura:

1. **Título + 1–2 linhas** dizendo o escopo.
2. **Como usar / quando ler o quê** (se houver `references/`).
3. **Regras** com exemplos curtos `✅ certo` / `❌ errado`.
4. **Checklist** final para validar antes de entregar.

Mantenha o `SKILL.md` **enxuto** (idealmente < ~300 linhas). Conteúdo extenso vai para
`references/` e é citado no corpo — assim só carrega quando necessário
(_progressive disclosure_).

## 5. Arquivos de apoio (`references/`)

Para skills grandes, divida por assunto e aponte no `SKILL.md`:

```markdown
| Tarefa envolve… | Leia |
| --------------- | ---- |
| Migrations      | `references/migrations.md` |
| Performance     | `references/performance.md` |
```

Use caminhos **relativos à pasta da skill**. Evite duplicar conteúdo entre referências.

## 6. Validar a skill

1. **Frontmatter válido**: YAML correto, `name` = nome da pasta, `description` preenchida.
2. **Detecção**: abra o Claude Code e confirme que a skill aparece/é sugerida. Você pode
   invocá-la pelo nome (ex.: `/minha-skill` ou pedindo a tarefa que ela cobre).
3. **Acionamento**: descreva uma tarefa que deveria acioná-la e veja se ela é considerada;
   se não, refine a `description` com gatilhos melhores.
4. **Markdown**: links relativos funcionando, exemplos de código corretos.

## 7. Subir para o repositório

```bash
cd ~/.claude/skills              # = raiz do repo (no Windows: $HOME\.claude\skills)

# 1. crie a pasta e o SKILL.md
mkdir minha-skill
# (edite minha-skill/SKILL.md)

# 2. registre na tabela do README.md da raiz

# 3. versione
git add minha-skill README.md
git commit -m "feat: skill minha-skill (<assunto>)"
git push
```

> Dica: faça um commit por skill (ou por mudança coesa), com mensagem no estilo
> Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`).

## 8. Boas práticas (resumo)

- **Uma skill = um domínio coeso.** Não crie uma skill "faz-tudo".
- **`description` é marketing honesto:** diga exatamente quando acionar.
- **Instruções acionáveis**, com exemplos e checklist — não um ensaio.
- **Enxuto no `SKILL.md`**, detalhe em `references/`.
- **Atualize** quando a tecnologia mudar (versões, defaults, novas práticas).
- **Pasta = `name` = kebab-case.** Sempre.

## TL;DR

1. `mkdir ~/.claude/skills/minha-skill`
2. Crie `minha-skill/SKILL.md` com frontmatter (`name`, `description`) + instruções + checklist.
3. (Opcional) Coloque material extenso em `minha-skill/references/`.
4. Adicione a linha na tabela do `README.md`.
5. `git add . && git commit -m "feat: skill minha-skill" && git push`.

Pronto — o Claude Code detecta automaticamente.
