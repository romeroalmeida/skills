---
name: front-architecture
description: >-
  Use ao iniciar, estruturar ou revisar a organização de pastas de um projeto
  front-end React (Vite). Define a arquitetura feature-based padrão do Romero,
  faz perguntas de scoping antes de começar e, sob confirmação, monta a estrutura
  de pastas real. Gatilhos: "criar projeto front", "novo app React", "onde coloco
  esse componente/hook/service", "estruturar features", "montar a arquitetura".
metadata:
  version: "1.0"
---

# Arquitetura Front-end (feature-based)

Arquitetura padrão para projetos React + Vite do Romero. **Feature-first**: cada
domínio é isolado em `features/`; só o que é genuinamente reutilizável sobe para
`shared/`. Antes de scaffoldar qualquer coisa, **rode as perguntas de scoping** da
seção final.

## A árvore

```
src/
├── app/                 # Configuração da aplicação (rotas, providers, setup)
│   ├── routes/          # Configuração das rotas (React Router Dom)
│   ├── App.tsx          # Componente raiz
│   ├── providers/       # Providers globais (ThemeProvider, QueryProvider…)
│   └── layout/          # Layouts globais (Header, Footer, Sidebar)
├── assets/              # Recursos estáticos (imagens, ícones, fontes)
├── features/            # Funcionalidades isoladas por domínio
│   ├── auth/
│   │   ├── components/  # Componentes específicos dessa feature
│   │   ├── hooks/       # Hooks exclusivos da feature
│   │   ├── pages/       # Páginas da feature (Login, Register…)
│   │   ├── services/    # Chamadas de API da feature
│   │   └── utils/       # Utilitários específicos da feature
│   └── dashboard/       # (mesma estrutura interna)
├── shared/              # Recursos compartilhados entre features
│   ├── components/      # Componentes reutilizáveis (Button, Modal…)
│   ├── hooks/           # Hooks reutilizáveis
│   ├── utils/           # Funções utilitárias globais
│   ├── styles/          # Estilos globais e temas
│   └── types/           # Tipos TypeScript compartilhados
├── api/                 # Configuração de chamadas à API
│   ├── apiClient.ts     # Instância do Axios
│   ├── queries/         # Queries do React Query
│   └── endpoints.ts     # Definição dos endpoints
├── state/               # Gerenciamento de estado global (Zustand)
├── tests/               # Testes integrados e e2e (Cypress, Vitest)
└── main.tsx             # Arquivo principal
```

## Onde vai cada coisa (regra de decisão)

Ao criar um arquivo, pergunte **"quantas features usam isso?"**:

- **Uma feature** → dentro da própria `features/<x>/` (component, hook, service, util, page).
- **Duas ou mais features** → sobe para `shared/`.
- **Config de app** (rotas, providers, layout raiz) → `app/`.
- **Falar com a API** → cliente e endpoints em `api/`; a *chamada de dados de uma feature* mora em `features/<x>/services/` usando o `apiClient`.
- **Estado global** de verdade (sessão, tema, carrinho) → `state/`. Estado local da feature fica na feature.

## Regras de ouro

- ✅ **Feature não importa de outra feature.** Se `dashboard` precisa de algo de `auth`, esse algo está no lugar errado → mova para `shared/`.
- ❌ `features/dashboard/components/UserCard.tsx` importando `features/auth/hooks/useUser`.
- ✅ `shared/` só recebe o que é **de fato genérico e sem dependência de domínio**. Um `Button` genérico, sim. Um `LoginForm`, não.
- ❌ Jogar tudo em `shared/components/` "pra reaproveitar depois" (YAGNI). Só sobe quando a segunda feature realmente pede.
- ✅ **Serviços de dados na feature** (`features/x/services/`) chamando o `api/apiClient`. Queries do React Query centralizadas em `api/queries/` ou por feature — escolha uma convenção e mantenha.
- ✅ **Barrel export por feature** (`features/x/index.ts`) expondo só o que é público da feature. O resto é privado.
- ❌ Página importando direto de `features/x/components/interno/detalhe`. Importe do barrel.
- ✅ **Path aliases** (`@/features`, `@/shared`, `@/api`) em vez de `../../../`. Configure em `vite.config.ts` + `tsconfig.json`.
- ✅ **Tipos compartilhados** em `shared/types/`; tipos internos da feature ficam na feature.
- ✅ Um arquivo cresceu demais / faz coisa de mais de um domínio → sinal de que precisa ser quebrado ou de que parte dele é `shared/`.

## Stack padrão (default do Romero)

Estes são os **defaults** — confirme nas perguntas de scoping, o usuário pode trocar:

| Camada         | Default                    | Onde vive        |
| -------------- | -------------------------- | ---------------- |
| Rotas          | React Router Dom           | `app/routes/`    |
| Estado global  | Zustand                    | `state/`         |
| Data-fetching  | React Query (+ Axios)      | `api/`           |
| Testes         | Vitest (unit) + Cypress (e2e) | `tests/`      |
| Build          | Vite                       | raiz             |

## Questionário de bibliotecas (rode ANTES de scaffoldar)

Ao iniciar um front, faça este questionário. Para **cada item marcado**, o scaffold
já **instala e configura** (não só cria a pasta). Defaults do Romero em **negrito**.

1. **Quais features** já dá pra nomear? (ex.: `auth`, `dashboard`, `settings`) — cada uma vira pasta em `features/`.
2. **Package manager** — **npm** / pnpm / yarn / bun?
3. **Router** — **React Router Dom** / nenhum? → configura `app/routes/` + `RouterProvider`.
4. **Estado global** — **Zustand** / Redux Toolkit / Context puro / nenhum agora? → cria `state/` com store base.
5. **Data-fetching** — **React Query + Axios** / fetch puro / nenhum? → configura `api/apiClient`, `QueryProvider` e um hook de exemplo.
6. **Styling** — **Tailwind** / CSS Modules / styled-components? → se Tailwind, instala e configura `tailwind.config` + diretivas em `shared/styles/`.
7. **UI kit (se Tailwind)** — **shadcn/ui** / Radix UI / Headless UI / só Tailwind puro? → inicializa o kit e o alias de componentes.
8. **Forms/validação** — **React Hook Form + Zod** / nenhum agora?
9. **Testes** — **Vitest + Testing Library** (unit) e/ou **Cypress** (e2e) / depois?
10. **Extras comuns** — ESLint+Prettier, `.env` (VITE_API_URL), path aliases `@/*`? (**sim** por padrão).

Confirme a lista final com o usuário ("vou instalar: X, Y, Z e configurar tudo — ok?")
**antes** de rodar. Só depois, **scaffold**.

## Scaffold (quando o usuário confirmar)

Quando o escopo estiver fechado e o usuário pedir para montar, **leia
`references/scaffold.md`** — ele tem os comandos e os templates dos arquivos-base
(apiClient, store Zustand, provider do React Query, aliases, feature de exemplo).
Gere só as pastas das features confirmadas e só as camadas escolhidas (não crie
`state/` se ele disse que não usa estado global agora).

## Checklist antes de entregar

- [ ] Nenhum import cruzado entre features (`features/a` não puxa de `features/b`).
- [ ] Nada específico de domínio dentro de `shared/`.
- [ ] Cada feature expõe um barrel `index.ts` com sua API pública.
- [ ] Path aliases configurados (sem `../../../`).
- [ ] `api/` só configura o cliente; a chamada de dados de cada feature está na feature.
- [ ] Só foram criadas as camadas confirmadas nas perguntas de scoping.
