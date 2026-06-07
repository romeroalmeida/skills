# TypeScript / JavaScript — Clean Code

Base para **todo** código TS/JS, front ou back. Alvo: TypeScript 5.x, Node 20+, ESM.

## 1. Configuração estrita (inegociável)

```jsonc
// tsconfig.json — mínimo saudável
{
  "compilerOptions": {
    "strict": true,                       // liga tudo: noImplicitAny, strictNullChecks...
    "noUncheckedIndexedAccess": true,     // arr[i] vira T | undefined (evita bugs)
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "verbatimModuleSyntax": true,         // import type explícito
    "moduleResolution": "bundler",        // ou "node16"/"nodenext" em libs
    "target": "ES2022"
  }
}
```

Se herdar um projeto sem `strict`, ligue incrementalmente — não introduza código novo
fora do modo estrito.

## 2. Tipos: modele o domínio, não `any`

- **`any` é proibido.** Use `unknown` na entrada e faça narrowing. `any` desliga o
  compilador e contamina tudo que toca.

```ts
// ❌ perde toda a segurança
function parse(input: any) { return input.value.toUpperCase(); }

// ✅ unknown obriga validar antes de usar
function parse(input: unknown): string {
  if (typeof input === "object" && input !== null && "value" in input
      && typeof input.value === "string") {
    return input.value.toUpperCase();
  }
  throw new TypeError("input.value deve ser string");
}
```

- **Discriminated unions** para estados mutuamente exclusivos — eliminam estados
  impossíveis (não use vários booleanos soltos).

```ts
// ❌ permite combinações inválidas (loading && error?)
type State = { loading: boolean; error?: string; data?: User };

// ✅ um discriminante 'status' torna estados impossíveis irrepresentáveis
type State =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; error: string };
```

- **`as const`** para literais e tuplas; **`satisfies`** para validar sem alargar o tipo.

```ts
const ROLES = ["admin", "editor", "viewer"] as const;
type Role = (typeof ROLES)[number]; // "admin" | "editor" | "viewer"

const config = {
  port: 3000,
  env: "production",
} satisfies Record<string, string | number>; // valida E mantém tipos literais
```

- **Prefira `type` para uniões/composição**; `interface` para contratos extensíveis de
  objeto. Seja consistente no projeto.
- **`readonly` e `Readonly<T>`** para imutabilidade; tuplas `readonly [a, b]`.
- **Branded types** para evitar misturar IDs do mesmo tipo primitivo:

```ts
type UserId = string & { readonly __brand: "UserId" };
type OrderId = string & { readonly __brand: "OrderId" };
// função que espera UserId não aceita OrderId por engano
```

- **`enum` evite** (gera runtime e tem pegadinhas). Use union de strings + `as const`.
- **Type guards** nomeados (`is`) e funções de **assertion** para narrowing reusável.
- **Evite type assertions (`as`)**: são uma promessa não verificada. Quando inevitável,
  prefira validar. **Nunca** `as unknown as T` para "calar" o compilador.

## 3. Null safety

- `strictNullChecks` ligado. Trate `null`/`undefined` explicitamente.
- `?.` (optional chaining) e `??` (nullish coalescing — não `||`, que pega `0`/`""`).
- **Non-null `!` é um code smell.** Cada uso é "confie em mim"; prefira guard/early return.

```ts
// ❌ || engole valores falsy válidos
const port = process.env.PORT || 3000;   // PORT="0" vira 3000
// ✅
const port = process.env.PORT ?? "3000";
```

## 4. Funções

- Pequenas, um nível de abstração, **early return** em vez de aninhar.
- ≤ 3 parâmetros; mais que isso, agrupe num objeto de opções nomeadas.
- **Sem parâmetros booleanos** que viram dois comportamentos — divida em duas funções.
- Tipos explícitos no retorno de funções públicas/exportadas (documenta e evita
  inferência acidental que vaza implementação).
- Funções **puras** por padrão; isole I/O nas bordas.

```ts
// ❌ flag booleana esconde dois comportamentos
function render(user: User, isAdmin: boolean) { /* ... */ }
// ✅
function renderUserView(user: User) {}
function renderAdminView(user: User) {}
```

## 5. Erros

