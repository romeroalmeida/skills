# Estilização / CSS — Clean Code

CSS moderno e estratégias de styling para apps TS/JS. Combine com `accessibility.md`
(contraste, motion) e `web-performance.md` (CSS crítico, fontes).

## 1. Escolha de abordagem (trade-offs)

| Abordagem                     | Quando usar                                           | Cuidado                                  |
| ----------------------------- | ----------------------------------------------------- | ---------------------------------------- |
| **CSS Modules**               | Escopo local sem runtime, projeto simples/SSR         | Sem tokens compartilhados nativos        |
| **Tailwind CSS**              | Velocidade, consistência por design tokens, time      | Markup verboso; extraia componentes      |
| **Zero-runtime CSS-in-JS** (vanilla-extract, Panda, Linaria) | Tipagem + tokens, sem custo em runtime | Build setup                              |
| **Runtime CSS-in-JS** (styled-components, emotion) | DX dinâmica                       | **Custo em runtime**; ruim em RSC/SSR    |

- Em **React Server Components / Next App Router**, prefira **CSS Modules, Tailwind ou
  zero-runtime**. CSS-in-JS de runtime briga com Server Components e adiciona JS.
- Seja **consistente**: uma abordagem principal por projeto. Não misture 3 sistemas.

## 2. Design tokens (fonte única de verdade)

- Centralize cor, espaçamento, tipografia, radius, sombra, z-index e breakpoints em
  **tokens** (CSS custom properties ou tema da lib). Nada de valores mágicos espalhados.

```css
:root {
  --color-bg: #fff;
  --color-fg: #111;
  --space-1: 0.25rem; --space-2: 0.5rem; --space-4: 1rem;
  --radius: 0.5rem;
}
[data-theme="dark"] { --color-bg: #111; --color-fg: #f5f5f5; }
```

- Escala de espaçamento consistente (4/8px). **Sem números mágicos** (`margin: 13px`).
- Tipografia com escala modular; `rem` para fontes (respeita zoom do usuário).

## 3. CSS moderno (use o que a plataforma já oferece)

- **Custom properties** para temas e variações em runtime (sem rebuild).
- **Nesting nativo** (`& .child`) — suportado nos browsers atuais; sem precisar de SASS só pra isso.
- **Layout com Flexbox/Grid**; `gap` em vez de margens entre itens. Evite `float`/`position`
  para layout. `subgrid` quando precisar alinhar entre containers.
- **Container queries** (`@container`) para componentes que se adaptam ao **container**, não
  só à viewport — chave para componentes realmente reutilizáveis.
- **`:has()`** (parent selector), `:is()`/`:where()` para simplificar seletores.
- **Logical properties** (`margin-inline`, `padding-block`, `inset`) para suporte a RTL/i18n.
- **`clamp()`/`min()`/`max()`** para tipografia e espaçamento fluidos sem media queries.
- **`aspect-ratio`** para mídia sem CLS; **`object-fit`** para imagens.

## 4. Responsividade

- **Mobile-first**: estilo base no menor, `min-width` para ampliar.
- Breakpoints a partir de tokens; poucos e semânticos. Prefira layouts intrinsecamente
  fluidos (`grid` + `minmax`, `clamp`) a empilhar media queries.
- Toque: alvos de no mínimo ~44px; estados `:hover` não podem ser a única forma de interação.

## 5. Tema escuro

- Implemente via tokens + `[data-theme]` ou `prefers-color-scheme`. Permita override do
  usuário (persistido) e respeite o padrão do sistema.
- Garanta **contraste** nos dois temas (veja `accessibility.md`). Use `color-scheme` para
  os controles nativos acompanharem o tema.

## 6. Manutenibilidade

- **Baixa especificidade**; **evite `!important`** (sinal de arquitetura de CSS quebrada).
  Prefira classes a seletores de elemento/ID encadeados.
- Estilo segue o componente (CSS Module/Tailwind/escopo) — sem CSS global vazando, exceto
  reset/tokens/base.
- **Sem estilos inline** com valores mágicos; use classes/tokens. Estilo inline só para
  valores realmente dinâmicos (ex.: `--x` calculado em runtime).
- Use um **reset/normalize** moderno e `box-sizing: border-box` global.
- Tailwind: extraia padrões repetidos para **componentes** (não copie 15 classes em N
  lugares); use `@apply`/preset com parcimônia.

## 7. Animação e movimento

- Anime apenas **`transform`** e **`opacity`** (compositadas pela GPU) — evite animar
  `width/height/top/left` (causam layout/reflow).
- **Respeite `prefers-reduced-motion`**: reduza/elimine animações não essenciais.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: .01ms !important; transition-duration: .01ms !important; }
}
```

## Checklist Estilo

- [ ] Uma abordagem principal coerente; sem CSS-in-JS de runtime em RSC/SSR.
- [ ] Design tokens centralizados; sem valores mágicos; escala de espaçamento/tipografia.
- [ ] Layout com Flex/Grid + `gap`; container queries onde o componente precisa se adaptar.
- [ ] Logical properties para i18n/RTL; unidades fluidas (`clamp`) onde cabe.
- [ ] Mobile-first; alvos de toque adequados; sem depender só de `:hover`.
- [ ] Dark mode com contraste garantido nos dois temas.
- [ ] Baixa especificidade; sem `!important`; sem CSS global vazando.
- [ ] Anima só `transform`/`opacity`; respeita `prefers-reduced-motion`.
