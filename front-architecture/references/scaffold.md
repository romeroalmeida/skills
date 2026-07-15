# Scaffold — arquitetura feature-based

Carregue este arquivo **só na hora de montar**, depois das perguntas de scoping.
Crie **apenas** as pastas das features confirmadas e **apenas** as camadas escolhidas
(sem `state/` se não usa Zustand, sem `api/queries/` se não usa React Query, etc.).

## 1. Criar o projeto (se ainda não existe)

```bash
npm create vite@latest <nome-do-app> -- --template react-ts
cd <nome-do-app>
npm install
```

Dependências conforme o questionário (instale **só as confirmadas**):

```bash
npm install react-router-dom              # rotas
npm install zustand                       # estado global
npm install @tanstack/react-query axios   # data-fetching
npm install react-hook-form zod @hookform/resolvers   # forms + validação
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom  # unit
npm install -D cypress                    # e2e
npm install -D eslint prettier            # lint/format (se ainda não vier do template)
```

### Tailwind (se confirmado)

```bash
npm install -D tailwindcss @tailwindcss/vite
```

`vite.config.ts` → adicione o plugin `tailwindcss()` ao array `plugins`.
`src/shared/styles/index.css` → `@import "tailwindcss";` e importe esse arquivo no `main.tsx`.

### UI kit sobre Tailwind (se confirmado)

- **shadcn/ui**: `npx shadcn@latest init` e depois `npx shadcn@latest add button` (por componente).
  Aponte os componentes para `src/shared/components/ui` no `components.json`.
- **Radix / Headless UI**: `npm install @radix-ui/react-*` ou `npm install @headlessui/react` — use dentro de `shared/components/`.

## 2. Estrutura de pastas

```bash
# base
mkdir -p src/app/{routes,providers,layout} src/assets src/shared/{components,hooks,utils,styles,types} src/api/queries src/state src/tests
# uma pasta por feature confirmada (repita para cada):
mkdir -p src/features/<feature>/{components,hooks,pages,services,utils}
```

## 3. Path aliases

`vite.config.ts`:

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'node:path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: { '@': path.resolve(__dirname, './src') },
  },
})
```

`tsconfig.json` (em `compilerOptions`):

```json
{
  "baseUrl": ".",
  "paths": { "@/*": ["src/*"] }
}
```

Uso: `import { Button } from '@/shared/components'`.

## 4. Arquivos-base (templates)

### `src/api/apiClient.ts`

```ts
import axios from 'axios'

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  headers: { 'Content-Type': 'application/json' },
})
```

### `src/api/endpoints.ts`

```ts
export const endpoints = {
  auth: {
    login: '/auth/login',
    register: '/auth/register',
  },
} as const
```

### `src/app/providers/QueryProvider.tsx` (só se usar React Query)

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { type ReactNode, useState } from 'react'

export function QueryProvider({ children }: { children: ReactNode }) {
  const [client] = useState(() => new QueryClient())
  return <QueryClientProvider client={client}>{children}</QueryClientProvider>
}
```

### `src/state/useSessionStore.ts` (só se usar Zustand)

```ts
import { create } from 'zustand'

interface SessionState {
  user: { id: string; name: string } | null
  setUser: (user: SessionState['user']) => void
  clear: () => void
}

export const useSessionStore = create<SessionState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  clear: () => set({ user: null }),
}))
```

### `src/app/routes/index.tsx` (só se usar React Router)

```tsx
import { createBrowserRouter } from 'react-router-dom'

export const router = createBrowserRouter([
  { path: '/', element: <div>Home</div> },
])
```

### `src/app/App.tsx`

```tsx
import { RouterProvider } from 'react-router-dom'
import { QueryProvider } from './providers/QueryProvider'
import { router } from './routes'

export function App() {
  return (
    <QueryProvider>
      <RouterProvider router={router} />
    </QueryProvider>
  )
}
```

### `src/main.tsx`

```tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { App } from './app/App'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

## 5. Exemplo de feature (`auth`) — o barrel é o contrato público

### `src/features/auth/services/authService.ts`

```ts
import { apiClient } from '@/api/apiClient'
import { endpoints } from '@/api/endpoints'

export async function login(email: string, password: string) {
  const { data } = await apiClient.post(endpoints.auth.login, { email, password })
  return data
}
```

### `src/features/auth/hooks/useLogin.ts` (React Query)

```ts
import { useMutation } from '@tanstack/react-query'
import { login } from '../services/authService'

export function useLogin() {
  return useMutation({ mutationFn: (v: { email: string; password: string }) => login(v.email, v.password) })
}
```

### `src/features/auth/index.ts` — barrel: expõe SÓ o público

```ts
export { LoginPage } from './pages/LoginPage'
export { useLogin } from './hooks/useLogin'
// componentes/services internos NÃO são reexportados
```

Regra: fora da feature, importe sempre de `@/features/auth` — nunca de caminhos internos.

## 6. Verificação rápida

```bash
npm run dev        # sobe e renderiza sem erro
npx tsc --noEmit   # tipos ok, aliases resolvem
```

Checklist final (do SKILL.md): sem import cruzado entre features, `shared/` sem
domínio, barrel por feature, aliases no lugar de `../../../`.
