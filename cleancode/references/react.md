# React — Clean Code (React 18/19+)

Referência de arquitetura front-end com React moderno e TS. Para Next, combine com
`nextjs.md`. Para estilo, acessibilidade e performance de runtime, veja `styling.md`,
`accessibility.md` e `web-performance.md`.

> Mapa rápido: **Componentes & tipagem** (§1–2) · **Composição** (§3) · **Hooks** (§4–6)
> · **Estado local/global** (§7–8) · **Data fetching** (§9) · **React 19** (§10) ·
> **Suspense/erros** (§11) · **Performance** (§12) · **Refs/DOM/portais** (§13) ·
> **Formulários** (§14) · **Estrutura** (§15) · **Anti-padrões** (§16) · **Checklist** (§17)

## 1. Componentes — fundamentos

- **Function components** sempre; sem classes (exceto Error Boundary legado).
- Componente é **puro durante o render**: nada de I/O, mutação de props/estado, escrita no
  DOM ou `Math.random()`/`Date.now()` que mude a saída. Efeitos colaterais só em handlers
  ou `useEffect`.
- **Uma responsabilidade.** Se passa de ~150 linhas, mistura fetch + layout + regra, ou tem
  muitos `useState`, quebre em subcomponentes + custom hooks.
- **Top-down de dados, bottom-up de eventos:** dados descem por props, mudanças sobem por
  callbacks. Não sincronize estado duplicado entre pai e filho.

## 2. Tipagem de props (TS)

- Tipe as props diretamente; **não use `React.FC`** (atrapalha generics e torna `children`
  implícito). Desestruture na assinatura.
- Reaproveite tipos do DOM/de libs com **`ComponentProps`** em vez de redeclarar:

```tsx
// herda todos os atributos nativos de <button> + adiciona variant
type ButtonProps = React.ComponentPropsWithoutRef<"button"> & {
  variant?: "primary" | "ghost";
};
function Button({ variant = "primary", ...rest }: ButtonProps) {
  return <button className={variant} {...rest} />;
}
```

- **Props mutuamente exclusivas → discriminated union** (torna estados impossíveis
  irrepresentáveis em tempo de compilação):

```tsx
type AlertProps =
  | { variant: "success"; message: string }
  | { variant: "error"; message: string; onRetry: () => void }; // retry só no erro
```

- **Componentes genéricos** para listas/tabelas reutilizáveis:

```tsx
function List<T>({ items, renderItem }: {
  items: readonly T[];
  renderItem: (item: T) => React.ReactNode;
}) {
  return <ul>{items.map(renderItem)}</ul>;
}
```

- **`children: React.ReactNode`**; para "render prop" use uma função tipada. Para um
  componente polimórfico (`as`), use `ElementType` + generics (ou prefira uma lib headless).
- **React 19:** `ref` é uma prop normal — não precisa mais de `forwardRef`. Em React 18,
  use `forwardRef` quando o componente precisa expor um nó do DOM.

## 3. Padrões de composição

Resolva reuso por **composição**, não por props infinitas nem herança.

- **Composição por `children`/slots** antes de prop drilling: passe JSX em vez de afundar
  dados por vários níveis.

```tsx
// ✅ slots: Card não precisa saber o que vai dentro
<Card header={<Title />} footer={<Actions />}>{body}</Card>
```

- **Compound components** para APIs declarativas com estado compartilhado via Context
  interno (`<Tabs><Tabs.List/><Tabs.Panel/></Tabs>`).
- **Custom hooks** para compartilhar *lógica*; **componentes** para compartilhar *UI*.
- **Headless UI** (Radix, React Aria, Headless UI) para componentes complexos
  (modal, combobox, menu): você ganha acessibilidade e comportamento corretos e só estiliza.
- O velho **container/presentational** foi superado por hooks + RSC — não force essa divisão.

## 4. Regras dos Hooks

- Só no **top level** de componentes/custom hooks — nunca em condicionais, loops, callbacks
  ou após um early return.
- Custom hooks começam com `use`.
- **Liste todas as dependências** honestamente; não silencie `react-hooks/exhaustive-deps`.
  Se a regra "incomoda", o design do efeito está errado (veja §6).

## 5. Custom hooks (design)

- Extraia para um hook quando há **lógica com estado reutilizável** (não só para "organizar").
- Retorne o mínimo necessário; prefira **tupla** para 2 valores posicionais
  (`const [open, toggle] = useToggle()`) ou **objeto** para vários nomeados.
- Estabilize funções retornadas com `useCallback` quando elas vão para deps de quem consome.
- Um hook = uma responsabilidade; componha hooks menores em vez de um hook gigante.

## 6. Você (quase sempre) NÃO precisa de `useEffect`

Maior fonte de bugs em React. Efeito serve para **sincronizar com sistema externo**
(rede não-gerenciada, DOM imperativo, subscriptions, timers) — **não** para reagir a
mudanças de props/estado.

