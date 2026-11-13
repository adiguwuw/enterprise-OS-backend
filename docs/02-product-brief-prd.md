# Product Brief / PRD

## Visi
Portofolio full-stack yang menunjukkan desain produk enterprise aman dari identity/org foundation hingga workflow HR, ERP, operasi dan integrasi, namun dapat dikerjakan bertahap oleh satu fresh graduate dengan layanan gratis.

## Problem
Data struktur, izin dan direktori pegawai sering terpecah; implementasi enterprise memerlukan batas akses, histori, audit dan kontrak API. EnterpriseOS mendemonstrasikan pondasi dan alur tersebut dengan data simulasi.

## Persona
Employee (profil dan pencarian kolega); Manager (directory scope dan approval); HR Operator (lifecycle pegawai); Org Admin (unit/posisi/lokasi); IAM Admin (grant dan policy); Auditor (read-only audit/review). Semua role fiktif.

## MVP
OIDC login; principal mapping; role+permission+scope; org tree efektif-tanggal; employee create/update/separation dasar; directory filter/detail dengan masking; audit event mutasi; UI admin dan employee. Seeder lokal sintetis. Sinkron DB cukup, RabbitMQ tidak masuk MVP.

## Outcome demo
Perjalanan login → lihat struktur → cari directory → buka profil sesuai scope; admin buat unit dan pegawai; perubahan tercatat; user tanpa grant ditolak. Tunjukkan test authz negatif dan UI loading/empty/error. Tidak mengklaim benchmark production.

## Fase
Phase 1 Foundation; Phase 2 HR (termasuk payroll sandbox); Phase 3 Finance/Accounting/Procurement/Assets; Phase 4 CRM/Project/DMS metadata/Collaboration; Phase 5 events/RabbitMQ/audit compliance/monitoring/tracing.

## Non-goals
ERP/CRM produksi, statutory payroll/tax, bank/payment, dokumen aman di cloud, keputusan HR otomatis, AI scoring, mobile native, integrasi berbayar, data pegawai nyata, SSO enterprise berbayar, SLA/certification claims.

## Acceptance MVP
Semua API terautentikasi; negative authz tests; directory hanya atribut terlihat dan scope-allowed tanpa count leak; org tree acyclic dan efektif tanggal; lifecycle HR konsisten; perubahan sensitif diaudit; duplicate request tidak menggandakan side effect; clone/run/test WSL didokumentasikan.
