# EnterpriseOS Backend

Backend blueprint untuk portofolio EnterpriseOS. Organisasi, pegawai, kebijakan, transaksi, dan akun demo seluruhnya fiktif. Telkom Indonesia hanya referensi struktur korporat yang dipublikasikan; proyek ini tidak berafiliasi dan tidak mengklaim mengetahui proses/internal Telkom.

## Dokumen dan status
Baca [indeks blueprint](docs/README.md), keputusan final, PRD, SRS, TDD, ERD, kontrak, skenario uji, dan spesifikasi per modul. Repository berisi blueprint; implementasi dibangun bertahap setelah review dokumen.

## Stack yang disepakati
- Node.js LTS + Express 5 + TypeScript; modular monolith HTTP API, REST/JSON dan OpenAPI 3.1.
- PostgreSQL: Neon Free untuk demo hosted dan PostgreSQL Docker Compose untuk WSL.
- OIDC: Auth0 Free untuk demo hosted atau Keycloak lokal; Authorization Code + PKCE. Password tetap dikelola IdP.
- Web/API: Vercel Hobby untuk demo portofolio. Express API harus stateless dan request singkat.
- Bukti foto presensi: Supabase Storage Free, private bucket, unggah langsung dari browser memakai signed upload URL singkat yang backend keluarkan. PostgreSQL hanya menyimpan metadata.
- RabbitMQ hanya bila Phase 5 mempunyai consumer konkret; lokal Compose. Redis tidak termasuk baseline.

## Biaya dan batas provider
Gunakan layanan gratis tanpa kartu/billing dan tanpa upgrade. Free quota/eligibility dapat berubah; demo dapat pause bila kuota habis. Supabase Free storage saat keputusan ini ditulis memiliki kuota terbatas (1 GB); ukur dan tampilkan batas, jangan mengklaim SLA produksi. Jangan unggah data pribadi nyata. Vercel Functions memiliki batas body request/response 4.5 MB dan filesystem runtime tidak persisten; foto tidak boleh melewati request Express/Vercel. Jika storage free tidak tersedia, gunakan mock/local demo, jangan diam-diam beralih ke layanan berbayar.

## WSL
Jalankan Node, package manager sesuai lockfile, Docker Compose, migration, dan test dari WSL/Linux checkout. Jangan mencampur dependency Windows dan WSL. Simpan hanya .env.example pada Git; secret nyata tetap lokal.
