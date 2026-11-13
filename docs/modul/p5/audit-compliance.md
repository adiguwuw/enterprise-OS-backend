# Audit & Compliance

Keputusan blueprint final · Phase 5 · audit-compliance

## Tujuan dan batas
Audit app-level append-only metadata; bukan sertifikasi compliance atau arsip legal anti-tamper.

## PRD/SRS dan acceptance
- Aktor, tindakan, scope organisasi, status lifecycle, dan data sensitivity harus tervalidasi server-side.
- Modul memiliki data canonical yang disebut di ERD; modul lain merujuk ID opaque dan memakai API/event, tanpa query/FK lintas bounded context.
- Mutasi harus atomik, aman diulang bila memakai Idempotency-Key, memakai version/If-Match pada aggregate yang dapat diedit, dan mencatat audit untuk perubahan sensitif.
- Data demo seluruhnya sintetis. Tidak ada asumsi kebijakan/pajak/hukum atau struktur internal perusahaan nyata.

## ERD logis (PostgreSQL)
- AuditRecord(id,occurred_at,actor_id,action,resource_type,resource_id,scope_id,result,reason,request_id,source_ip_hash,metadata_redacted)
- Control(id,code,title,owner_id,status,review_interval_days)
- Evidence(id,control_id,reference_type,reference_id,collected_at,collector_id,checksum)
- AccessReview(id,scope_id,reviewer_id,status,due_at,completed_at)
- AccessReviewItem(id,review_id,assignment_id,decision,comment)

### Relasi, constraint, indeks
- AuditRecord memiliki PK UUID; FK internal sesuai field *_id.
- Control memiliki PK UUID; FK internal sesuai field *_id.
- Evidence memiliki PK UUID; FK internal sesuai field *_id.
- AccessReview memiliki PK UUID; FK internal sesuai field *_id.
- AccessReviewItem memiliki PK UUID; FK internal sesuai field *_id.
- Timestamp simpan UTC timestamptz; tanggal bisnis DATE + timezone IANA bila jadwal/lokasi berpengaruh. Kolom uang NUMERIC + currency ISO, tidak pakai float.
- Terapkan NOT NULL, CHECK (nilai/status/rentang), UNIQUE (kode/natural key dan constraint aktif), FK, index untuk FK + filter/pagination yang dipakai. Hindari cascade delete histori.
- Tambah created_at, updated_at, created_by, updated_by, version pada aggregate mutable. Tabel append-only/history tidak diubah untuk mengoreksi; buat kompensasi/reversal.

## API REST kontrak modul
Prefix /api/v1. JSON, Bearer access token, Problem Details dengan request_id. Koleksi memakai cursor dan batas page size; retryable command memakai Idempotency-Key; update memakai ETag/If-Match.

| Operasi | Endpoint | Ekspektasi |
|---|---|---|
| Create | GET /api/v1/audit?from=&to=&actorId= | Authz server-side, validasi domain, audit bila mutasi |
| Read/search | GET /api/v1/audit/{id} | Authz server-side, validasi domain, audit bila mutasi |
| Update/command | POST /api/v1/compliance/access-reviews | Authz server-side, validasi domain, audit bila mutasi |
| Workflow command | POST /api/v1/compliance/access-reviews/{id}/decisions | Authz server-side, validasi domain, audit bila mutasi |
| Report/read | GET /api/v1/compliance/controls | Authz server-side, validasi domain, audit bila mutasi |

Error umum: 400 schema invalid, 401 token invalid, 403 disallowed, 404 missing/out-of-scope sesuai anti-enumeration, 409 duplicate/state conflict, 412 version stale, 422 aturan bisnis, 429 throttling. Response list {data,next_cursor}; error {type,title,status,code,detail,request_id,field_errors}.

## Alur utama
1. Parse/validate payload; identitas aktor hanya dari token valid, jangan percaya actor/tenant/scope dalam body.
2. Periksa permission resource:action, ABAC scope, relasi target, masa berlaku grant dan state transition.
3. Jalankan command dalam transaksi singkat; cek version dan constraint DB.
4. Simpan audit dan outbox secara atomik bila perubahan event-worthy.
5. Balikkan resource dan version; proses async tidak menahan request.

## TDD — test case wajib
- **AUD-01** sensitive mutation emits redacted audit. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **AUD-02** app role cannot alter audit. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **AUD-03** query time/scope bounded. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **AUD-04** no secret/PII metadata. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **AUD-05** review decision has rationale. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **AUD-06** export bounded, authorized, logged. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.

## Security dan NFR modul
- Default deny; uji positif dan negatif tiap permission/scope. Setiap service tetap melakukan otorisasi meski gateway sudah autentikasi.
- Minimalkan/masking PII; jangan masukkan PII, nilai payroll, token atau isi dokumen ke log, event, metric, trace, URL, error, demo fixtures publik.
- Query parameterized; batas request, response, pagination, timeout; TLS hosted; secret hanya env; log redacted dengan request/correlation ID.
- SLO hanya baseline demo yang diukur, bukan janji produksi. Tulis retention, recovery, replay, migration, dan fallback untuk dependensi bila applicable.

## Definition of Done
Acceptance di atas + semua test ID lulus otomatis; migration dari DB kosong dan upgrade path berhasil; OpenAPI contoh response cocok; permission negative test ada; fixture sintetis; UI state unauthorized/error/loading/empty bila ada UI; README WSL dan dokumentasi diperbarui. Jangan klaim test telah berjalan sebelum benar-benar dijalankan.
