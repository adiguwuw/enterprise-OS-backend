# Keputusan Final

## Produk dan integritas referensi
EnterpriseOS adalah portofolio internal enterprise untuk organisasi fiktif. Telkom Indonesia hanya referensi terbuka untuk memahami ragam fungsi korporat publik; aplikasi tidak berafiliasi, tidak mengaku memakai bagan internal, data pegawai, kebijakan, workflow atau sistem Telkom. Semua nama dan dataset demo fiktif.

## Stack final — gratis saja
- Frontend: React + TypeScript + Vite + shadcn/ui; deploy Vercel Hobby dengan subdomain gratis.
- Backend: Go REST/JSON, OpenAPI, API gateway/HTTP routing; Vercel Go Functions menangani request pendek saja.
- PostgreSQL: Neon Free hosted dan PostgreSQL Docker Compose di WSL.
- OIDC: Auth0 Free untuk demo hosted, Keycloak lokal. Authorization Code + PKCE; aplikasi tidak menyimpan password.
- RabbitMQ baru Phase 5: Docker Compose lokal; CloudAMQP free boleh sebagai demo opsional, bukan kebutuhan. Worker berjalan lokal, bukan Vercel.
- Redis tidak digunakan; PostgreSQL cukup untuk MVP. File DMS hanya metadata dan fixture lokal. Metrics/tracing OSS lokal optional. Tidak memakai layanan email/SMS, bank, storage cloud atau monitoring berbayar.

Free tier memiliki kuota dan ketentuan yang dapat berubah. Tidak menambahkan kartu/billing, paid plan, custom domain berbayar atau fallback berbayar. Saat limit tercapai demo boleh pause dan dijalankan lokal. Hobby hosting bukan jaminan production SLA dan tunduk syarat personal/noncommercial provider. Cek syarat vendor saat deploy.

## Boundary produk
MVP: Organization, IAM/SSO, RBAC/ABAC, struktur perusahaan simulasi, employee basics, directory, audit dasar. Fase 2–5 adalah blueprint untuk tahap berikutnya. Payroll, accounting, procurement, compliance hanya simulasi, bukan penerapan kewajiban hukum atau pemrosesan uang.

## Keputusan arsitektur
Gunakan Go modular monolith dengan bounded context. Jangan pecah 24 modul jadi microservices untuk seorang developer. Single PostgreSQL project awal; tiap konteks punya ownership dan repository boundary. Tidak ada FK/query lintas konteks. Outbox/RabbitMQ setelah event consumer nyata dibutuhkan. Redis ditolak sampai benchmark dan ADR membuktikan manfaat.

## Konvensi final
UUID; UTC timestamps; timezone bisnis eksplisit; amount NUMERIC + currency; API /api/v1; OpenAPI 3.1; cursor pagination; problem response dengan request_id; JWT access token OIDC; default deny; idempotency untuk retryable command; ETag/If-Match untuk concurrency; audit/outbox transactional.

## Keputusan tertunda secara kebijakan (bukan hambatan teknis)
Kuota cuti, formula payroll/tax, retention hukum, klasifikasi bisnis dan SLA produksi harus tetap demo-configurable dan **tidak diisi sebagai fakta** tanpa pemilik kebijakan. Tidak diperlukan untuk Phase 1. Tidak ada klaim kepatuhan.
