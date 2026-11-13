# Roadmap Development dan Urutan Review

## Review pertama sampai terakhir
1 Product brief + keputusan final (apa produk, batas, free stack).
2 SRS, persona, privacy dan acceptance.
3 TDD dan batas domain/deploy.
4 ERD global dan data ownership.
5 Spesifikasi Phase 1 per modul beserta test case.
6 Kontrak API/event dan test integration.
7 Frontend route/UX/API/test blueprint di repo frontend.
8 Roadmap, DoR/DoD dan baru mulai development.

## Tahap development
0 Bootstrap dua repo di WSL: versi Go/Node dicatat, Compose PostgreSQL+Keycloak, env.example tanpa secret, CI.
1 Phase 1 vertical slice: health → OIDC → principal → org tree → RBAC/scope → employee basics → directory → audit → UI → integration tests.
2 HR: lifecycle, attendance, leave, performance, lalu payroll sandbox paling akhir dengan maker-checker/privacy tests.
3 ERP: budget/expense → journal post/reverse → procurement approval/receipt → asset lifecycle.
4 Operations: project → CRM → document metadata only → collaboration.
5 Platform: outbox/inbox → Rabbit local worker → access review/audit → metrics/logs → tracing/failure recovery.

## Ready/Done
Ready: actor, acceptance Given/When/Then, schema, endpoint, owner, permission, negative tests, UI state, policies known. Done: tests for all scenario IDs pass, migration fresh/upgrade, OpenAPI/ERD match, authz/privacy checks, UI states, docs/WSL steps, no new paid dependency.

## Task template
[ ] Baca spec → [ ] kunci acceptance/contract → [ ] tulis failing tests → [ ] migration/repository → [ ] domain+authz → [ ] handler/OpenAPI → [ ] integration/security tests → [ ] FE journey → [ ] jalankan suite di WSL → [ ] update docs.

Hentikan task jika butuh kebijakan hukum/HR yang tak tersedia, migration berisiko menghapus data, scope authorization tak dapat dibuktikan, atau dependency berbayar muncul; catat keputusan dan pilih behavior aman.
