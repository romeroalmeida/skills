---
name: cleancode
description: >-
  Padrões de Clean Code modernos para JavaScript/TypeScript — frontend e backend.
  CONSULTE SEMPRE antes de escrever, revisar ou refatorar código em TS/JS, React,
  Next.js (App Router), NestJS ou Node. Cobre tipagem estrita, arquitetura SOLID,
  hooks, Server/Client Components, DTOs/validação, testes e segurança. Use quando o
  usuário pedir boas práticas, code review, refatoração, ou iniciar qualquer feature.
---

# Clean Code — JavaScript / TypeScript (Front & Back)

Guia operacional para produzir código **correto, legível e seguro** em TS/JS moderno.
Não é teoria: é um conjunto de regras verificáveis. Antes de entregar qualquer tarefa,
rode o **Checklist universal** abaixo e o checklist da referência da stack envolvida.

## Como usar esta skill

1. Identifique a stack da tarefa e **leia o(s) arquivo(s) de referência** relevantes:

   | Tarefa envolve…                          | Leia                              |
   | ---------------------------------------- | --------------------------------- |
   | Qualquer código TS/JS                    | `references/typescript.md`        |
   | Componentes, hooks, UI                   | `references/react.md`             |
   | App Router, RSC, server actions, SSR     | `references/nextjs.md`            |
   | API, controllers, DI, DTOs               | `references/nestjs.md`            |
   | Testes (unit/integration/e2e)            | `references/testing.md`           |
   | Estrutura de pastas, camadas, SOLID      | `references/architecture.md`      |

2. Aplique as regras. Quando houver conflito, a ordem de prioridade é:
   **Correção > Segurança > Legibilidade > Consistência > Performance**.
   (Não otimize performance antes de medir.)

3. Antes de finalizar, valide com os checklists.

## Princípios universais (valem para todo o código)

- **Clareza sobre esperteza.** Código é lido muito mais do que escrito. Prefira o
  explícito ao clever. Se precisa de um comentário pra explicar *o quê*, reescreva;
  comentários explicam *por quê*, não *o quê*.
- **Nomes revelam intenção.** `elapsedDays` e não `d`; `isActive` e não `flag`.
  Funções são verbos (`calculateTotal`), variáveis/booleanos são substantivos/predicados
  (`hasPermission`). Sem abreviações obscuras, sem `data`/`info`/`temp`/`obj` genéricos.
- **Funções pequenas e com um propósito.** Uma função faz uma coisa, num nível de
  abstração. Sinal de alerta: > ~20 linhas, > 3 parâmetros, mais de 1 nível de `if`
  aninhado, ou nome com "and"/"e". Use **early return** em vez de aninhar.
- **Imutabilidade por padrão.** `const`, `readonly`, dados imutáveis. Evite mutar
  argumentos e estado compartilhado. Mutação local controlada é ok quando mais clara.
- **Sem números/strings mágicos.** Extraia para constantes nomeadas (`const MAX_RETRIES = 3`).
- **DRY com equilíbrio.** Elimine duplicação *de conhecimento*, não coincidências.
  Duas linhas iguais por acaso não justificam uma abstração errada (regra do três).
  Prefira **YAGNI**: não construa para um futuro hipotético.
- **Trate o erro, não o ignore.** Nunca `catch` vazio, nunca engula exceções. Falhe
  alto e cedo (fail-fast) com mensagens acionáveis. Não use exceções para fluxo normal.
- **Sem efeitos colaterais escondidos.** Uma função cujo nome sugere cálculo não deve
  gravar em disco/rede. Separe decisão (puro) de efeito (impuro).
- **Fronteiras tipadas.** Todo dado que entra no sistema (HTTP, env, JSON, formulário)
  é `unknown` até ser **validado em runtime** (zod/class-validator). Tipo estático não
  protege contra dados externos.
- **Segurança não é opcional.** Nunca logue segredos/PII; nunca interpole input em
  SQL/HTML/shell; valide e sanitize tudo na borda; segredos só no servidor.
- **Deixe o campo mais limpo do que encontrou** — mas em mudanças focadas; não misture
  refactor amplo com correção de bug no mesmo commit.

## Checklist universal (rode antes de entregar)

- [ ] `tsconfig` em `strict: true`; **zero `any`** novo (use `unknown` + narrowing).
- [ ] Sem `// @ts-ignore`/`!` (non-null) sem justificativa em comentário.
- [ ] Nomes claros; funções pequenas; sem aninhamento profundo (early return).
- [ ] Sem código morto, `console.log` esquecido, imports não usados, TODO órfão.
- [ ] Erros tratados de forma significativa; nada de `catch {}` silencioso.
- [ ] Dados externos validados em runtime na borda.
- [ ] Sem segredos no código/cliente; nada sensível logado.
- [ ] `Promise`s sempre `await`/encadeadas (sem floating promises); `Promise.all`
      para paralelismo independente.
- [ ] Lint e type-check passam; testes do que mudou passam.
- [ ] A mudança é focada e o diff é mínimo para o objetivo.

> Detalhes e exemplos ✅/❌ de cada ponto estão nos arquivos de `references/`.
