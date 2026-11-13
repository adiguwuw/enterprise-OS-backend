# Accounting

Keputusan blueprint final · Phase 3 · accounting

## Tujuan dan batas
Ledger double-entry simulasi; posted journal immutable.

## PRD/SRS dan acceptance
- Aktor, tindakan, scope organisasi, status lifecycle, dan data sensitivity harus tervalidasi server-side.
- Modul memiliki data canonical yang disebut di ERD; modul lain merujuk ID opaque dan memakai API/event, tanpa query/FK lintas bounded context.
- Mutasi harus atomik, aman diulang bila memakai Idempotency-Key, memakai version/If-Match pada aggregate yang dapat diedit, dan mencatat audit untuk perubahan sensitif.
- Data demo seluruhnya sintetis. Tidak ada asumsi kebijakan/pajak/hukum atau struktur internal perusahaan nyata.

## ERD logis (PostgreSQL)
- Account(id,code,name,account_type,parent_id,active)
- FiscalPeriod(id,period_key,start_date,end_date,status)
- Journal(id,journal_no,source_type,source_id,period_id,status,created_by,posted_at,reversal_of_id)
- JournalLine(id,journal_id,line_no,account_id,debit,credit,currency,description)
- PostingBatch(id,source_event_id,status,attempts)

### Relasi, constraint, indeks
- Account memiliki PK UUID; FK internal sesuai field *_id.
- FiscalPeriod memiliki PK UUID; FK internal sesuai field *_id.
- Journal memiliki PK UUID; FK internal sesuai field *_id.
- JournalLine memiliki PK UUID; FK internal sesuai field *_id.
- PostingBatch memiliki PK UUID; FK internal sesuai field *_id.
- Timestamp simpan UTC timestamptz; tanggal bisnis DATE + timezone IANA bila jadwal/lokasi berpengaruh. Kolom uang NUMERIC + currency ISO, tidak pakai float.
- Terapkan NOT NULL, CHECK (nilai/status/rentang), UNIQUE (kode/natural key dan constraint aktif), FK, index untuk FK + filter/pagination yang dipakai. Hindari cascade delete histori.
- Tambah created_at, updated_at, created_by, updated_by, version pada aggregate mutable. Tabel append-only/history tidak diubah untuk mengoreksi; buat kompensasi/reversal.

## API REST kontrak modul
Prefix /api/v1. JSON, Bearer access token, Problem Details dengan request_id. Koleksi memakai cursor dan batas page size; retryable command memakai Idempotency-Key; update memakai ETag/If-Match.

| Operasi | Endpoint | Ekspektasi |
|---|---|---|
| Create | POST /api/v1/accounting/journals | Authz server-side, validasi domain, audit bila mutasi |
| Read/search | POST /api/v1/accounting/journals/{id}/post | Authz server-side, validasi domain, audit bila mutasi |
| Update/command | POST /api/v1/accounting/journals/{id}/reverse | Authz server-side, validasi domain, audit bila mutasi |
| Workflow command | GET /api/v1/accounting/ledger?accountId=&from=&to= | Authz server-side, validasi domain, audit bila mutasi |
| Report/read | GET /api/v1/accounting/trial-balance?period= | Authz server-side, validasi domain, audit bila mutasi |

Error umum: 400 schema invalid, 401 token invalid, 403 disallowed, 404 missing/out-of-scope sesuai anti-enumeration, 409 duplicate/state conflict, 412 version stale, 422 aturan bisnis, 429 throttling. Response list {data,next_cursor}; error {type,title,status,code,detail,request_id,field_errors}.

## Alur utama
1. Parse/validate payload; identitas aktor hanya dari token valid, jangan percaya actor/tenant/scope dalam body.
2. Periksa permission resource:action, ABAC scope, relasi target, masa berlaku grant dan state transition.
3. Jalankan command dalam transaksi singkat; cek version dan constraint DB.
4. Simpan audit dan outbox secara atomik bila perubahan event-worthy.
5. Balikkan resource dan version; proses async tidak menahan request.

## TDD — test case wajib
- **ACC-01** debit=credit per currency policy. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **ACC-02** closed period rejects post. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **ACC-03** posted journal immutable. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **ACC-04** reversal mirrors original. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **ACC-05** source event idempotent. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **ACC-06** ledger pagination/authz scoped. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.

## Security dan NFR modul
- Default deny; uji positif dan negatif tiap permission/scope. Setiap service tetap melakukan otorisasi meski gateway sudah autentikasi.
- Minimalkan/masking PII; jangan masukkan PII, nilai payroll, token atau isi dokumen ke log, event, metric, trace, URL, error, demo fixtures publik.
- Query parameterized; batas request, response, pagination, timeout; TLS hosted; secret hanya env; log redacted dengan request/correlation ID.
- SLO hanya baseline demo yang diukur, bukan janji produksi. Tulis retention, recovery, replay, migration, dan fallback untuk dependensi bila applicable.

## Definition of Done
Acceptance di atas + semua test ID lulus otomatis; migration dari DB kosong dan upgrade path berhasil; OpenAPI contoh response cocok; permission negative test ada; fixture sintetis; UI state unauthorized/error/loading/empty bila ada UI; README WSL dan dokumentasi diperbarui. Jangan klaim test telah berjalan sebelum benar-benar dijalankan.
