# Acessibilidade (a11y) — Clean Code

Acessibilidade é requisito, não enfeite. Alvo prático: **WCAG 2.2 AA**. Combine com
`react.md` (componentes) e `styling.md` (contraste, motion).

## 1. HTML semântico primeiro (a regra mais importante)

- Use o elemento **certo**: `button` para ações, `a[href]` para navegação, `nav`, `main`,
  `header`, `footer`, `ul/ol`, `table` para dados tabulares, `h1..h6` em ordem lógica.
- **`<div onClick>` não é botão**: não tem foco, teclado, nem papel. Quase todo "ARIA" que
  você precisaria some quando você usa o elemento nativo correto.
- Uma `<h1>` por página/seção principal; hierarquia de headings sem pular níveis.
- **Primeira regra do ARIA:** não use ARIA se um elemento nativo resolve. ARIA não adiciona
  comportamento (foco/teclado) — só semântica.

## 2. Teclado e foco

- **Tudo operável só com teclado** (Tab/Shift+Tab/Enter/Espaço/Esc/setas). Teste navegando
  sem mouse.
- **Foco visível** sempre: nunca `outline: none` sem substituto. Use `:focus-visible`.
- **Ordem de foco** lógica (segue o DOM); não use `tabindex` positivo. `tabindex="0"` para
  tornar focável um custom widget; `tabindex="-1"` para foco programático.
- **Gerência de foco** em UI dinâmica: ao abrir modal, mova o foco pra dentro e **prenda**
  (focus trap); ao fechar, devolva ao gatilho. Em SPA, mova o foco/anuncie ao trocar de rota.
- **Skip link** ("pular para o conteúdo") no topo.

## 3. ARIA (quando o nativo não basta)

- Use **roles/states/properties** corretos em widgets customizados, seguindo os **WAI-ARIA
  Authoring Practices** (padrões prontos para dialog, menu, tabs, combobox, accordion,
  disclosure, tooltip).
- Estados dinâmicos: `aria-expanded`, `aria-selected`, `aria-checked`, `aria-current`,
  `aria-disabled`, `aria-pressed`. Mantenha-os sincronizados com o estado real.
- Nome acessível: `aria-label`/`aria-labelledby` quando não há texto visível; `aria-describedby`
  para descrição/erro. **ARIA errado é pior que nenhum** — não invente roles.
- **Prefira componentes headless** (Radix, React Aria) para esses widgets: já trazem ARIA,
  foco e teclado corretos.

## 4. Formulários

- Todo input com **`<label>` associado** (`htmlFor`/`id`) — placeholder **não** é label.
- Erros: associe a mensagem com `aria-describedby`, marque o campo com `aria-invalid`, e
  **mova o foco** para o primeiro campo inválido no submit.
- Agrupe relacionados com `fieldset`/`legend`. Indique campos obrigatórios de forma textual
  (não só cor). Use `autocomplete` e `inputmode` adequados.

## 5. Imagens, mídia e ícones

- `alt` **descritivo** em imagens informativas; `alt=""` em decorativas. Ícone-botão precisa
  de nome acessível (`aria-label`).
- Vídeo/áudio: **legendas** e transcrição; não dependa só de áudio para informação.
- Não transmita informação **só por cor** (ex.: erro só em vermelho) — adicione ícone/texto.

## 6. Cor, contraste e movimento

- Contraste mínimo **4,5:1** (texto normal) e **3:1** (texto grande/ícones/bordas de UI).
  Verifique nos temas claro e escuro.
- Respeite **`prefers-reduced-motion`** (reduza animações; veja `styling.md`). Sem conteúdo
  que pisque > 3x/s (risco de convulsão).
- Texto deve sobreviver a **zoom 200%** e reflow em 320px sem perda de conteúdo (use `rem`).

## 7. Conteúdo dinâmico (anúncios para leitor de tela)

- Use **live regions** (`aria-live="polite"`/`"assertive"`, `role="status"`/`"alert"`) para
  toasts, validações e resultados de busca carregados sem reload.
- Reflita estados de loading de forma perceptível (não só spinner visual).

## 8. Testes de acessibilidade

- **Lint**: `eslint-plugin-jsx-a11y` no CI.
- **Automático**: `axe`/`@axe-core/react`, Playwright `@axe-core/playwright`, Lighthouse a11y.
  (Automação pega ~30–50% — não substitui teste manual.)
- **Manual**: navegue só com teclado; teste com leitor de tela (NVDA/VoiceOver); verifique
  zoom 200% e foco visível.
- Em testes (Testing Library), consulte por papel/label (`getByRole`, `getByLabelText`) —
  isso já valida acessibilidade básica (veja `testing.md`).

## Checklist Acessibilidade

- [ ] HTML semântico; elemento certo para cada função; headings em ordem.
- [ ] 100% operável por teclado; foco sempre visível (`:focus-visible`); ordem lógica.
- [ ] Foco gerenciado em modal/menu/rota (trap + retorno ao gatilho); skip link.
- [ ] Widgets customizados seguem WAI-ARIA (ou usam lib headless); estados ARIA sincronizados.
- [ ] Inputs com label; erros com `aria-invalid` + `aria-describedby` + foco; sem info só por cor.
- [ ] `alt` adequado; ícone-botão nomeado; mídia com legendas.
- [ ] Contraste AA nos dois temas; respeita `prefers-reduced-motion`; zoom 200% ok.
- [ ] Conteúdo dinâmico anunciado via live region.
- [ ] `jsx-a11y` + axe no CI; verificação manual de teclado/leitor de tela feita.

## Fontes

Razões de contraste verificadas na fonte oficial (W3C) em jun/2026: WCAG 2.2 SC 1.4.3
exige 4,5:1 (texto normal) e 3:1 (texto grande, ≥18pt ou ≥14pt negrito).

- W3C — WCAG 2.2 (norma): https://www.w3.org/TR/WCAG22/
- W3C — Understanding 1.4.3 Contrast (Minimum): https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html
- W3C — WAI-ARIA Authoring Practices (padrões de widgets): https://www.w3.org/WAI/ARIA/apg/
- MDN — Accessibility: https://developer.mozilla.org/en-US/docs/Web/Accessibility
- Deque — axe / regras: https://www.deque.com/axe/ · eslint-plugin-jsx-a11y: https://github.com/jsx-eslint/eslint-plugin-jsx-a11y
