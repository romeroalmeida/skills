# Performance Web — Clean Code

Performance de carregamento e runtime no front (Core Web Vitals e bundle). Para
re-renders do React, veja `react.md` §12.

## 1. Core Web Vitals (as métricas que importam)

| Métrica  | O que mede                          | Meta "bom" | Principais alavancas                                   |
| -------- | ----------------------------------- | ---------- | ------------------------------------------------------ |
| **LCP**  | Maior elemento visível pintado      | < 2,5 s    | Otimizar imagem/hero, server response, preload, CSS crítico |
| **INP**  | Responsividade à interação          | < 200 ms   | Menos JS, quebrar tarefas longas, evitar work no main thread |
| **CLS**  | Estabilidade visual (sem "pulos")   | < 0,1      | Dimensões em mídia, espaço reservado, `font-display`   |

Meça com **Lighthouse** (lab) e **dados de campo** (RUM via lib `web-vitals`,
CrUX). Otimize o que os dados apontam — não no escuro.

## 2. LCP (carregamento percebido)

- **Servidor rápido** (TTFB): cache, CDN, SSR/streaming, edge quando fizer sentido.
- **Imagem/hero**: formato moderno (AVIF/WebP), tamanho certo, `priority`/`fetchpriority="high"`
  no LCP, `preload` do recurso crítico.
- **CSS crítico** inline / sem CSS render-blocking desnecessário; **fontes** com `preload`
  e `font-display: swap` (evita texto invisível).
- Não esconda o conteúdo principal atrás de JS (evite "tudo client-side" para a primeira dobra).

## 3. INP (responsividade)

- **Menos JavaScript** é a maior alavanca. Quebre **long tasks** (> 50 ms); use
  `scheduler`/`isInputPending`, `requestIdleCallback`, ou `useTransition` (React) para
  trabalho não urgente.
- **Debounce/throttle** handlers de input/scroll/resize. Mova cálculo pesado para
  **Web Workers**.
- Evite layout thrash (ler/escrever DOM alternadamente); agrupe leituras e escritas.

## 4. CLS (estabilidade)

- **Sempre** defina dimensões de imagem/vídeo/iframe (`width`/`height` ou `aspect-ratio`).
- Reserve espaço para conteúdo que chega depois (ads, embeds, banners). Não injete conteúdo
  acima do que já está visível.
- Pré-carregue fontes e use `size-adjust`/fallback métrico para evitar reflow ao trocar a fonte.

## 5. Bundle de JavaScript

- **Code splitting** por rota e por componente pesado/abaixo da dobra (`React.lazy` +
  `Suspense`, `next/dynamic`, `import()` dinâmico).
- **Tree shaking**: imports nomeados, ESM, evitar `import * as`; cuidado com **barrel files**
  que arrastam módulos inteiros. Prefira libs modulares.
- **Meça o bundle** (`@next/bundle-analyzer`, `source-map-explorer`). Troque libs pesadas
  (ex.: `date-fns`/`dayjs` no lugar de `moment`; evitar lodash inteiro).
- Cuidado com **CSS-in-JS de runtime** e polyfills desnecessários (ajuste `browserslist`).
- Em Next, mantenha lógica no servidor (RSC) para reduzir JS enviado ao cliente.

## 6. Imagens e mídia

- Formatos modernos (AVIF/WebP), **responsivas** (`srcset`/`sizes`), `loading="lazy"`
  fora da primeira dobra (mas **não** no LCP), `decoding="async"`.
- Use otimizadores (`next/image` e equivalentes): redimensiona, serve formato ideal, evita CLS.
- Vídeo: `preload="none"`/poster; não autoplay pesado.

## 7. Carregamento de recursos (resource hints)

- `preconnect`/`dns-prefetch` para origens de terceiros críticas.
- `preload` para recurso crítico (fonte, imagem LCP); `prefetch` para próxima navegação provável.
- **Reduza terceiros**: cada script externo (analytics, tags, widgets) custa main thread.
  Carregue de forma assíncrona/diferida e audite o impacto.

## 8. Cache e rede

- Cache-Control / immutable + hash no nome do arquivo para assets versionados.
- HTTP/2-3, compressão (Brotli/gzip). Service Worker/PWA quando offline/repeat-visit importa.
- Paginação/infinite scroll + virtualização para listas grandes; não traga dados demais.

## Checklist Performance Web

- [ ] LCP/INP/CLS medidos em lab **e** campo; otimizações guiadas por dados.
- [ ] Imagem LCP priorizada; CSS crítico sem render-block; fontes com `preload` + `swap`.
- [ ] JS minimizado: code splitting, tree shaking, bundle analisado, libs pesadas trocadas.
- [ ] Long tasks quebradas; input com debounce/throttle; trabalho pesado fora do main thread.
- [ ] Mídia com dimensões/`aspect-ratio` (sem CLS); imagens responsivas e lazy (exceto LCP).
- [ ] Resource hints (`preconnect`/`preload`/`prefetch`); terceiros auditados e diferidos.
- [ ] Cache de assets com hash/immutable; compressão ativa; listas grandes virtualizadas.
