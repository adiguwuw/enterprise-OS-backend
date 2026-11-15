# Roadmap Development dan Urutan Review

## Urutan review dokumen
1. Keputusan final + product brief: tujuan, audience, portofolio/fiksi, stack gratis dan scope.
2. SRS: requirements, attendance privacy, failure/exception policies.
3. TDD + ADR: module boundaries, Vercel constraints, upload trust design.
4. ERD global + ownership dan invariants.
5. Spesifikasi tiap modul dan acceptance tests (mulai Phase 1).
6. Kontrak OpenAPI/event dan test matrix.
7. Blueprint frontend: routes, states, API, tests.
8. Roadmap dan coding-agent guardrails; development hanya setelah review keputusan terbuka yang menghalangi slice selesai.

## Urutan implementasi
0. Reconcile/reset local worktree only after explicit review; bootstrap Node LTS + Express 5 TS + React TS in WSL; lockfile, env examples, Compose PostgreSQL/Keycloak, free-provider setup, CI.
1. Foundation vertical slices: health/config → OIDC → principal → org structure → RBAC/ABAC → employee basics → directory → audit.
2. HR: employee lifecycle → attendance policy/config + remote-work approval + evidence upload/punch → leave → performance → payroll sandbox terakhir.
3. ERP: expense/budget → journal posting/reversal → procurement approval/receipt → asset lifecycle.
4. Operations: project → CRM → document metadata → collaboration.
5. Platform: audit review/retention → outbox/inbox and RabbitMQ only for real consumer → monitoring/logs → tracing.

## Attendance readiness gate
Sebelum coding kunci policy radius GPS, accuracy threshold, shifts/timezone, exceptional punch, retention and reviewer access. Implement IP/CIDR trusted proxy path and private upload direct-to-storage with tests. Free storage quota exhaustion must fail safely.

## Ready / Done
Ready: actor, Given/When/Then IDs, schema, endpoint, ownership, permission, negative tests, UI state, policy known. Done: tests pass, migration fresh/upgrade, OpenAPI/ERD match, authz/privacy controls and redaction verified, WSL steps, no paid dependency.
