# Banco de Dados — Clean Code

Acesso a dados em apps TS/JS (Prisma, Drizzle, TypeORM, SQL). Combine com `nestjs.md`
(repository), `api-design.md` (paginação) e `security.md` (injeção, least privilege).

## 1. Modelagem

- **Normalize** por padrão (evita anomalias e dado duplicado); **desnormalize** só com
  motivo de performance medido. Não guarde o mesmo fato em dois lugares sem necessidade.
- Use o **tipo certo** da coluna (`timestamptz` para tempo, `numeric`/`decimal` para dinheiro
  — nunca `float`; `uuid`/`bigint` para id). Datas em UTC.
- **Constraints no banco** são parte da regra de integridade: `NOT NULL`, `UNIQUE`, `CHECK`,
  **foreign keys**. Não confie só na aplicação para integridade referencial.
- Modele o domínio, não a tela. Soft delete (`deletedAt`) quando o negócio exige histórico.

## 2. Camada de acesso

- Isole o ORM atrás de um **repository/serviço** — a regra de negócio não monta query crua
  nem conhece detalhes do driver (veja `nestjs.md`). Facilita teste e troca.
- Prefira o **query builder/ORM tipado** (Prisma, Drizzle) a SQL string concatenada. SQL
  cru quando preciso, sempre **parametrizado** (nunca interpolando input — veja `security.md`).

## 3. N+1: o gargalo clássico

- N+1 = 1 query para a lista + 1 por item para a relação. Detecte logando queries em dev.
- Resolva carregando relações de forma explícita: `include`/`select` (Prisma), `JOIN`,
  ou **DataLoader** (batch) em GraphQL. Não busque relação dentro de um loop.

```ts
// ❌ N+1
const posts = await prisma.post.findMany();
for (const p of posts) p.author = await prisma.user.findUnique({ where: { id: p.authorId } });
// ✅ uma query
const posts = await prisma.post.findMany({ include: { author: true } });
```

## 4. Índices

- Crie índice para colunas usadas em `WHERE`, `JOIN`, `ORDER BY` e FKs. Use **índice
  composto** na ordem das consultas; **covering index** quando vale.
- Não **superindexe**: cada índice custa escrita e espaço. Meça com `EXPLAIN (ANALYZE)`.
- Cuidado com índice inútil por função/cast na coluna (impede uso do índice).

## 5. Transações e isolamento

- Agrupe operações que devem ser **atômicas** numa transação (ex.: debitar A e creditar B).
- **Mantenha transações curtas** — transações longas degradam performance e causam
  deadlocks. Nada de chamada de rede/IO lento dentro da transação.
- Para concorrência forte, eleve o **isolation level** (RepeatableRead/Serializable) e
  implemente **retry** em conflito de serialização (no Prisma, erro `P2034`).
- No Prisma: *nested writes* (relações dependentes), `$transaction([...])` (writes
  independentes em sequência atômica) ou **transação interativa** (read-modify-write).

## 6. Migrations

- Schema versionado por **migrations** no controle de versão. Toda mudança = migration
  revisável e **reversível**.
- **Nunca** `synchronize: true` / auto-migrate em produção (perda de dados).
- Mudanças **zero-downtime**: aditivas primeiro (coluna nullable → backfill → tornar NOT
  NULL → remover antiga depois), compatíveis com a versão atual do app (expand/contract).

## 7. Conexões e performance

- **Connection pool** dimensionado ao banco. Em **serverless**, conexões explodem → use
  pooler (PgBouncer, Prisma Accelerate/Data Proxy) ou driver serverless.
- Evite `SELECT *`; busque só as colunas necessárias. Pagine com **keyset** (veja
  `api-design.md`), não offset grande.
- Cache leituras quentes (Redis) com invalidação clara; cuidado com cache stale.
- Feche conexões no **graceful shutdown** (veja `node.md`).

## Checklist Banco

- [ ] Tipos de coluna corretos (tempo UTC, dinheiro decimal/inteiro); constraints + FKs no banco.
- [ ] Normalizado por padrão; desnormalização só com motivo medido.
- [ ] Acesso isolado em repository; ORM tipado; SQL cru sempre parametrizado.
- [ ] Sem N+1 (relações carregadas com include/join/DataLoader); nada de query em loop.
- [ ] Índices nas colunas de WHERE/JOIN/ORDER BY/FK; sem superíndice; validado com EXPLAIN.
- [ ] Transações curtas e atômicas; isolation + retry onde há concorrência.
- [ ] Migrations versionadas e reversíveis; sem auto-sync em prod; estratégia expand/contract.
- [ ] Pool dimensionado (pooler em serverless); sem `SELECT *`; paginação keyset; shutdown fecha conexões.

## Fontes

Verificado em jun/2026 na doc do Prisma: técnicas de transação (nested writes,
`$transaction([])`, interativas), isolation levels configuráveis, retry de `P2034` e a
recomendação de **manter transações curtas**.

- Prisma — Transactions: https://www.prisma.io/docs/orm/prisma-client/queries/transactions
- Prisma — Boas práticas de performance / N+1: https://www.prisma.io/docs/orm/prisma-client/queries/query-optimization-performance
- PostgreSQL — Indexes: https://www.postgresql.org/docs/current/indexes.html ·
  Transaction Isolation: https://www.postgresql.org/docs/current/transaction-iso.html
- Use The Index, Luke! (indexação SQL): https://use-the-index-luke.com
- Drizzle ORM: https://orm.drizzle.team/docs · TypeORM: https://typeorm.io