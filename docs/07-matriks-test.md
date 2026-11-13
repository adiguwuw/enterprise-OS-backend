# Matriks Test Integrasi

| ID | Skenario | Hasil wajib |
|---|---|---|
| INT-01 | OIDC login lalu directory | principal valid; field terlihat saja |
| INT-02 | issuer/audience invalid atau JWKS gagal | fail closed, tanpa bypass |
| INT-03 | manager lihat unit sendiri dan luar scope | scope benar; tak ada count leak |
| INT-04 | HR buat employee+assignment valid | transaksi, audit dan projection konsisten |
| INT-05 | update paralel stale If-Match | 412, tak ada state parsial |
| INT-06 | command retry same idempotency key | satu efek domain |
| INT-07 | broker kirim event dua kali | inbox mencegah duplikasi |
| INT-08 | broker mati atau consumer gagal | outbox pending/retry/DLQ; tak hilang |
| INT-09 | role dicabut lalu request baru | deny; stale allow tidak lewat batas cache |
| INT-10 | payload actor/scope palsu | diabaikan/ditolak; identity dari token |
| INT-11 | SQL/HTML pada search | parameterized, escaped, cursor bounded |
| INT-12 | log/trace payroll/profile | tidak berisi PII/token/nominal sensitif |
| INT-13 | migration fresh dan upgrade | constraints/history benar |
| INT-14 | principal disable/employee separated | akses & directory sesuai lifecycle |
| INT-15 | keyboard/mobile/loading/empty/error | operable dan feedback benar |
| INT-16 | kuota/free DB limit | degrade tanpa paid upgrade; local run tersedia |

Test layers: unit domain/policy, component handler, PostgreSQL integration+constraints, OpenAPI/event contract, E2E journey, security (authz matrix, IDOR, injection, CSRF/XSS/CORS, rate/body limit, secret scan). Merge gate format/lint/test/migration/OpenAPI/docs. Catat hasil aktual.
