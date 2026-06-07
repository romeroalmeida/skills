# Design de API HTTP — Clean Code

Boas práticas para APIs HTTP/REST (vale para NestJS, Express, Fastify, Route Handlers do
Next). Combine com `nestjs.md`, `security.md` e `typescript.md`.

## 1. Recursos e verbos

- Modele **recursos** (substantivos no plural): `/users`, `/users/{id}/orders`. Verbo é o
  **método HTTP**, não a URL (`POST /users`, não `POST /createUser`).
- Semântica correta dos métodos:

| Método  | Uso                    | Seguro | Idempotente |
| ------- | ---------------------- | :----: | :---------: |
| GET     | ler                    |   ✅   |     ✅      |
| POST    | criar / ação           |   ❌   |     ❌      |
| PUT     | substituir por inteiro |   ❌   |     ✅      |
| PATCH   | atualização parcial    |   ❌   |     ❌¹     |
| DELETE  | remover                |   ❌   |     ✅      |

¹ PATCH pode ser desenhado idempotente. **GET nunca** muda estado.

## 2. Status codes corretos

| Código | Quando                                                        |
| ------ | ------------------------------------------------------------ |
| 200    | OK (GET/PATCH/PUT com corpo)                                  |
| 201    | Created (POST que cria — retorne `Location` e o recurso)      |
| 204    | No Content (DELETE/ação sem corpo)                           |
| 400    | Requisição malformada / validação falhou                     |
| 401    | Não autenticado · **403** autenticado mas sem permissão       |
| 404    | Recurso não existe (ou esconder existência por segurança)     |
| 409    | Conflito (duplicado, versão obsoleta)                        |
| 422    | Entidade não processável (validação semântica)               |
| 429    | Rate limit excedido (com `Retry-After`)                      |
| 500    | Erro inesperado do servidor (sem vazar stack)                |

Não responda 200 com `{ "error": ... }` no corpo — use o status HTTP certo.

## 3. Formato de erro padronizado (RFC 9457)

Use **Problem Details for HTTP APIs** (RFC 9457, que obsoleta a 7807),
`Content-Type: application/problem+json`:

```json
{
  "type": "https://api.exemplo.com/errors/validation",
  "title": "Dados inválidos",
  "status": 422,
  "detail": "O campo email é obrigatório",
  "instance": "/users",
  "errors": [{ "field": "email", "message": "obrigatório" }]
}
```

Erros consistentes em toda a API; mensagens acionáveis; **nunca** stack trace/SQL/detalhes
internos no corpo.

## 4. Validação na borda

- **Todo** input (body, query, params, headers) é validado em runtime no controller/handler
  (zod, class-validator). Tipo estático não valida dado externo. Rejeite com 400/422 +
  Problem Details. Veja `nestjs.md` (DTO + `ValidationPipe` com `whitelist`).

## 5. Coleções: paginação, filtro, ordenação

- **Pagine sempre** listas. **Cursor/keyset** (`?cursor=…&limit=…`) escala melhor que
  offset em dados grandes (offset fica lento e "pula" itens sob escrita concorrente).
- Filtros e ordenação explícitos e validados (`?status=active&sort=-createdAt`). Defina
  `limit` máximo. Retorne metadados de paginação (próximo cursor / total quando viável).

## 6. Versionamento e evolução

- Versione quando quebrar contrato: `/v1/…` (simples e explícito) ou header. Prefira
  **mudanças aditivas** não-quebráveis (novos campos opcionais) a versão nova.
- Nunca remova/renomeie campo sem deprecação anunciada. Respostas devem tolerar campos
  extras (clientes idem).

## 7. Idempotência e concorrência

- Operações com retry (pagamentos, criação) devem aceitar **`Idempotency-Key`** para evitar
  efeito duplicado em reenvio.
- Concorrência otimista com **`ETag` + `If-Match`** (ou campo `version`) → responda **409**
  em conflito de versão.

## 8. Contrato e consistência

- **Contrato primeiro / OpenAPI**: documente e, idealmente, gere tipos do schema (consistência
  cliente⇄servidor). No Nest, use `@nestjs/swagger`.
- Convenção de nomes consistente (camelCase ou snake_case — escolha uma); datas em **ISO 8601
  UTC**; dinheiro em inteiro de menor unidade (centavos) ou decimal string, nunca float.
- Respostas previsíveis: mesma forma para o mesmo recurso; envelope só se padronizado.

## 9. Transversais (apontam para outras referências)

- **Segurança**: authn/authz por request, CORS restrito, rate limit, headers — `security.md`.
- **Cache HTTP**: `Cache-Control`/`ETag` em GETs cacheáveis; compressão (gzip/brotli).
- **Observabilidade**: log estruturado com correlação, métricas, tracing — `node.md`.

## Checklist API

- [ ] Recursos como substantivos; método HTTP com semântica correta (safe/idempotent).
- [ ] Status codes adequados (não 200-com-erro); `Location` no 201.
- [ ] Erros padronizados em Problem Details (RFC 9457); sem detalhes internos vazados.
- [ ] Todo input validado em runtime na borda (400/422).
- [ ] Listas paginadas (cursor/keyset), com filtro/ordenação validados e `limit` máximo.
- [ ] Evolução aditiva; versionamento explícito quando quebrar contrato.
- [ ] Idempotency-Key em operações sensíveis a retry; ETag/If-Match para concorrência.
- [ ] OpenAPI/contrato; nomes/datas/dinheiro consistentes.
- [ ] Segurança, cache e observabilidade transversais aplicados.

## Fontes

Verificado em jun/2026: RFC 9457 define "Problem Details for HTTP APIs" (campos `type`,
`title`, `status`, `detail`, `instance`) e **obsoleta a RFC 7807**.

- IETF — RFC 9457 (Problem Details): https://www.rfc-editor.org/rfc/rfc9457
- IETF — RFC 9110 (HTTP Semantics, métodos e status): https://www.rfc-editor.org/rfc/rfc9110
- MDN — HTTP (métodos, status, headers): https://developer.mozilla.org/en-US/docs/Web/HTTP
- MDN — Idempotent / Safe: https://developer.mozilla.org/en-US/docs/Glossary/Idempotent
- NestJS — OpenAPI/Swagger: https://docs.nestjs.com/openapi/introduction