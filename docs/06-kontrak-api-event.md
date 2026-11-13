# Kontrak API dan Event

## HTTP
Canonical prefix /api/v1; JSON; OpenAPI 3.1; Bearer access token OIDC (bukan ID token). Headers: Authorization, X-Request-ID, Idempotency-Key untuk command retryable, If-Match; respons resource mutable punya ETag. List: {data,next_cursor}; errors: {type,title,status,code,detail,request_id,field_errors}. ISO-8601 UTC, uang decimal string + ISO currency.

Setiap module spec menetapkan endpoint baseline. Sebelum implementation, tiap path harus punya schema, examples, permission scope, status codes, pagination/idempotency/concurrency semantics, event side effect. Lint OpenAPI dan cek breaking change di CI; FE mengikuti kontrak tersebut.

## Event envelope Phase 5
```json
{"id":"uuid","type":"employee.assignment.changed","version":1,"occurred_at":"RFC3339","producer":"employee-management","aggregate":{"type":"employee","id":"uuid","version":7},"correlation_id":"uuid","causation_id":null,"data":{},"metadata":{"schema_uri":"..."}}
```

Tidak memasukkan credential, private HR, payroll amount, isi dokumen, raw email/token. Producer memiliki schema; optional additive fields compatible; breaking change versi baru.

## Delivery
Write outbox satu transaksi dengan business update. Publisher confirm broker sebelum published. Consumer commit inbox unique(consumer,event_id) bersama side effect. Semantik at-least-once; order hanya per aggregate version. Retry bounded+jitter, DLQ, replay permissioned+audited. Tidak klaim exactly once.

## Event awal
organization.unit.changed, iam.principal.disabled, employee.created/assignment.changed/separated, leave.request.approved, finance.expense.approved, procurement.order.approved, asset.assigned, project.task.changed, document.metadata.changed. Mulai event hanya saat consumer/projection konkret diperlukan.
