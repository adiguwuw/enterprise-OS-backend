# TDD / Technical Design Description

## Komponen
React/TypeScript SPA pada repo frontend; Express 5 + TypeScript modular monolith sebagai HTTP API; PostgreSQL; OIDC provider; private Supabase Storage untuk bukti foto. Phase 5 opsional menambahkan outbox publisher/RabbitMQ consumer lokal dan telemetry OSS.

## Batas layanan
Istilah service adalah bounded context/module yang berjalan dalam satu proses API, bukan satu deployable per domain. Organization memiliki struktur/lokasi; Employee memiliki employment; Directory adalah projection; IAM memiliki identity mapping/policy, sedangkan setiap domain melakukan authz. Attendance memiliki workflow presensi dan remote-work request.

## Runtime/deployment gratis
WSL: Node LTS, package manager sesuai lockfile, Docker Compose PostgreSQL dan Keycloak. Hosted: Vercel Hobby untuk frontend dan Express HTTP function stateless; Neon Free PostgreSQL, Auth0 Free OIDC (atau konfigurasi lokal Keycloak), Supabase Storage Free private bucket. Semua kuota dapat berubah; tanpa kartu/billing/upgrade. Jangan menjalankan worker persisten di Vercel. Jangan proxy bytes foto melalui Vercel (limit 4.5 MB dan filesystem tak persisten); gunakan direct signed upload ke storage. Fallback kuota penuh adalah error aman/local demo.

## Struktur kode backend yang disarankan
src/app.ts; src/server.ts; src/config; src/platform/{db,http,auth,storage,logging}; src/modules/<context>/{domain,application,ports,adapters/http,adapters/postgres}; migrations; api/openapi; tests/{unit,integration,contract}; deploy/docker-compose.yml; docs/adr. Tetap sederhana; jangan membuat worker sampai Phase 5.

## Kontrak dan transaksi
Prefix /api/v1; OpenAPI 3.1; access token, cursor pagination, problem errors, Idempotency-Key untuk command retryable, ETag/If-Match untuk aggregate mutable. Tiap module owns tables. Audit mutation tersimpan atomik dengan domain state; outbox hanya bila ada consumer. Jangan lakukan network call saat transaksi DB.

## Desain upload presensi
1. Browser meminta upload intent ke API dengan punch draft, mime/size, nonce.
2. Backend authz, validasi kebijakan lokasi, membuat random object key serta signed upload URL berumur pendek di bucket private; simpan intent PENDING dan batas ukuran/type.
3. Browser mengirim file langsung ke Supabase Storage dengan signed URL.
4. Browser meminta confirm; backend memverifikasi object ada, ukuran/mime/checksum sesuai, pemilik dan expiry; baru punch dapat commit.
5. Signed read URL dibuat hanya setelah authz; tidak pernah menjadi data permanen. Expired/orphan intent dibersihkan dengan mekanisme manual/cron provider free hanya setelah ADR dan tersedia.

## Trust boundary lokasi
IP kantor diambil dari request peer/proxy chain yang disanitasi konfigurasi trusted proxy; jangan menerima office_ip dari body. Cocokkan public egress CIDR → WorkLocation. GPS adalah sinyal browser permission, bukan trusted attestation; simpan accuracy/timestamp dan cek batas/radius yang dikonfigurasi. Remote punch memerlukan request approved. Tidak membaca SSID/BSSID; tidak memperkirakan kecamatan dari IP.

## Test approach
Test domain state machine, IP CIDR/trusted proxy, remote approval + reason, location accuracy/radius, signed upload ownership/expiry, MIME/size/confirmation, idempotency/concurrency, scope, audit access and privacy redaction. Integration memakai PostgreSQL migrasi fresh dan storage adapter fake/local; contract test OpenAPI; E2E sintetis. Tidak pernah menggunakan foto/PII riil.


## Policy management presensi
Policy adalah aggregate berversi milik Attendance. Admin mengubahnya melalui API Dashboard; backend memvalidasi bounds, effective date, overlap, org/location scope, permission, reason dan If-Match. Simpan versi baru serta audit before/after atomik; bila maker-checker aktif, creator tidak dapat menyetujui versinya. Resolver memilih policy aktif pada waktu kerja dan punch menyimpan versi yang digunakan. Versi historis immutable; rollback dibuat sebagai versi baru. Cache tidak boleh melampaui effective boundary dan resolver fail closed bila tidak ada policy aktif.

## Policy seed dan validasi
Seeder demo membuat policy versi awal dengan nilai: Asia/Jakarta; Senin–Jumat 09:00–17:00; istirahat 12:00–13:00; grace 15 menit; radius GPS 150 m; max accuracy 100 m; location age 60 detik; photo retention 30 hari; metadata retention 365 hari. Server bounds: radius 25–500 m; accuracy 10–200 m; location age 30–300 detik; photo retention 1–90 hari; metadata retention 30–3650 hari. Admin boleh memilih angka dalam bounds; backend, bukan UI, memvalidasi. Office CIDR harus dimasukkan dan diverifikasi oleh Admin; tidak ada CIDR aktif default. Maker-checker aktif untuk publish policy, creator berbeda dari approver.