| Situação                                   | Em vez de Effect, faça                          |
| ------------------------------------------ | ------------------------------------------------ |
| Derivar valor de props/estado             | Calcule no render (ou `useMemo` se caro)         |
| Responder a evento do usuário              | Lógica no **event handler**                      |
| Resetar estado quando um prop muda         | `key` no componente                              |
| Buscar dados                               | RSC / TanStack Query / SWR (§9)                  |
| "Notificar o pai" quando algo muda         | Chame o callback no handler, não num effect      |
| Estado computado de outro estado           | Derive; não duplique em outro `useState`         |

```tsx
// ❌ estado redundante + effect (re-render extra, pode dessincronizar)
const [full, setFull] = useState("");
useEffect(() => setFull(`${first} ${last}`), [first, last]);

// ✅ derive direto no render
const full = `${first} ${last}`;
```

Quando o Effect é legítimo, **sempre faça cleanup**:

```tsx
useEffect(() => {
  const ctrl = new AbortController();
  api.subscribe(id, { signal: ctrl.signal });
  return () => ctrl.abort();      // cancela ao desmontar/redepender
}, [id]);
```

## 7. Estado local

- **Mínimo e com dono único.** Não duplique; derive o resto. **Colocation:** mantenha o
  estado o mais perto de quem usa; só levante (lift) quando vários precisam.
- `useState` para valores simples e independentes; **`useReducer`** quando há várias
  transições relacionadas, a próxima depende da anterior, ou a lógica de update ficou complexa.
- Updates baseados no anterior usam função: `setCount(c => c + 1)`. Estado é objeto/array →
  crie novo (imutável), não mute.
- Não guarde no estado o que dá para derivar; não copie props para o estado (a menos que
  intencionalmente "uncontrolled" — e aí use `key` para resetar).

## 8. Estado global / compartilhado

Escolha a ferramenta pelo tipo de estado:

- **Estado de servidor** (dados que vêm de API) **não é estado global** — use TanStack
  Query/SWR/RSC (§9). Não jogue resposta de API no Redux/Zustand "na mão".
- **Context** só para valores **estáveis e globais** (tema, auth, i18n, locale). Context
  re-renderiza **todos** os consumidores quando o value muda → não use para estado que
  muda com frequência. Se precisar, **divida** em contexts menores (state vs dispatch) e
  memoize o `value`.
- **Estado de cliente complexo** (carrinho, filtros, UI global): **Zustand** (simples,
  sem boilerplate, seletores evitam re-render) ou **Jotai** (atômico) ou **Redux Toolkit**
  (apps grandes, devtools, middleware). Evite Context como "gerenciador de estado".
- **URL como estado**: filtros, paginação, abas e busca devem morar nos **search params**
  (compartilhável, navegável, sobrevive a refresh), não só no `useState`.

## 9. Data fetching (estado de servidor)

- Em **Server Components** (Next), busque no servidor com `async/await` (veja `nextjs.md`).
  Em apps client-side, use uma lib — **nunca** `useEffect` + `fetch` manual (race
  conditions, waterfalls, sem cache/retry/dedupe).
- **TanStack Query** (padrão de mercado): cache, dedupe, revalidação, retry, paginação.

```tsx
// query keys descritivas e hierárquicas → cache e invalidação previsíveis
const { data, isPending, error } = useQuery({
  queryKey: ["user", userId],
  queryFn: ({ signal }) => fetchUser(userId, signal),
  staleTime: 60_000,            // evita refetch desnecessário
});

const mutation = useMutation({
  mutationFn: updateUser,
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ["user", userId] }),
});
```

- **Optimistic updates** para UX instantânea; reverta no `onError`.
- Trate sempre os 3 estados: **loading**, **erro** e **vazio** (não só o feliz).
- Evite **waterfalls**: dispare requests independentes em paralelo; use prefetch.

## 10. React 19 — recursos modernos

- **Actions + `useActionState`**: gerenciam pending/erro/resultado de uma mutação async sem
  `useState` manual.

```tsx
const [state, formAction, isPending] = useActionState(submitForm, initialState);
return <form action={formAction}>{isPending ? "Enviando…" : <Submit />}</form>;
```

- **`useFormStatus`**: um botão de submit sabe se o `<form>` pai está enviando, sem prop drilling.
- **`useOptimistic`**: estado otimista declarativo durante uma action.
- **`use()`**: lê uma Promise (com Suspense) ou Context — pode ser chamado condicionalmente.
- **`ref` como prop** (sem `forwardRef`); **`<title>`/`<meta>`/`<link>`** podem ser
  renderizados em qualquer componente (document metadata).
- **React Compiler**: memoização automática — pare de espalhar `useMemo`/`useCallback`/`memo`
  preventivos (§12).

## 11. Suspense e Error Boundaries

- **`<Suspense fallback>`** para estados de carregamento declarativos (RSC, `use()`, lazy,
  libs compatíveis). Coloque a fronteira no nível certo para fazer **streaming** das partes
  prontas sem travar a página.
- **Error Boundary** para falhas de render (não pega erro de event handler/async — esses
  trate no `try/catch`/`onError`). Use **`react-error-boundary`** com `FallbackComponent` e
  `onReset`/`resetKeys`. Tenha um boundary global + boundaries locais nos pontos de risco.

