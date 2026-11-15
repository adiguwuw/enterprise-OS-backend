# EnterpriseOS Backend — Dokumentasi Keputusan Final

Blueprint portofolio, bukan implementasi selesai. Seluruh organisasi, pegawai, transaksi dan event adalah fiksi. Referensi Telkom hanya inspirasi fungsi organisasi yang dipublikasikan; tidak menyalin bagan internal, kebijakan, data atau sistem rahasia. Frontend: https://github.com/adiguwuw/enterprise-OS-frontend

## Stack yang disepakati
Node.js LTS + Express 5 + TypeScript sebagai modular monolith; React + TypeScript + Vite + shadcn/ui; PostgreSQL Neon Free dan Docker Compose lokal WSL; OIDC free hosted/Keycloak lokal; Vercel Hobby untuk demo stateless; Supabase Storage Free private untuk foto presensi direct upload. Tanpa kartu, billing, upgrade atau fallback berbayar. Kuota/ketentuan gratis dapat berubah dan tidak menjamin production SLA.

Presensi Phase 2: WFO divalidasi dari IP egress publik kantor yang dipetakan ke lokasi terkonfigurasi; WFH/WFA memerlukan persetujuan atasan beserta alasan; setiap punch memakai foto privat dan GPS. Tidak ada SSID/BSSID browser, IP geolocation kecamatan, pelacakan latar belakang, pengenalan wajah, atau foto melalui Vercel. Detail dan kebijakan terbuka ada di keputusan final, SRS, TDD dan spesifikasi Attendance.

## Review dokumen berurutan
1. docs/01-keputusan-final.md — batas produk, teknologi dan biaya.
2. docs/02-product-brief-prd.md — pengguna, nilai dan outcome.
3. docs/03-srs.md — kebutuhan lintas produk.
4. docs/04-tdd.md — keputusan arsitektur.
5. docs/05-erd-global.md — kepemilikan dan relasi.
6. Spesifikasi setiap modul Phase 1, kemudian Phase 2–5.
7. docs/06-kontrak-api-event.md dan docs/07-matriks-test.md.
8. docs/08-roadmap.md; lanjut review route/UX/test di repo frontend.

## Spesifikasi modul per fase
- Phase 1 — [Organization Service](modul/p1/organization-service.md)
- Phase 1 — [IAM Service](modul/p1/iam-service.md)
- Phase 1 — [Authentication & SSO](modul/p1/authentication-sso.md)
- Phase 1 — [RBAC & ABAC](modul/p1/rbac-abac.md)
- Phase 1 — [Company Structure](modul/p1/company-structure.md)
- Phase 1 — [Employee Directory](modul/p1/employee-directory.md)
- Phase 2 — [Employee Management](modul/p2/employee-management.md)
- Phase 2 — [Attendance](modul/p2/attendance.md)
- Phase 2 — [Leave Management](modul/p2/leave-management.md)
- Phase 2 — [Payroll](modul/p2/payroll.md)
- Phase 2 — [Performance Management](modul/p2/performance-management.md)
- Phase 3 — [Finance](modul/p3/finance.md)
- Phase 3 — [Accounting](modul/p3/accounting.md)
- Phase 3 — [Procurement](modul/p3/procurement.md)
- Phase 3 — [Asset Management](modul/p3/asset-management.md)
- Phase 4 — [CRM](modul/p4/crm.md)
- Phase 4 — [Project Management](modul/p4/project-management.md)
- Phase 4 — [Document Management](modul/p4/document-management.md)
- Phase 4 — [Collaboration](modul/p4/collaboration.md)
- Phase 5 — [RabbitMQ](modul/p5/rabbitmq.md)
- Phase 5 — [Event-driven Architecture](modul/p5/event-driven-architecture.md)
- Phase 5 — [Audit & Compliance](modul/p5/audit-compliance.md)
- Phase 5 — [Monitoring](modul/p5/monitoring.md)
- Phase 5 — [Distributed Tracing](modul/p5/distributed-tracing.md)

## Keputusan sumber kebenaran
API/ERD final berada di setiap spesifikasi modul dan kontrak global. Produk dikembangkan sebagai modular monolith, bukan 24 deployable services. Repository ini saat ini dokumentasi/scaffold saja; kode belum dibuat. Mulai development hanya pada vertical slice Phase 1 setelah urutan review.


## Keputusan tambahan presensi
Policy operasional presensi ditetapkan Admin berwenang lewat Dashboard Admin: radius GPS, batas akurasi, shift/timezone, kalender kerja, retensi, exception dan konfigurasi lokasi/CIDR. Perubahan berversi, effective-dated, memiliki alasan dan audit; setiap punch merekam policy version. Detail API, ERD, test dan UI ada di SRS/TDD/ERD/kontrak/matriks backend dan attendance frontend.

Nilai awal demo (dapat diubah dalam bounds): Asia/Jakarta; Senin–Jumat 09:00–17:00; break 12:00–13:00; grace 15 menit; GPS radius 150 m; accuracy maksimum 100 m; capture age 60 detik; retensi foto 30 hari; metadata 365 hari. Tidak ada CIDR kantor aktif sebelum Admin mengisi dan memverifikasinya. Role/permission baseline dan bounds detail berada pada spec Attendance.
