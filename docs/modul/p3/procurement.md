# Procurement

Keputusan blueprint final · Phase 3 · procurement

## Tujuan dan batas
Procure-to-receive simulasi; supplier sintetis, tanpa ERP eksternal.

## PRD/SRS dan acceptance
- Aktor, tindakan, scope organisasi, status lifecycle, dan data sensitivity harus tervalidasi server-side.
- Modul memiliki data canonical yang disebut di ERD; modul lain merujuk ID opaque dan memakai API/event, tanpa query/FK lintas bounded context.
- Mutasi harus atomik, aman diulang bila memakai Idempotency-Key, memakai version/If-Match pada aggregate yang dapat diedit, dan mencatat audit untuk perubahan sensitif.
- Data demo seluruhnya sintetis. Tidak ada asumsi kebijakan/pajak/hukum atau struktur internal perusahaan nyata.

## ERD logis (PostgreSQL)
- Supplier(id,code,name,status,tax_ref_masked)
- Requisition(id,requester_id,cost_center_id,status,needed_by,version)
- RequisitionLine(id,requisition_id,item,quantity,unit,estimated_unit_cost)
- PurchaseOrder(id,po_no,supplier_id,requisition_id,status,currency,total,approved_at)
- GoodsReceipt(id,po_id,received_by,received_at,status)
- SupplierQuote(id,requisition_id,supplier_id,amount,currency,valid_until)

### Relasi, constraint, indeks
- Supplier memiliki PK UUID; FK internal sesuai field *_id.
- Requisition memiliki PK UUID; FK internal sesuai field *_id.
- RequisitionLine memiliki PK UUID; FK internal sesuai field *_id.
- PurchaseOrder memiliki PK UUID; FK internal sesuai field *_id.
- GoodsReceipt memiliki PK UUID; FK internal sesuai field *_id.
- SupplierQuote memiliki PK UUID; FK internal sesuai field *_id.
- Timestamp simpan UTC timestamptz; tanggal bisnis DATE + timezone IANA bila jadwal/lokasi berpengaruh. Kolom uang NUMERIC + currency ISO, tidak pakai float.
- Terapkan NOT NULL, CHECK (nilai/status/rentang), UNIQUE (kode/natural key dan constraint aktif), FK, index untuk FK + filter/pagination yang dipakai. Hindari cascade delete histori.
- Tambah created_at, updated_at, created_by, updated_by, version pada aggregate mutable. Tabel append-only/history tidak diubah untuk mengoreksi; buat kompensasi/reversal.

## API REST kontrak modul
Prefix /api/v1. JSON, Bearer access token, Problem Details dengan request_id. Koleksi memakai cursor dan batas page size; retryable command memakai Idempotency-Key; update memakai ETag/If-Match.

| Operasi | Endpoint | Ekspektasi |
|---|---|---|
| Create | POST /api/v1/procurement/requisitions | Authz server-side, validasi domain, audit bila mutasi |
| Read/search | POST /api/v1/procurement/requisitions/{id}/submit | Authz server-side, validasi domain, audit bila mutasi |
| Update/command | POST /api/v1/procurement/requisitions/{id}/quotes | Authz server-side, validasi domain, audit bila mutasi |
| Workflow command | POST /api/v1/procurement/purchase-orders | Authz server-side, validasi domain, audit bila mutasi |
| Report/read | POST /api/v1/procurement/purchase-orders/{id}/receipts | Authz server-side, validasi domain, audit bila mutasi |

Error umum: 400 schema invalid, 401 token invalid, 403 disallowed, 404 missing/out-of-scope sesuai anti-enumeration, 409 duplicate/state conflict, 412 version stale, 422 aturan bisnis, 429 throttling. Response list {data,next_cursor}; error {type,title,status,code,detail,request_id,field_errors}.

## Alur utama
1. Parse/validate payload; identitas aktor hanya dari token valid, jangan percaya actor/tenant/scope dalam body.
2. Periksa permission resource:action, ABAC scope, relasi target, masa berlaku grant dan state transition.
3. Jalankan command dalam transaksi singkat; cek version dan constraint DB.
4. Simpan audit dan outbox secara atomik bila perubahan event-worthy.
5. Balikkan resource dan version; proses async tidak menahan request.

## TDD — test case wajib
- **PROC-01** quantity positive/quote valid. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **PROC-02** requester tak approve sendiri. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **PROC-03** PO dari requisition approved dan total valid. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **PROC-04** supplier/PO unique. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **PROC-05** receipt tidak melebihi qty PO. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.
- **PROC-06** partial receipt mencegah cancel sembarang. Implementasikan sebagai unit/domain test, API integration test, DB constraint test; tambahkan authz/audit/event test jika terkait.

## Security dan NFR modul
- Default deny; uji positif dan negatif tiap permission/scope. Setiap service tetap melakukan otorisasi meski gateway sudah autentikasi.
- Minimalkan/masking PII; jangan masukkan PII, nilai payroll, token atau isi dokumen ke log, event, metric, trace, URL, error, demo fixtures publik.
- Query parameterized; batas request, response, pagination, timeout; TLS hosted; secret hanya env; log redacted dengan request/correlation ID.
- SLO hanya baseline demo yang diukur, bukan janji produksi. Tulis retention, recovery, replay, migration, dan fallback untuk dependensi bila applicable.

## Definition of Done
Acceptance di atas + semua test ID lulus otomatis; migration dari DB kosong dan upgrade path berhasil; OpenAPI contoh response cocok; permission negative test ada; fixture sintetis; UI state unauthorized/error/loading/empty bila ada UI; README WSL dan dokumentasi diperbarui. Jangan klaim test telah berjalan sebelum benar-benar dijalankan.
