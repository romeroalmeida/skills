# NestJS — Clean Code (Nest 10/11)

Backend Node com Nest + TS. Combine com `typescript.md` e `architecture.md`.

## 1. Arquitetura em camadas e módulos

- Organize **por feature/domínio** (um módulo por contexto), não por tipo técnico.
- Camadas com fluxo de dependência **para dentro**:
  **Controller** (HTTP/transport) → **Service** (regra de negócio) → **Repository**
  (persistência). Controllers e repositórios são finos; a regra vive no service.

```
src/
  users/
    users.controller.ts   # só HTTP: rota, status, delega
    users.service.ts       # regra de negócio
    users.repository.ts     # acesso a dados (TypeORM/Prisma)
    dto/                    # create-user.dto.ts, update-user.dto.ts
    entities/
    users.module.ts
```

- **Controller é magro.** Sem regra de negócio, sem query de banco. Recebe DTO, chama
  service, devolve resultado. Se há `if` de negócio no controller, mova para o service.

## 2. Injeção de dependência

- Dependa de **abstrações** via DI; não instancie (`new`) serviços manualmente.
- Constructor injection com `private readonly`. Para trocar implementações, use tokens/
  interfaces (`@Inject(TOKEN)`), não classes concretas em fronteiras importantes.

```ts
@Injectable()
export class UsersService {
  constructor(private readonly repo: UsersRepository) {}
}
```

- Respeite o escopo (singleton por padrão); use request-scoped só quando necessário
  (tem custo de performance).

## 3. DTOs e validação (borda HTTP)

- **Todo input** entra por um **DTO com `class-validator`** + `ValidationPipe` global
  com `whitelist` (remove campos não declarados) e `forbidNonWhitelisted`.

```ts
// main.ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,                 // converte payload na instância do DTO
  transformOptions: { enableImplicitConversion: false },
}));

// create-user.dto.ts
export class CreateUserDto {
  @IsEmail() email!: string;
  @IsString() @MinLength(8) password!: string;
  @IsOptional() @IsInt() @Min(0) age?: number;
}
```

- DTO de entrada ≠ entidade do banco ≠ DTO de resposta. **Nunca** retorne a entidade
  crua (vaza `passwordHash`, campos internos). Use DTO de resposta / serialização
  (`ClassSerializerInterceptor` + `@Exclude`).
- Nunca confie em tipo estático para input externo — a validação em runtime é o contrato.

## 4. Building blocks no lugar certo

| Necessidade                                  | Use            |
| -------------------------------------------- | -------------- |
| Transformar/validar input                    | **Pipe**       |
| Autenticação/autorização (pode acessar?)     | **Guard**      |
| Lógica transversal (logging, cache, mapear)  | **Interceptor**|
| Traduzir exceção → resposta HTTP             | **Exception Filter** |
| Metadados de rota (roles, etc.)              | **Decorator + Reflector** |

- Não enfie auth no service nem logging espalhado: use guards/interceptors (cross-cutting).

## 5. Erros e respostas

- Lance **`HttpException`** e subclasses (`NotFoundException`, `BadRequestException`,
  `ForbiddenException`, `ConflictException`) — nada de `throw new Error("...")` cru chegando
  ao cliente.
- Erros de domínio (camada de negócio) são exceções próprias, traduzidas para HTTP por um
  **Exception Filter** — assim o service não conhece HTTP.
- Nunca vaze stack trace / detalhes internos na resposta. Log estruturado no servidor.

## 6. Configuração

- **`ConfigModule`** + validação de schema (Joi ou zod) que **falha no boot** se faltar env.
- Nada de `process.env.X` espalhado pelo código — acesse via `ConfigService` tipado.
- Use `registerAsync`/`forRootAsync` para módulos que dependem de config (DB, JWT).

## 7. Persistência

- Repositório isola a regra de negócio do ORM (TypeORM/Prisma). O service não monta query
  bruta nem conhece detalhes do ORM além do necessário.
- **Transações** para operações multi-passo que devem ser atômicas.
- Migrations versionadas; nunca `synchronize: true` em produção.
- Cuidado com **N+1**: carregue relações de forma explícita/eficiente.

## 8. Assíncrono e resiliência

- Tudo I/O é `async`; sem floating promises (ligue a regra). `Promise.all` para chamadas
  independentes.
- Timeouts e cancelamento em chamadas externas; retry/backoff com cautela (idempotência).

## 9. Segurança (backend)

- `helmet`, CORS restrito, **rate limiting** (`@nestjs/throttler`).
- Senhas com `argon2`/`bcrypt`; **nunca** logue segredos/PII/tokens.
- Autenticação (JWT/sessão) via Guards; autorização por roles/policies (`@Roles` + Guard).
- Queries parametrizadas (ORM já faz) — nunca concatene input em SQL. Valide tudo na borda.
- Princípio do menor privilégio em credenciais de DB/serviços.

## 10. Observabilidade e testes

- Logger estruturado (pino/nestjs-pino) com correlação de request; sem `console.log`.
- Health checks (`@nestjs/terminus`).
- Service testável com unit tests (mock do repo); controllers e fluxo com e2e
  (`Test.createTestingModule`). Veja `testing.md`.

## Checklist NestJS

- [ ] Módulo por feature; controller magro, regra no service, dados no repository.
- [ ] DI por constructor (`private readonly`); sem `new` de serviços.
- [ ] `ValidationPipe` global com `whitelist`; todo input num DTO `class-validator`.
- [ ] Entidade nunca retornada crua; DTO/serialização de resposta sem campos sensíveis.
- [ ] Cross-cutting em guards/interceptors/filters, não espalhado nos services.
- [ ] `HttpException` adequadas; exceções de domínio traduzidas por filter; sem stack vazada.
- [ ] `ConfigModule` validando env no boot; sem `process.env` solto.
- [ ] Transações onde precisa; migrations; sem `synchronize` em prod; sem N+1.
- [ ] helmet + CORS + rate limit; senhas com hash; nada sensível logado.
- [ ] Services com unit tests; fluxo crítico com e2e.
