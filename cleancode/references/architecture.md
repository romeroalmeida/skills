# Arquitetura — SOLID, camadas e organização (TS/JS)

Princípios transversais (front e back). Aplique com bom senso — são heurísticas para
reduzir acoplamento e facilitar mudança, não dogmas.

## 1. SOLID em TypeScript

- **S — Single Responsibility.** Um módulo/classe/função tem **uma razão para mudar**. Se
  você descreve o que faz com "e" (valida *e* salva *e* envia email), separe.
- **O — Open/Closed.** Aberto para extensão, fechado para modificação. Em vez de crescer um
  `switch` a cada novo caso, use polimorfismo/estratégia (map de handlers, interface).

```ts
// ❌ cresce a cada tipo novo
function area(s: Shape) {
  switch (s.kind) { case "circle": ...; case "square": ...; }
}
// ✅ cada forma sabe sua área; novo tipo não toca código existente
interface Shape { area(): number; }
```

- **L — Liskov.** Subtipos devem ser substituíveis sem quebrar expectativas (sem lançar
  "não suportado", sem enfraquecer contratos). Em TS, isso se traduz em respeitar a
  variância e não mentir nos tipos.
- **I — Interface Segregation.** Interfaces pequenas e específicas. Não force quem consome a
  depender de métodos que não usa. Prefira várias interfaces focadas a uma "deus".
- **D — Dependency Inversion.** Dependa de **abstrações**, não de implementações concretas.
  Módulos de alto nível (regra de negócio) não importam detalhes (DB, HTTP, SDK) — recebem-nos
  por injeção/parâmetro. Isso torna o núcleo testável e substituível.

## 2. Regra de dependência (camadas)

Dependências apontam **para dentro**, em direção ao domínio. Detalhes externos
(framework, banco, UI, APIs) dependem do núcleo, nunca o contrário.

```
   UI / HTTP / CLI        ← detalhes (trocáveis)
        ↓
   Aplicação (casos de uso / services)
        ↓
   Domínio (regras, entidades)   ← núcleo puro, sem import de framework
        ↑
   Infra (DB, APIs) implementa as interfaces que o domínio define
```

- O **domínio não importa** React, Nest, ORM, axios. Lógica de negócio é TS puro,
  testável sem subir framework nem banco.
- **Ports & adapters (lite):** o núcleo declara a interface (port: `UserRepository`); a
  infra fornece o adapter (`PrismaUserRepository`). Inverte a dependência e isola o I/O.
- Não precisa de "clean architecture" completa em todo projeto — adote a profundidade
  proporcional ao tamanho/vida do sistema (YAGNI).

## 3. Organização por feature (não por tipo)

Agrupe por **domínio**, mantendo junto o que muda junto (alta coesão):

```
src/
  features/
    billing/        # tudo de billing: ui, lógica, dados, tipos, testes
      components/
      api.ts
      billing.service.ts
      types.ts
      billing.test.ts
    auth/
  shared/           # genuinamente reutilizável (ui kit, utils, libs)
  lib/              # wrappers de libs externas (client http, db)
```

- Evite pastas globais gigantes (`components/`, `utils/`, `helpers/`) que viram lixeira.
- `shared` só para o que é **realmente** transversal. Na dúvida, deixe dentro da feature
  até a duplicação provar que vale extrair (regra do três).
- Controle o sentido das dependências entre features: evite ciclos; comunicação entre
  features por contratos explícitos, não importando internals umas das outras.

## 4. Composição sobre herança

- Prefira **compor** comportamento (funções, hooks, providers, mixins de objeto) a herdar
  hierarquias profundas. Herança acopla forte e é rígida; composição é flexível e testável.

## 5. Abstração na hora certa

- **Regra do três:** abstraia na terceira repetição, não na primeira. Abstração prematura
  é mais cara que duplicação.
- A abstração precisa ter **um nome honesto** e esconder complexidade real — se ela só
  renomeia o que já era claro, remova.
- **Acoplamento baixo, coesão alta**: peça poucos detalhes (Lei de Deméter — não navegue
  `a.b.c.d`), exponha pouco, agrupe o que muda junto.

## 6. Fronteiras e contratos

- Defina **tipos/contratos explícitos** nas fronteiras entre camadas e features (DTOs,
  interfaces de port). A fronteira é onde você valida dados externos em runtime.
- Mantenha modelos separados por contexto: o modelo de UI, o de domínio e o de persistência
  podem (e costumam) divergir — não force um único tipo a servir a todos.

## 7. Heurísticas finais

- **KISS / YAGNI:** a solução mais simples que resolve o problema atual. Não construa para
  requisitos imaginários.
- **Custo de mudança:** boa arquitetura se mede por quão barato é alterar o comportamento.
  Se uma mudança simples exige tocar muitos arquivos não relacionados, o acoplamento está alto.
- **Decisões reversíveis primeiro:** adie o que é difícil de reverter; mantenha detalhes
  (DB, provider, framework) na borda para trocar barato.

## Checklist Arquitetura

- [ ] Cada módulo tem uma responsabilidade/uma razão para mudar.
- [ ] Regra de negócio não importa framework/ORM/HTTP (núcleo puro e testável).
- [ ] Dependências apontam para o domínio; I/O isolado atrás de interfaces (ports/adapters).
- [ ] Organização por feature; `shared` só para o genuinamente transversal; sem ciclos.
- [ ] Composição preferida a herança profunda.
- [ ] Sem abstração prematura (regra do três); abstrações com nome honesto.
- [ ] Baixo acoplamento, alta coesão; contratos explícitos nas fronteiras.
- [ ] Complexidade proporcional ao problema (KISS/YAGNI).

## Fontes

Verificado em jun/2026: "The Clean Architecture" e a Regra de Dependência (dependências
apontam para dentro).

- Robert C. Martin — The Clean Architecture (Dependency Rule):
  https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- Alistair Cockburn — Hexagonal Architecture (Ports & Adapters), originador do padrão:
  https://alistair.cockburn.us/hexagonal-architecture/
- Martin Fowler — catálogo de padrões e refatoração (ex.: Law of Demeter): https://martinfowler.com
- SOLID — princípios de design OO (Robert C. Martin): https://en.wikipedia.org/wiki/SOLID
- MDN/refactoring.guru — composição vs herança: https://refactoring.guru/design-patterns
