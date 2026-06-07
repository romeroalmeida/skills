# Testes — Clean Code

Para todo TS/JS, front e back. Alvo: **Vitest** (ou Jest), **Testing Library**, **Playwright**.

## 1. Estratégia (pirâmide / troféu)

- Muitos **unitários** (funções puras, regra de negócio) — rápidos e focados.
- Camada saudável de **integração** (componentes com seus filhos, service + repo, rotas) —
  é onde a maioria dos bugs reais aparece (o "troféu de testes").
- Poucos **e2e** nos fluxos críticos do usuário (login, checkout).
- Não persiga 100% de cobertura. Cubra **comportamento e caminhos de risco**, não getters
  triviais. Cobertura é diagnóstico, não meta.

## 2. Teste comportamento, não implementação

- Verifique **o que** o código faz (saída/efeito observável), não **como** (estado interno,
  métodos privados). Testar implementação gera testes frágeis que quebram em todo refactor.
- Se o teste precisa espiar detalhes privados, repense o teste ou o design.

## 3. Estrutura e nomes

- **AAA**: Arrange → Act → Assert, visualmente separados.
- Nome descreve o cenário e o esperado: `deve rejeitar email inválido`,
  `retorna 404 quando usuário não existe`.
- **Um conceito por teste**; vários `expect` ok se testam o mesmo comportamento.
- Testes são **determinísticos e isolados**: sem dependência de ordem, relógio real,
  rede real, ou estado compartilhado. Controle tempo (`vi.useFakeTimers`) e aleatoriedade.

```ts
import { describe, it, expect } from "vitest";

describe("calculateDiscount", () => {
  it("aplica 10% para pedidos acima de 100", () => {
    const total = calculateDiscount(200);      // Act
    expect(total).toBe(180);                    // Assert
  });
});
```

## 4. Componentes React (Testing Library)

- Teste como o **usuário** interage. Queries por **papel/acessibilidade**:
  `getByRole`, `getByLabelText`, `getByText` — evite `getByTestId` (só como último recurso)
  e nunca selecione por classe CSS/estrutura DOM.
- Interaja com `@testing-library/user-event` (mais realista que `fireEvent`).
- Asserções com `@testing-library/jest-dom` (`toBeVisible`, `toBeDisabled`).
- Não teste detalhe de estado interno; teste o que renderiza/acontece.

```tsx
it("envia o formulário com o nome digitado", async () => {
  const user = userEvent.setup();
  const onSubmit = vi.fn();
  render(<NameForm onSubmit={onSubmit} />);

  await user.type(screen.getByLabelText(/nome/i), "Ana");
  await user.click(screen.getByRole("button", { name: /salvar/i }));

  expect(onSubmit).toHaveBeenCalledWith({ name: "Ana" });
});
```

## 5. Mocks com parcimônia

- Faça **mock só de fronteiras** (rede, sistema de arquivos, relógio, serviços externos),
  não da lógica que você está testando.
- **Rede**: use **MSW** (Mock Service Worker) — intercepta no nível HTTP, sem mockar `fetch`
  manualmente; serve para front e back.
- Evite mocks profundos que duplicam a implementação real (quebram junto e dão falsa segurança).
- Prefira **fakes/builders** a mocks gigantes. Use **test data builders** para criar dados:

```ts
const aUser = (over: Partial<User> = {}): User => ({
  id: "u1", email: "a@b.com", role: "viewer", ...over,
});
```

## 6. Backend / NestJS

- Service: unit test com repositório **mockado** (DI facilita), cobrindo regras e erros.
- Fluxo HTTP: e2e com `Test.createTestingModule` + `supertest`, validando status, payload e
  validação de DTO. Banco de teste real (containers/SQLite) > mockar o ORM, quando viável.
- Teste os caminhos de erro (401/403/404/409), não só o feliz.

## 7. E2E (Playwright)

- Apenas jornadas críticas; selecione por papel/texto (`getByRole`), espere por estado
  (auto-waiting), evite `sleep` fixo. Mantenha poucos e estáveis (flaky test é pior que
  nenhum teste).

## 8. Higiene

- Roda no CI; falha quebra o build. Rápidos no caminho local (watch).
- Sem testes ignorados (`.skip`/`.only`) commitados. Sem `console.log` de debug.
- Trate teste como código de produção: legível, DRY no setup (helpers/builders), sem lógica
  condicional complexa dentro do teste.

## Checklist Testes

- [ ] Cobre comportamento e caminhos de risco (incl. erros), não detalhes internos.
- [ ] AAA claro; nomes descritivos; um conceito por teste.
- [ ] Determinístico e isolado (sem rede/relógio/ordem reais não controlados).
- [ ] React: queries por papel/label; `user-event`; sem `testId`/seletor de CSS.
- [ ] Mock só de fronteiras (MSW para HTTP); builders para dados.
- [ ] Backend: service com repo mockado + e2e no fluxo; casos de erro cobertos.
- [ ] E2E só em jornadas críticas, sem flakiness; sem `.only`/`.skip` commitado.
- [ ] Roda e passa no CI.

## Fontes

Verificado em jun/2026: o "Testing Trophy" e o princípio guia da Testing Library.

- Kent C. Dodds — Testing Trophy ("Write tests. Not too many. Mostly integration."):
  https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications
- Testing Library — Guiding Principles ("the more your tests resemble the way your
  software is used…"): https://testing-library.com/docs/guiding-principles
- Testing Library — sobre queries por papel/acessibilidade: https://testing-library.com/docs/queries/about
- Vitest: https://vitest.dev · Playwright: https://playwright.dev · MSW: https://mswjs.io
- NestJS — Testing: https://docs.nestjs.com/fundamentals/testing