## 12. Performance (meça antes de otimizar)

- **Por que re-renderiza:** um componente re-renderiza quando seu estado/props mudam ou o
  pai re-renderiza. Re-render não é "ruim" por si só — só vira problema se for caro/frequente.
- **React 19 + React Compiler** memoiza automaticamente. Sem o compiler:
  - `React.memo` em componente puro **caro** que recebe as mesmas props com frequência.
  - `useCallback`/`useMemo` para manter **referências estáveis** passadas a filhos
    memoizados ou a deps de hooks — não como reflexo em tudo.
- **Context:** divida providers e use seletores (Zustand) para não re-renderizar a árvore toda.
- **`useTransition`** para updates não urgentes (manter a UI responsiva durante filtros
  pesados); **`useDeferredValue`** para adiar valor derivado custoso.
- **Code splitting:** `React.lazy` + `Suspense` (ou `next/dynamic`) para rotas/componentes
  pesados e abaixo da dobra.
- **Listas longas:** virtualização (`@tanstack/react-virtual`).
- **Hot paths:** evite criar objetos/funções/arrays inline como props de filhos memoizados;
  estabilize `key`s; não faça trabalho pesado no render. Detalhes de rede/assets em
  `web-performance.md`.

## 13. Refs, DOM e portais

- `useRef` para valores mutáveis que **não** disparam render e para acessar nós do DOM.
- Não leia/escreva DOM durante o render; faça em effect/handler. Para medir layout antes de
  pintar, `useLayoutEffect` (com parcimônia).
- **`useImperativeHandle`** só quando precisa expor uma API imperativa (raro).
- **Portais** (`createPortal`) para modais/tooltips/toasts que precisam escapar do overflow
  — mas mantenha o foco e a semântica corretos (veja `accessibility.md`).

## 14. Formulários

- Forms pequenos: estado controlado mínimo. Forms médios/grandes: **React Hook Form**
  (menos re-render, validação integrada) + **zod** via `zodResolver` — **mesmo schema** no
  cliente e no servidor.
- Controlado vs não-controlado: prefira não-controlado (RHF/`defaultValue`) quando não
  precisa do valor a cada tecla.
- Acessibilidade de form não é opcional: `label` associado, `aria-invalid`,
  `aria-describedby` para a mensagem de erro, foco no primeiro campo inválido (veja
  `accessibility.md`).
- **Cliente nunca é fonte de verdade:** valide também no servidor.

## 15. Estrutura de arquivos

- Organize **por feature**, não por tipo técnico (evite `components/`, `hooks/`, `utils/`
  globais virando lixeira). Veja `architecture.md`.
- Co-localize: componente + seu hook + seus testes + tipos perto uns dos outros.
- Lógica não-visual sai do JSX para hooks/funções puras testáveis. Sem regra de negócio
  embutida no JSX — extraia para variáveis/funções nomeadas.

## 16. Anti-padrões comuns (evite)

- `useEffect` para derivar/sincronizar estado (§6).
- Índice do array como `key` em lista que reordena/insere/remove.
- "Prop drilling" profundo onde caberia composição/Context.
- Context para estado que muda rápido (re-render geral).
- `useMemo`/`useCallback`/`memo` espalhados sem medir.
- Estado de servidor dentro de Redux/Zustand "na mão" em vez de TanStack Query.
- Mutar estado/props (`arr.push`, `obj.x = …`) em vez de criar novo.
- `dangerouslySetInnerHTML` sem sanitização (XSS).
- `<div onClick>` no lugar de `<button>` (sem teclado/foco/semântica).
- Lógica pesada e objetos inline no render do hot path.

## 17. Checklist React

- [ ] Function components, props tipadas (sem `React.FC`), pureza no render.
- [ ] Props mutuamente exclusivas modeladas com discriminated union; `ComponentProps` reusado.
- [ ] Reuso por composição/slots/compound/headless; sem prop drilling profundo.
- [ ] Nenhum `useEffect` que só deriva/sincroniza/notifica estado; effects legítimos com cleanup.
- [ ] Estado mínimo, com dono único, colocado perto do uso; `key` estável por id.
- [ ] Estado de servidor via RSC/TanStack Query (não `useEffect`+fetch); loading/erro/vazio tratados.
- [ ] Context só para estado global estável (dividido + memoizado); cliente complexo em Zustand/Jotai/RTK; filtros/paginação na URL.
- [ ] React 19 usado quando cabe (Actions/`useActionState`/`useOptimistic`/`use()`); ref como prop.
- [ ] Suspense + Error Boundary cobrindo carregamento e falhas.
- [ ] Memoização só onde o profiler justifica (ou via React Compiler); listas longas virtualizadas.
- [ ] Forms com RHF + zod (schema único client/server); acessíveis.
- [ ] Organização por feature; lógica fora do JSX; sem os anti-padrões da §16.
- [ ] Acessibilidade, estilo e performance de assets conferidos (`accessibility.md`,
      `styling.md`, `web-performance.md`).
