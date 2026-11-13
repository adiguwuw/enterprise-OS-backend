# TDD / Technical Design Description

## Komponen
Browser SPA React/TS; API gateway/Go HTTP; domain package per konteks; PostgreSQL; OIDC IdP. Phase 5 menambah outbox publisher, RabbitMQ consumer lokal dan telemetry. Frontend repo terpisah.

## Batas layanan
Istilah “service” pada roadmap bermakna domain module, bukan deployable microservice. Gunakan modular monolith untuk biaya dan operabilitas. Company Structure pakai Organization Service; Directory adalah projection Employee; IAM mengelola policy tetapi tiap domain enforce; broker, monitoring, tracing adalah platform capabilities.

## Deploy dan local dev
Di WSL Docker Compose menjalankan Postgres, Keycloak; RabbitMQ hanya Phase 5. Hosted: Vercel Hobby untuk SPA + Go HTTP function; Neon Free DB; Auth0 Free OIDC; CloudAMQP free optional. Vercel bukan worker persistent. Tanpa Redis, provider berbayar, upgrade atau billing.

## API dan data
/ api/v1 (tanpa spasi; canonical path /api/v1), OpenAPI 3.1; Bearer access token; cursor pagination; standard error; Idempotency-Key dan ETag/If-Match. Data PostgreSQL dibatasi schema/domain owner. FK hanya dalam bounded context; cross-context opaque IDs + APIs/events. DB transaction menulis aggregate dan audit/outbox; no network call dalam transaction.

## Test-driven development
Tulis scenario ID dari module spec sebagai test yang gagal dahulu; unit untuk policy/domain, integration real PostgreSQL + migration, contract OpenAPI/event, E2E per journey, security negative authz/IDOR/injection. Jalankan tests terkait tiap perubahan, full suite sebelum merge. Jangan update status acceptance tanpa hasil.

## Struktur kode target
cmd/api; cmd/worker (Phase 5 lokal); internal/<bounded-context>/{domain,application,ports,adapter}; pkg/platform; migrations/<context>; api/openapi; tests/integration; deploy/docker-compose.yml; docs/adr. Frontend repo tersendiri.

## Reliability dan security
Timeout/bounded retry; fail closed; request/correlation ID; readiness berbeda liveness. Outbox/inbox at-least-once; dedupe consumer unique key; DLQ + replay auditable. No sensitive data in events/logs/traces. Setiap keputusan lintas boundary dicatat ADR dengan cost, security, migration impact.
