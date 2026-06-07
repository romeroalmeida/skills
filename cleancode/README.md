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
| `references/react.md`          | Componentes/tipagem, composição, hooks, estado local/global, data fetching (TanStack), React 19, Suspense, performance, forms |
| `references/nextjs.md`         | App Router, Server/Client Components, Server Actions, caching, segurança |
| `references/styling.md`        | CSS moderno, design tokens, Tailwind/CSS Modules/CSS-in-JS, temas, responsividade |
| `references/web-performance.md`| Core Web Vitals (LCP/INP/CLS), bundle, code splitting, imagens, carregamento |
| `references/accessibility.md`  | WCAG AA, semântica, teclado/foco, ARIA, formulários, contraste, testes a11y |
| `references/nestjs.md`         | Módulos, DI, DTOs/validação, guards/interceptors/filters, segurança |
| `references/api-design.md`     | HTTP/REST, status codes, Problem Details (RFC 9457), paginação, versionamento, idempotência |
| `references/database.md`       | Modelagem, ORM (Prisma/Drizzle), N+1, índices, transações/isolamento, migrations, pooling |
| `references/security.md`       | OWASP Top 10:2025, authn/authz, injeção/XSS, segredos, headers, dependências |
| `references/node.md`           | Event loop, async/streams, erros, config 12-factor, graceful shutdown, observabilidade |
| `references/testing.md`        | Vitest, Testing Library, Playwright, MSW, testar comportamento     |
| `references/architecture.md`   | SOLID, regra de dependência, ports & adapters, organização por feature |

Cada referência termina com um **checklist acionável** e uma seção **Fontes** com links
para a documentação primária. Fatos "duros" (Core Web Vitals, contraste WCAG, APIs do
React 19, defaults do Next 15) foram verificados na fonte oficial em jun/2026.

Stack-alvo: TypeScript 5.x, React 18/19, Next.js 14/15 (App Router), NestJS 10/11,
Node 20+ LTS, Prisma/Drizzle, Vitest/Jest, Playwright.

## Instalação

Esta skill faz parte do repositório [`skills`](https://github.com/romeroalmeida/skills).
Clone o repositório inteiro na pasta de skills do Claude Code (veja o
[README raiz](../README.md)); a pasta `cleancode/` é detectada automaticamente.

A skill é acionada quando você trabalha com código TS/JS (React, Next, Nest) ou pede
boas práticas / code review / refatoração.

## Como usar

1. Identifique a stack da tarefa.
2. Leia a referência correspondente em `references/`.
3. Aplique as regras e rode o checklist antes de finalizar.

## Manutenção

As referências evoluem com as stacks. PRs/commits bem-vindos ao atualizar versões,
defaults (ex.: cache do Next) e novas práticas.
