# Segurança — Clean Code (front & back)

Segurança é requisito transversal. Use o **OWASP Top 10:2025** como mapa de risco. Combine
com `nestjs.md`, `api-design.md`, `database.md` e `nextjs.md`.

## OWASP Top 10:2025 (mapa de risco)

| #   | Categoria                              | Mitigação principal aqui                  |
| --- | -------------------------------------- | ----------------------------------------- |
| A01 | Broken Access Control                  | §2 authz no servidor, deny-by-default     |
| A02 | Security Misconfiguration              | §6 headers, config segura, sem defaults   |
| A03 | Software Supply Chain Failures         | §7 dependências, lockfile, SCA            |
| A04 | Cryptographic Failures                 | §5 hashing/cripto, TLS, segredos          |
| A05 | Injection                              | §3 validação + queries parametrizadas     |
| A06 | Insecure Design                        | threat modeling, princípios desde o design|
| A07 | Authentication Failures                | §4 authn forte, sessão/token corretos     |
| A08 | Software or Data Integrity Failures    | §7 integridade de pipeline/deps           |
| A09 | Security Logging & Alerting Failures   | §8 log/auditoria sem vazar, alertas       |
| A10 | Mishandling of Exceptional Conditions  | tratar erro sem vazar; fail-safe          |

> Broken Access Control segue **#1** em 2025; Security Misconfiguration subiu para **#2**;
> "Software Supply Chain Failures" entrou em **#3**.

## 1. Princípios

- **Menor privilégio** (usuário de DB, tokens, escopos). **Defesa em profundidade**.
  **Deny by default**. **Fail-safe**: erro fecha o acesso, não abre. **Nunca confie no
  cliente** — toda decisão de segurança é no servidor.

## 2. Autorização (A01 — o risco #1)

- Verifique permissão **em cada request**, no servidor, perto do recurso — não esconda só
  no front. Cheque **ownership** ("este pedido é deste usuário?"), não só o papel.
- Cuidado com **IDOR**: `GET /orders/123` deve checar se 123 pertence ao requisitante.
- Nega por padrão; valide em Server Actions/Route Handlers (são endpoints públicos).

## 3. Injeção e validação de entrada (A05)

- **Valide e tipifique todo input** na borda (zod/class-validator). Allowlist > denylist.
- **SQL/NoSQL/command**: sempre **parametrizado**/ORM; nunca concatene input. Sem `eval`,
  sem montar shell com input.
- **XSS** (front): evite `dangerouslySetInnerHTML`; se inevitável, **sanitize** (DOMPurify).
  Aplique **CSP**. React escapa por padrão — não desfaça isso.

## 4. Autenticação (A07)

- Senhas com hash forte e lento: **argon2id** (preferido) ou **bcrypt** — nunca SHA/MD5,
  nunca em texto puro. Compare em tempo constante.
- Sessões: cookie **`HttpOnly` + `Secure` + `SameSite`**; rotacione no login; expire/renove.
  JWT: assinatura forte, `exp` curto + refresh; valide `aud`/`iss`; não guarde dado sensível
  no payload. Considere MFA e proteção contra brute force (lockout/rate limit).

## 5. Criptografia e segredos (A04)

- **TLS** em tudo (HSTS). Cripto autenticada (AES-GCM); não invente cripto.
- **Segredos fora do código e do cliente**: variáveis de ambiente / secret manager; nunca
  commitados; rotacionáveis. No front, só `NEXT_PUBLIC_*`/equivalente não-sensível.

## 6. Configuração e headers (A02)

- **Helmet**/headers: `Content-Security-Policy`, `Strict-Transport-Security`,
  `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `X-Frame-Options`/CSP frame-ancestors.
- **CORS restrito** a origens conhecidas (não `*` com credenciais).
- Desligue stack trace/listagem em produção; sem credenciais default; superfície mínima.

## 7. Dependências e integridade (A03/A08)

- **Lockfile** commitado; `npm audit`/Dependabot/Renovate; SCA no CI. Atualize libs com CVE.
- Minimize dependências; verifique pacotes (typosquatting). CI/CD com permissões mínimas e
  artefatos íntegros.

## 8. Logging e tratamento de erro (A09/A10)

- Log estruturado de eventos de segurança (login, falha de authz) **sem PII/segredos/tokens**.
  Tenha **alertas**. Não logue corpo de request sensível.
- Trate exceções de forma previsível: resposta genérica ao cliente (Problem Details), detalhe
  só no log do servidor. Erro inesperado → fail-safe, não exponha internals.

## 9. Específicos de front / Next

- Server Actions e Route Handlers são **públicos**: authn + authz + validação **dentro** deles.
- Nada sensível em código de cliente; use `server-only`. Cuidado com dados serializados
  Server→Client. Veja `nextjs.md`.

## Checklist Segurança

- [ ] Authz verificada no servidor em cada request, com ownership (sem IDOR); deny-by-default.
- [ ] Todo input validado/allowlisted; queries parametrizadas; XSS mitigado (CSP, sanitização).
- [ ] Senhas com argon2id/bcrypt; sessão/JWT corretos (HttpOnly/Secure/SameSite, exp curto); MFA/anti-brute-force.
- [ ] TLS/HSTS; cripto autenticada; segredos fora do código/cliente e rotacionáveis.
- [ ] Headers de segurança (CSP/HSTS/nosniff) + CORS restrito; sem defaults/stack em prod.
- [ ] Lockfile + audit/SCA no CI; deps minimizadas e atualizadas; pipeline com menor privilégio.
- [ ] Log de segurança sem PII/segredos + alertas; erros tratados sem vazar internals.
- [ ] (Front/Next) Server Actions/handlers com authn+authz+validação; nada sensível no cliente.

## Fontes

Lista verificada na fonte oficial em jun/2026: **OWASP Top 10:2025** (A01 Broken Access
Control, A02 Security Misconfiguration, A03 Software Supply Chain Failures, A04 Cryptographic
Failures, A05 Injection, A06 Insecure Design, A07 Authentication Failures, A08 Software or
Data Integrity Failures, A09 Security Logging & Alerting Failures, A10 Mishandling of
Exceptional Conditions).

- OWASP — Top 10:2025: https://owasp.org/Top10/2025/
- OWASP — Cheat Sheet Series (Auth, Password Storage, JWT, etc.): https://cheatsheetseries.owasp.org
- OWASP — ASVS (verificação): https://owasp.org/www-project-application-security-verification-standard/
- MDN — Web Security / CSP: https://developer.mozilla.org/en-US/docs/Web/Security
- NestJS — Security (helmet, CORS, throttler): https://docs.nestjs.com/security/helmet