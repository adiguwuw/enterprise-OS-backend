# Aturan Codex — EnterpriseOS Backend

Gunakan bahasa Indonesia untuk diskusi dan dokumentasi. Pengembangan dilakukan di **WSL/Linux shell**, dari root checkout; jangan jalankan tool Go atau package manager dari PowerShell/Windows. Gunakan path Linux dan LF, bukan path campuran.

## Sebelum mengubah file
1. Baca README.md, docs/README.md, docs/01-keputusan-final.md, kontrak global, lalu spesifikasi modul yang disentuh.
2. Kerjakan satu acceptance criterion/issue per perubahan. Cek pola yang ada lebih dahulu. Hindari abstraksi/provider/service baru tanpa alasan dan ADR.
3. Kontrak dan test lebih dulu; catat command dan hasil aktual. Jangan mengklaim test tanpa menjalankannya.
4. Ubah ERD, OpenAPI, skenario dan dokumen FE bersama jika kontrak berubah.
5. Pakai data sintetis; jangan commit token, secret, database dump atau PII riil.

## Batas arsitektur
Go modular monolith per bounded context dalam satu API deployable; jangan membangun 24 microservices. Directory adalah read projection Employee; company structure memakai Organization; IAM menyimpan policy, domain meng-enforce. Vercel Go Functions untuk HTTP request singkat saja. Tidak ada background worker persisten di Vercel. RabbitMQ hanya fase 5, consumer lokal; Redis tidak digunakan kecuali benchmark+ADR. PostgreSQL adalah source of truth.

## Environment target
WSL, Docker Compose, PostgreSQL lokal, Go modules, REST/OpenAPI. Gunakan go test ./..., go vet ./... dan linter yang dikonfigurasi setelah scaffold ada. Jangan mengarang output perintah. File env hanya .env.example tanpa credential.

## Security
Verifikasi JWT issuer/audience/signature/expiry; default deny; authz service-side; query parameterized; Idempotency-Key/If-Match sesuai kontrak; audit/outbox transactional; error/log redacted. Jangan simpan password atau token IdP.
