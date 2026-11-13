# Asset Management

Keputusan blueprint final · Phase 3 · asset-management

## Tujuan dan batas
Lifecycle aset demo; bukan depresiasi fiskal resmi.

## PRD/SRS dan acceptance
- Aktor, tindakan, scope organisasi, status lifecycle, dan data sensitivity harus tervalidasi server-side.
- Modul memiliki data canonical yang disebut di ERD; modul lain merujuk ID opaque dan memakai API/event, tanpa query/FK lintas bounded context.
- Mutasi harus atomik, aman diulang bila memakai Idempotency-Key, memakai version/If-Match pada aggregate yang dapat diedit, dan mencatat audit untuk perubahan sensitif.
- Data demo seluruhnya sintetis. Tidak ada asumsi kebijakan/pajak/hukum atau struktur internal perusahaan nyata.

## ERD logis (PostgreSQL)
- AssetCategory(id,code,name,depreciation_method,default_life_months)
- Asset(id,asset_tag,name,category_id,serial_no,acquired_on,cost,currency,status,location_id)
- AssetAssignment(id,asset_id,employee_id,assigned_at,returned_at,condition_out,condition_in)
- AssetMovement(id,asset_id,from_location_id,to_location_id,approved_by,moved_at)
- MaintenanceRecord(id,asset_id,kind,status,opened_at,closed_at,cost)

### Relasi, constraint, indeks
- AssetCategory memiliki PK UUID; FK internal sesuai field *_id.
- Asset memiliki PK UUID; FK internal sesuai field *_id.
- AssetAssignment memiliki PK UUID; FK internal sesuai field *_id.
- AssetMovement memiliki PK UUID; FK internal sesuai field *_id.
- MaintenanceRecord memiliki PK UUID; FK internal sesuai field *_id.
- Timestamp simpan UTC timestamptz; tanggal bisnis DATE + timezone IANA bila jadwal/lokasi berpengaruh. Kolom uang NUMERIC + currency ISO, tidak pakai float.
- Terapkan NOT NULL, CHECK (nilai/status/rentang), UNIQUE (kode/natural key dan constraint aktif), FK, index untuk FK + filter/pagination yang dipakai. Hindari cascade delete histori.
- Tambah created_at, updated_at, created_by, updated_by, version pada aggregate mutable. Tabel append-only/history tidak diubah untuk mengoreksi; buat kompensasi/reversal.

## API REST kontrak modul
Prefix /api/v1. JSON, Bearer access token, Problem Details dengan request_id. Koleksi memakai cursor dan batas page size; retryable command memakai Idempotency-Key; update memakai ETag/If-Match.

| Operasi | Endpoint | Ekspektasi |
|---|---|---|
| Create | POST /api/v1/assets | Authz server-side, validasi domain, audit bila mutasi |
| Read/search | GET /api/v1/assets/{id} | Authz server-side, validasi domain, audit bila mutasi |
| Update/command | POST /api/v1/assets/{id}/assignments | Authz server-side, validasi domain, audit bila mutasi |
| Workflow command | POST /api/v1/assets/{id}/movements | Authz server-side, validasi domain, audit bila mutasi |
| Report/read | POST /api/v1/assets/{id}/retire | Authz server-side, validasi domain, audit bila mutasi |

Error umum: 400 schema invalid, 401 token invalid, 403 disallowed, 404 missing/out-of-scope sesuai anti-enumeration, 409 duplicate/state conflict, 412 version stale, 422 aturan bisnis, 429 throttling. Response list {data,next_cursor}; error {type,title,status,code,detail,request_id,field_errors}.

## Alur utama
1. Parse/validate payload; identitas aktor hanya dari token valid, jangan percaya actor/tenant/scope dalam body.
2. Periksa permission resource:action, ABAC scope, relasi target, masa berlaku grant dan state transition.
3. Jalankan command dalam transaksi singkat; cek version dan constraint DB.
4. Simpan audit dan outbox secara atomik bila perubahan event-worthy.
5. Balikkan resource dan version; proses async tidak menahan request.

## TDD — test case wajib
- **ASSET-01** asset_tag unique. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **ASSET-02** retired tak dapat assign/move. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **ASSET-03** satu assignment aktif. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **ASSET-04** movement scoped+approved. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **ASSET-05** return mencatat kondisi. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **ASSET-06** delete dilarang setelah lifecycle. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.

## Security dan NFR modul
- Default deny; uji positif dan negatif tiap permission/scope. Setiap service tetap melakukan otorisasi meski gateway sudah autentikasi.
- Minimalkan/masking PII; jangan masukkan PII, nilai payroll, token atau isi dokumen ke log, event, metric, trace, URL, error, demo fixtures publik.
- Query parameterized; batas request, response, pagination, timeout; TLS hosted; secret hanya env; log redacted dengan request/correlation ID.
- SLO hanya baseline demo yang diukur, bukan janji produksi. Tulis retention, recovery, replay, migration, dan fallback untuk dependensi bila applicable.

## Definition of Done
Acceptance di atas + semua test ID lulus otomatis; migration dari DB kosong dan upgrade path berhasil; OpenAPI contoh response cocok; permission negative test ada; fixture sintetis; UI state unauthorized/error/loading/empty bila ada UI; README WSL dan dokumentasi diperbarui. Jangan klaim test telah berjalan sebelum benar-benar dijalankan.
