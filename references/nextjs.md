# Next.js — Clean Code (App Router, Next 14/15)

Foco no **App Router** (`app/`). Combine com `react.md` e `typescript.md`.

## 1. Server Components por padrão

- Componentes em `app/` são **Server Components** por padrão. Mantenha assim: eles não
  vão pro bundle do cliente, acessam back-end direto e fazem fetch sem expor segredos.
- `"use client"` **só quando precisa** de interatividade (estado, efeitos, event handlers,
  APIs de browser). Marque a **folha** da árvore, não o topo — empurre o `"use client"`
  para o menor componente possível.
- **Nunca** importe código sensível (DB, chaves) dentro de um Client Component. Use o
  pacote `server-only` para garantir em build:

```ts
import "server-only"; // erro de build se vazar pro cliente
```

- Passe dados de Server → Client via props **serializáveis** (sem funções/classes).

## 2. Data fetching e caching

- Busque dados no **Server Component** com `async/await` direto — sem `useEffect`.

```tsx
// app/users/page.tsx (Server Component)
export default async function UsersPage() {
  const users = await getUsers(); // roda no servidor
  return <UserList users={users} />;
}
```

- Controle de cache do `fetch` (Next estende o fetch):
  - `fetch(url, { cache: "force-cache" })` — estático (padrão varia por versão; seja explícito).
  - `fetch(url, { next: { revalidate: 60 } })` — ISR, revalida a cada 60s.
  - `fetch(url, { cache: "no-store" })` — sempre dinâmico (dados por request/usuário).
- Use **tags** + `revalidateTag`/`revalidatePath` para invalidação sob demanda após mutações.
- **Paralelize** requests independentes (`Promise.all`) e evite waterfalls. Use `Suspense`
  para streaming de partes lentas sem travar a página.
- ⚠️ Next 15 mudou defaults de cache (menos agressivo, `fetch` não-cacheado por padrão e
  `cookies()`/`headers()`/`params` assíncronos). **Confirme a versão** antes de assumir cache.

## 3. Mutações: Server Actions

- Mutações via **Server Actions** (`"use server"`), não API routes manuais para o próprio app.

```tsx
"use server";
import { z } from "zod";

const Schema = z.object({ title: z.string().min(1).max(120) });

export async function createPost(formData: FormData) {
  // 1. autentique/autorize SEMPRE — actions são endpoints públicos
  const user = await requireUser();
  // 2. valide a entrada (nunca confie no cliente)
  const data = Schema.parse({ title: formData.get("title") });
  // 3. execute e revalide
  await db.post.create({ data: { ...data, authorId: user.id } });
  revalidatePath("/posts");
}
```

- Regras de ouro das actions: **toda action é um endpoint público** → autentique,
  autorize e valide a entrada dentro dela. Nunca passe segredos/dados de autorização
  pelo cliente.

## 4. Route Handlers (`app/api/.../route.ts`)

- Use para webhooks, APIs públicas/externas, ou integrações — não para a comunicação
  interna do próprio app (prefira Server Components/Actions).
- Valide entrada, trate métodos, devolva status corretos, sem vazar stack traces.

## 5. Arquivos especiais de rota

- `loading.tsx` (Suspense fallback), `error.tsx` (Error Boundary — **Client Component**),
  `not-found.tsx`, `layout.tsx` (estado/markup compartilhado, não re-renderiza ao navegar).
- Trate estados de loading/erro em **todo** segmento que faz fetch.

## 6. Metadata e SEO

- API de `metadata`/`generateMetadata` (estática ou dinâmica), não `<head>` manual.
- Use `generateStaticParams` para rotas dinâmicas estáticas.

## 7. Performance e assets

- `next/image` (otimização, lazy, `sizes`), `next/font` (sem layout shift, self-host).
- `next/dynamic` para code-splitting de componentes pesados/abaixo da dobra.
- Minimize JS no cliente mantendo lógica no servidor; cuidado com libs grandes em
  Client Components.

## 8. Configuração e segurança

- **Env**: `NEXT_PUBLIC_*` vai pro browser (só dados não-sensíveis). Todo o resto é
  server-only. Valide envs com zod num módulo central.
- Nunca exponha chaves de API/DB no cliente; nunca confie em dados vindos do cliente.
- Headers de segurança (CSP etc.) via `next.config`/middleware.
- `middleware.ts` para auth/redirect leve na borda — mantenha rápido e sem lógica pesada.

## Checklist Next.js

- [ ] Server Components por padrão; `"use client"` só nas folhas interativas.
- [ ] Sem fetch via `useEffect`; dados buscados no servidor com `async`.
- [ ] Estratégia de cache **explícita** por fetch; invalidação com tag/path após mutação.
- [ ] Mutações em Server Actions com auth + validação (zod) **dentro** da action.
- [ ] `loading`/`error`/`not-found` cobrindo segmentos com fetch; `Suspense` no que é lento.
- [ ] `server-only` protegendo código sensível; nada sensível em `NEXT_PUBLIC_*`.
- [ ] `next/image` e `next/font`; metadata pela API.
- [ ] Defaults de cache conferidos para a versão do Next em uso.