- Lance **`Error` (ou subclasse)**, nunca strings/objetos crus.
- Crie erros de domínio com contexto:

```ts
class NotFoundError extends Error {
  constructor(resource: string, id: string) {
    super(`${resource} ${id} não encontrado`);
    this.name = "NotFoundError";
  }
}
```

- `catch (e)` → `e` é `unknown`. Faça narrowing (`e instanceof Error`).
- **Nunca** `catch` vazio. Se decidir ignorar, comente o porquê explicitamente.
- Para erros esperados (validação, "não encontrado"), considere o **Result pattern**
  em vez de exceptions, deixando exceptions para o inesperado:

```ts
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };
```

## 6. Assíncrono

- `async/await` em vez de cadeias `.then`. Sempre `await` ou retorne a promise —
  **floating promises** escondem erros (ligue a regra `no-floating-promises`).
- Paralelize o que é independente com `Promise.all` / `Promise.allSettled`.

```ts
// ❌ sequencial sem necessidade
const user = await getUser(id);
const orders = await getOrders(id);
// ✅ paralelo (independentes)
const [user, orders] = await Promise.all([getUser(id), getOrders(id)]);
```

- Não misture `async` com callbacks; não use `forEach` com `async` (não espera) —
  use `for...of` com `await`, ou `Promise.all(map(...))`.
- Propague `AbortSignal` em operações canceláveis (fetch, timeouts).

## 7. Módulos e organização

- **ESM**, named exports por padrão (melhor para refactor/tree-shaking). `default`
  export só quando a convenção do framework pede (ex.: páginas Next).
- `import type { ... }` para imports apenas de tipo.
- Cuidado com **barrel files** (`index.ts` reexportando tudo): facilitam ciclos de
  import e prejudicam tree-shaking em projetos grandes.
- Um arquivo = uma responsabilidade coesa. Ordene: imports → tipos → constantes →
  implementação.

## 8. Coleções e dados

- Métodos de array declarativos (`map`/`filter`/`reduce`/`some`/`every`) sobre loops
  imperativos quando aumentam a clareza; `for...of` quando há efeito/`await`.
- Não mute arrays/objetos recebidos — gere novos (`[...arr]`, `{...obj}`, `structuredClone`).
- `Map`/`Set` para coleções com chave/unicidade em vez de objetos como dicionário.

## 9. Toolchain (padrão moderno)

- **TypeScript** estrito + **ESLint** (typescript-eslint) + **Prettier** (ou Biome,
  que faz lint+format rápido). Lint e format no CI e em pre-commit.
- Regras-chave: `no-floating-promises`, `no-explicit-any`, `no-unused-vars`,
  `consistent-type-imports`, `no-non-null-assertion`.
- Valide **env vars** com zod num módulo central, falhando no boot se faltarem.

## Checklist TS/JS

- [ ] `strict` + `noUncheckedIndexedAccess` ligados; zero `any`.
- [ ] Estados modelados com discriminated unions; sem estados impossíveis.
- [ ] `as`/`!`/`@ts-ignore` ausentes ou justificados.
- [ ] `??`/`?.` no lugar de `||` para nullish; null/undefined tratados.
- [ ] Funções pequenas, ≤3 params, sem flags booleanas, retorno tipado se público.
- [ ] Erros são `Error`, com contexto; nenhum `catch` vazio.
- [ ] Sem floating promises; paralelismo com `Promise.all`.
- [ ] Imutabilidade preservada (sem mutar args); `import type` onde cabe.
- [ ] Env e dados externos validados em runtime.

## Fontes

- TypeScript — Handbook: https://www.typescriptlang.org/docs/handbook/intro.html
- TypeScript — Referência do tsconfig (`strict`, `noUncheckedIndexedAccess`, etc.):
  https://www.typescriptlang.org/tsconfig
- typescript-eslint — regras (`no-explicit-any`, `no-floating-promises`,
  `consistent-type-imports`): https://typescript-eslint.io/rules/
- Matt Pocock — Total TypeScript (padrões modernos, `satisfies`, branded types):
  https://www.totaltypescript.com
- MDN — JavaScript (async/await, Promise, módulos ESM):
  https://developer.mozilla.org/en-US/docs/Web/JavaScript
