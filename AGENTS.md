# Aturan Codex — EnterpriseOS Backend

Gunakan Bahasa Indonesia untuk diskusi dan dokumentasi. Jalankan semua project command di WSL/Linux dari root checkout; gunakan Node.js LTS, Express 5, TypeScript, dan package manager yang ditentukan lockfile.

## Sebelum coding
1. Baca README, docs/README, keputusan final, SRS/TDD/ERD/kontrak, dan spesifikasi modul yang disentuh.
2. Satu acceptance criterion per task. Jangan mulai modul fase lanjut sebelum dependency fase sebelumnya diterima.
3. Kontrak dan scenario test menjadi dasar implementasi. Ubah API, ERD, spesifikasi backend dan konsumsi frontend secara konsisten.
4. Jalankan hanya command yang benar-benar tersedia; laporkan hasil aktual. Jangan mengklaim test atau migrasi berhasil tanpa menjalankannya.
5. Gunakan data sintetis. Jangan commit secret, token, dump DB, data pegawai nyata atau foto wajah/pegawai nyata.

## Batas arsitektur
Modular monolith Express/TypeScript, bukan 24 microservices. Domain module memiliki ownership dan akses repository sendiri. Tidak ada query/FK lintas domain; pakai application API atau projection. PostgreSQL source of truth. Fase awal tanpa RabbitMQ, Redis, worker, microservices, atau service discovery. Tambahkan hanya dengan ADR dan kebutuhan teruji.

Deployment Vercel adalah HTTP stateless, bukan worker persisten. Jangan simpan file di filesystem runtime. Upload foto presensi langsung ke private Supabase Storage menggunakan signed upload URL singkat, lalu backend memverifikasi metadata/ownership. Bila provider kuota penuh, gagal aman/local demo; jangan menambahkan billing.

## Security wajib
Validasi JWT OIDC issuer/audience/signature/expiry; default deny; otorisasi pada server/domain; validasi input; query parameterized; batas body/rate/timeouts; CORS allowlist; redaksi log; jangan percaya actor, IP forwarding header, lokasi, role, atau scope dari payload browser. Hanya percaya IP proxy setelah konfigurasi trusted proxy terbatas. Data foto dan koordinat adalah data sensitif: least privilege, private storage, retensi/akses tercatat, URL tidak permanen.
