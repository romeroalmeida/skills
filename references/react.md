# React — Clean Code (React 19+)

Para componentes, hooks e UI. Alvo: React 18/19, function components, TS. Vale também
dentro de Next (combine com `nextjs.md`).

## 1. Componentes

- **Function components** sempre. Sem classes.
- Props tipadas; desestruture na assinatura. Sem `React.FC` (esconde children e atrapalha
  generics) — tipe as props diretamente.

```tsx
// ✅
type ButtonProps = { variant: "primary" | "ghost"; onClick: () => void; children: React.ReactNode };
function Button({ variant, onClick, children }: ButtonProps) { /* ... */ }
```

- **Um componente, uma responsabilidade.** Se passa de ~150 linhas ou mistura fetch +
  layout + lógica, quebre. Extraia subcomponentes e custom hooks.
- **Componha, não faça prop drilling.** Passe JSX como `children`/slots antes de afundar
  props por vários níveis. Context só para estado verdadeiramente global (tema, auth, i18n)
  — não como substituto de gerenciador de estado.
- Componentes são **puros na renderização**: nada de efeito colateral, mutação de props,
  ou leitura/escrita de DOM durante o render.

## 2. Regras dos Hooks (rigorosas)

- Só chame hooks no **top level** de componentes/custom hooks — nunca em condicionais,
  loops ou callbacks.
- Custom hooks começam com `use`.
- **Liste todas as dependências** de `useEffect`/`useMemo`/`useCallback` honestamente.
  Não silencie a regra `exhaustive-deps`; se "incomoda", o design está errado.

## 3. Você (provavelmente) NÃO precisa de `useEffect`

A maior fonte de bugs em React. Efeito é para **sincronizar com sistemas externos**
(rede, DOM não-React, subscriptions), não para reagir a props/estado.

- **Derivar estado de props/estado → calcule no render**, não em efeito + `useState`.

```tsx
// ❌ estado redundante + efeito
const [fullName, setFullName] = useState("");
useEffect(() => { setFullName(`${first} ${last}`); }, [first, last]);
// ✅ derive direto
const fullName = `${first} ${last}`;
```

- **Cálculo caro → `useMemo`**, não efeito.
- **Responder a evento do usuário → handler do evento**, não efeito que observa estado.
- **Resetar estado quando um prop muda → `key`** no componente.
- Buscar dados: prefira **React Server Components** (Next) ou uma lib (**TanStack Query**,
  SWR) que cuida de cache, dedupe, race e loading. `useEffect`+`fetch` manual sofre com
  race conditions e waterfalls — evite.

## 4. Estado

- **Mínimo e único dono.** Não duplique; derive o resto. Coloque o estado o mais
  **próximo de quem usa** (colocation); levante só quando vários precisam.
- `useState` para valores independentes simples; **`useReducer`** quando há várias
  transições relacionadas ou a próxima depende da anterior.
- Atualizações baseadas no anterior usam função: `setCount(c => c + 1)`.
- Não guarde no estado o que dá pra derivar; não guarde props no estado (a menos que
  intencionalmente "uncontrolled").

## 5. Listas e keys

- `key` **estável e única do dado** (id), nunca o índice quando a lista reordena/insere.
- Não gere `key` com `Math.random()` (remonta tudo a cada render).

## 6. Performance (meça antes)

- React 19 + **React Compiler** memoiza automaticamente — não encha o código de
  `useMemo`/`useCallback`/`memo` preventivamente. Otimize só com profiler apontando o gargalo.
- Quando manual: `React.memo` em componente puro pesado; `useCallback` para refs estáveis
  passadas a filhos memoizados; `useMemo` para cálculo realmente caro.
- Listas longas → virtualização (`@tanstack/react-virtual`).
- Evite criar objetos/funções inline como prop de componentes memoizados no hot path.

## 7. Efeitos colaterais corretos

- Cleanup sempre que houver subscription/timer/listener:

```tsx
useEffect(() => {
  const ctrl = new AbortController();
  fetch(url, { signal: ctrl.signal }).then(/* ... */);
  return () => ctrl.abort(); // cancela ao desmontar/redepender
}, [url]);
```

- Não dispare efeito que seta estado que dispara o mesmo efeito (loop).

## 8. Formulários e eventos

- Inputs controlados com estado mínimo, ou libs (**React Hook Form**) para forms grandes
  — menos re-render e validação integrada (zod resolver).
- Valide no submit **e** no servidor (cliente nunca é fonte de verdade).

## 9. Acessibilidade (não opcional)

- HTML semântico (`button`, `nav`, `label` associado) antes de ARIA. `<div onClick>` não
  é botão (sem foco/teclado).
- Toda imagem com `alt`; inputs com `label`; foco visível; navegável por teclado.
- Rode `eslint-plugin-jsx-a11y`.

## 10. Estrutura e estilo

- Organize **por feature**, não por tipo técnico (evite pastas gigantes `components/`,
  `hooks/` globais para tudo). Veja `architecture.md`.
- Lógica não-visual sai do componente para custom hooks/funções puras testáveis.
- Sem lógica de negócio dentro do JSX; extraia para variáveis/funções nomeadas.

## Checklist React

- [ ] Function components, props tipadas (sem `React.FC`), pureza no render.
- [ ] Nenhum `useEffect` que apenas deriva/calcula estado (derivado no render/`useMemo`).
- [ ] Data fetching via RSC/TanStack Query/SWR, não `useEffect`+`fetch` manual.
- [ ] Dependências de hooks completas; `exhaustive-deps` não silenciado.
- [ ] Estado mínimo, colocado perto do uso; `key` estável por id.
- [ ] Memoização só onde o profiler justifica.
- [ ] Cleanup em subscriptions/timers/fetch.
- [ ] Acessível: semântica, labels, teclado, `jsx-a11y` limpo.
- [ ] Organização por feature; lógica fora do JSX.
