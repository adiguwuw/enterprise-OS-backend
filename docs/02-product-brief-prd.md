# Product Brief / PRD

## Visi
EnterpriseOS adalah portofolio aplikasi internal enterprise yang menunjukkan penguasaan identity, organisasi, workflow HR, data ownership, security dan integrasi. Organisasi serta data demo sepenuhnya fiktif; Telkom Indonesia hanya referensi fungsi korporat publik.

## Pengguna
Employee melihat profil dan mengajukan layanan; Manager menyetujui request dalam scope tim; HR Operator mengelola lifecycle; Org Admin mengelola struktur/lokasi; IAM Admin mengelola role/grant; Auditor membaca jejak secara read-only. Role ini persona simulasi.

## Nilai produk
Satu pengalaman terpadu bagi employee dan admin, dengan izin server-side, struktur organisasi efektif tanggal, direktori terkontrol, dan workflow yang memiliki status, alasan keputusan dan audit. Demo harus bisa ditunjukkan end-to-end, termasuk skenario ditolak dan recovery.

## Ruang lingkup fase
1. Foundation: OIDC, principal, RBAC/ABAC, Organization/OrgUnit, Employee Directory.
2. HR: Employee Management, attendance, leave, payroll sandbox, performance. Presensi WFO memakai public egress IP kantor yang dikonfigurasi; WFH/WFA melalui rekomendasi/keputusan atasan dengan alasan; setiap punch memiliki foto + bukti lokasi sesuai spesifikasi.
3. ERP: Finance, Accounting, Procurement, Asset Management sebagai simulasi workflow.
4. Business Operations: CRM, project, metadata dokumen, collaboration.
5. Integrasi: RabbitMQ hanya untuk event consumer konkret; audit, monitoring, tracing.

## Outcome demo
Login → lihat org chart → cari directory → ajukan/minta keputusan kerja remote → clock in/out dengan foto dan lokasi → manager memeriksa bukti dalam scope → perubahan tercatat di audit. Demo menampilkan error izin kamera/GPS, IP tak cocok, file invalid, request belum disetujui, upload quota habis, dan koreksi manual.

## Batas
Tidak memakai data pegawai nyata, bukan pengganti HR legal/payroll/statutory, tidak melakukan pengenalan wajah/monitoring kontinyu, tidak membuat keputusan HR otomatis, tidak mengklaim menyerupai proses internal Telkom. No paid service/billing. Produk diimplementasikan per vertical slice; fitur fase akhir bukan MVP.


## Role presensi demo
Employee: punch dan request sendiri; Manager: review request/team dalam scope; Attendance Policy Admin: mengelola policy/lokasi/CIDR scope tertentu; Attendance Policy Approver: menyetujui versi policy oleh orang lain; HR Attendance Admin: laporan/koreksi scope HR; Auditor: membaca audit dan policy history; Org Admin: kelola struktur/lokasi; IAM Admin: grant role non-global dalam scope; Platform Admin: platform setup tanpa akses HR otomatis. Assignment akun demo sintetis.
