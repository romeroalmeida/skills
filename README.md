# cleancode — Claude Code Skill

Padrões de **Clean Code modernos para JavaScript/TypeScript**, frontend e backend, em
formato de [Claude Code Skill](https://docs.claude.com/en/docs/claude-code/skills).
Pensado para ser **consultado antes de qualquer tarefa de código** (escrever, revisar ou
refatorar), garantindo boas práticas e evitando erros.

## O que cobre

| Arquivo                        | Conteúdo                                                            |
| ------------------------------ | ------------------------------------------------------------------ |
| `SKILL.md`                     | Roteador + princípios universais + checklist universal             |
| `references/typescript.md`     | TS estrito, sem `any`, unions, `satisfies`, erros, async, módulos  |
| `references/react.md`          | React 19, "você não precisa de useEffect", estado, performance, a11y |
| `references/nextjs.md`         | App Router, Server/Client Components, Server Actions, caching, segurança |
| `references/nestjs.md`         | Módulos, DI, DTOs/validação, guards/interceptors/filters, segurança |
| `references/testing.md`        | Vitest, Testing Library, Playwright, MSW, testar comportamento     |
| `references/architecture.md`   | SOLID, regra de dependência, ports & adapters, organização por feature |

Cada referência termina com um **checklist acionável** para rodar antes de entregar.

Stack-alvo: TypeScript 5.x, React 18/19, Next.js 14/15 (App Router), NestJS 10/11,
Node 20+, Vitest/Jest, Playwright.

## Instalação

Clone dentro da pasta de skills do Claude Code:

```bash
# Skill pessoal (vale para todos os projetos)
git clone git@github.com:romeroalmeida/skills.git ~/.claude/skills/cleancode

# ou, para uma skill de projeto específico
git clone git@github.com:romeroalmeida/skills.git .claude/skills/cleancode
```

No Windows (PowerShell): `git clone ... $HOME\.claude\skills\cleancode`.

O Claude Code detecta a skill automaticamente. Ela é acionada quando você trabalha com
código TS/JS (React, Next, Nest) ou pede boas práticas/code review/refatoração.

## Como usar

1. Identifique a stack da tarefa.
2. Leia a referência correspondente em `references/`.
3. Aplique as regras e rode o checklist antes de finalizar.

## Manutenção

As referências evoluem com as stacks. PRs/commits bem-vindos ao atualizar versões,
defaults (ex.: cache do Next) e novas práticas.
