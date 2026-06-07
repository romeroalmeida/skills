# Node.js (backend) — Clean Code

Fundamentos de runtime para serviços Node em TS, independente do framework (Nest, Express,
Fastify). Combine com `typescript.md`, `api-design.md`, `database.md` e `security.md`.

## 1. Modelo de execução (não bloqueie o event loop)

- Node é **single-thread** para seu código + I/O assíncrono. Bloquear o event loop trava
  **todas** as requisições.
- **Nada de APIs síncronas** no hot path (`fs.readFileSync`, `crypto.*Sync`, JSON gigante,
  regex catastrófica, loop pesado). Trabalho **CPU-bound** → `worker_threads` (ou fila/serviço
  separado). Escale I/O-bound com `cluster`/múltiplas instâncias atrás de um balanceador.

## 2. Assíncrono correto

- `async/await`; **sem floating promises** (ligue `no-floating-promises`). Erros em promise
  não-aguardada viram `unhandledRejection`.
- Paralelize independentes com `Promise.all`; tolere falhas parciais com `allSettled`.
- Propague **`AbortSignal`** para timeout/cancelamento de fetch/queries.
- Não use `async` em `forEach` (não espera) — `for...of` com `await` ou `Promise.all(map)`.

## 3. Streams e backpressure

- Para arquivos/respostas/ETL grandes, use **streams** (`pipeline`) em vez de carregar tudo
  na memória. `stream/promises.pipeline` cuida de erro e backpressure.
- Não acumule buffers ilimitados (risco de OOM); respeite o ritmo do consumidor.

## 4. Tratamento de erros

- Distinga **erro operacional** (esperado: input inválido, timeout, 404) de **bug de
  programação** (inesperado). Trate operacionais; deixe bugs falharem visível.
- `try/catch` em torno de `await`; em Express, encaminhe ao middleware de erro (ou use Nest
  exception filters). Nunca `catch` vazio.
- Registre handlers de último recurso para **`unhandledRejection`** e **`uncaughtException`**:
  logue e **encerre o processo** de forma controlada (estado pode estar corrompido) — deixe o
  orquestrador reiniciar. Não os use para "continuar como se nada fosse".

## 5. Configuração (12-Factor)

- Config por **variáveis de ambiente**, **validadas no boot** (zod/envalid) — o processo
  falha rápido se faltar/estiver inválida. Sem `process.env.X` espalhado: centralize num
  módulo tipado.
- Segredos fora do código (veja `security.md`). Paridade dev/prod; sem config hardcoded.

## 6. Graceful shutdown

- Em **SIGTERM/SIGINT**: pare de aceitar novas conexões, **drene** as em andamento, feche
  pool de DB/filas/sockets, então saia. Defina timeout de força. Essencial em
  containers/k8s para deploys sem erro 5xx.

```ts
process.on("SIGTERM", async () => {
  server.close();              // para de aceitar novas
  await pool.end();            // fecha conexões
  process.exit(0);
});
```

## 7. Observabilidade

- **Log estruturado** (JSON) com **pino** — sem `console.log` em produção. Inclua nível,
  timestamp e **correlation/request id**; **redija** PII/segredos.
- **Métricas** (Prometheus/OpenTelemetry) e **tracing distribuído** (OpenTelemetry) nos
  pontos de I/O. **Health checks** (`/health`, readiness/liveness) para o orquestrador.

## 8. Módulos, deps e runtime

- **ESM** + TS; `package.json` com `"type": "module"`, `engines.node`, e `exports` se for lib.
- Rode **Node LTS** atualizado (correções de segurança). Lockfile commitado; deps mínimas.
- Builds reproduzíveis; `NODE_ENV=production` em produção.

## 9. Performance

- Reuse conexões (pool de DB/HTTP keep-alive); **cache** leituras quentes (Redis) com
  invalidação clara. Offload de trabalho pesado para **filas** (BullMQ) — não faça na request.
- Meça antes de otimizar (clinic.js, `--prof`, OpenTelemetry). Cuidado com vazamento de
  memória (listeners/closures retidos).

## Checklist Node

- [ ] Event loop livre: sem APIs síncronas no hot path; CPU-bound em worker/fila.
- [ ] Sem floating promises; paralelismo com `Promise.all`/`allSettled`; `AbortSignal` em I/O.
- [ ] Streams + `pipeline` para grandes volumes; sem buffers ilimitados.
- [ ] Erros operacionais tratados; `unhandledRejection`/`uncaughtException` logam e encerram.
- [ ] Config por env validada no boot; segredos fora do código; módulo de config tipado.
- [ ] Graceful shutdown em SIGTERM (drena + fecha pool) com timeout.
- [ ] Log estruturado (pino) com correlação e redação; métricas/tracing/health checks.
- [ ] Node LTS atual; ESM; lockfile; `NODE_ENV=production`; deps mínimas.
- [ ] Conexões reusadas; trabalho pesado em fila; sem vazamento de memória.

## Fontes

- Node.js — Documentação (Async, Streams, Worker Threads, Cluster, Process):
  https://nodejs.org/docs/latest/api/
- Node.js — Guia "Don't Block the Event Loop":
  https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop
- The Twelve-Factor App (config, paridade, processos): https://12factor.net
- OpenTelemetry (tracing/métricas) JS: https://opentelemetry.io/docs/languages/js/
- pino (logging estruturado): https://getpino.io