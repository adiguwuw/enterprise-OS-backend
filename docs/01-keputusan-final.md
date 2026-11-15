# Keputusan Final

## Produk dan integritas referensi
EnterpriseOS adalah aplikasi enterprise fiktif untuk portofolio. Telkom Indonesia hanya rujukan dari struktur/fungsi publik untuk inspirasi kategori organisasi; tidak berafiliasi, tidak meniru klaim internal, tidak memakai data/kebijakan rahasia. Semua perusahaan, unit, role, pegawai dan skenario demo adalah simulasi.

## Stack final — gratis saja
- Backend: Node.js LTS, Express 5, TypeScript, REST/JSON, OpenAPI 3.1; modular monolith.
- Frontend: React + TypeScript + Vite + shadcn/ui.
- Data: PostgreSQL Neon Free hosted dan PostgreSQL Docker Compose di WSL.
- Identity: Auth0 Free hosted atau Keycloak lokal; OIDC Authorization Code + PKCE. Evaluasi syarat free tier saat deploy, tanpa memasukkan billing.
- Hosting: Vercel Hobby untuk demo frontend/API stateless. Jangan menaruh worker persisten atau file pada filesystem function.
- Foto presensi: Supabase Storage Free private bucket, direct-to-storage upload dengan signed upload URL berumur pendek; PostgreSQL menyimpan metadata. Saat kuota free provider tidak mencukupi, matikan hosted upload dan jalankan demo lokal/mock.
- RabbitMQ: baru Phase 5 dan hanya bila ada consumer konkret; lokal Compose. Redis ditolak dari baseline tanpa benchmark+ADR.

Kuota dan syarat provider dapat berubah. Jangan membeli, menambahkan kartu, mengaktifkan billing, atau upgrade. Free-tier bukan jaminan SLA produksi. Supabase Free memiliki kuota penyimpanan terbatas; gunakan hanya data sintetis dan tangani kuota penuh dengan aman. Vercel Functions membatasi request/response body sekitar 4.5 MB dan filesystem-nya sementara/tidak persisten, sehingga bytes foto tidak dikirim melewati function.

## Batas produk
Roadmap lima fase tetap sebagai katalog fitur portofolio, tetapi implementasi bertahap dan setiap fase harus punya acceptance. Phase 1: Organization, IAM/SSO, RBAC/ABAC, struktur, directory. Phase 2 HR meliputi employee, attendance, leave, payroll sandbox, performance. Fase lanjut ERP, operasi, integrasi. Tidak ada klaim bahwa payroll adalah kalkulator legal atau sistem menangani uang sungguhan.

## Keputusan presensi
- WFO: validasi server terhadap public egress IP/CIDR kantor yang dikonfigurasi pada lokasi kerja; browser tidak dapat membaca SSID/BSSID secara andal. IP hanya sinyal lokasi jaringan, bukan bukti identitas tunggal. Jangan percaya X-Forwarded-For kecuali proxy tepercaya dikonfigurasi.
- WFH/WFA: pegawai mengajukan jadwal/lokasi dan alasan; atasan berwenang memutuskan approve/reject dengan alasan wajib. Punch remote hanya untuk request approved dan hari yang sesuai.
- Setiap check-in/out mengharuskan foto bukti yang diambil saat aksi dan koordinat GPS browser beserta accuracy/timestamp izin. Untuk WFO, IP dipetakan ke WorkLocation terkonfigurasi (termasuk label kecamatan); GPS tetap dicatat bila diizinkan sebagai sinyal tambahan. Untuk remote, koordinat GPS harus berada dalam kebijakan radius yang dapat dikonfigurasi; tidak mengandalkan geolokasi IP untuk menentukan kecamatan karena akurasinya tidak memadai.
- Foto diunggah langsung ke Supabase private bucket memakai signed URL; simpan object key, checksum/ukuran/mime, waktu, aktor dan status verifikasi. URL signed tidak disimpan sebagai permanen.
- Tidak ada pelacakan latar belakang, pengenalan wajah, biometrik, spoof-proof claim, atau keputusan disipliner otomatis. Berikan alternatif koreksi manual dengan audit saat izin kamera/GPS atau jaringan bermasalah. Retensi dan akses harus dapat dikonfigurasi; jangan klaim kepatuhan UU PDP.

## Arsitektur dan data
Gunakan modular monolith Express dengan bounded context. Satu PostgreSQL awal dengan schema/ownership terpisah; repository domain melarang query lintas konteks. Domain mutasi sensitif mencatat audit; outbox/event baru ketika consumer nyata diperlukan. UUID, UTC timestamptz, timezone bisnis eksplisit, amount NUMERIC.

## Administrasi policy presensi
Radius GPS, toleransi accuracy, shift/timezone, kalender/libur, retensi bukti, dan aturan exceptional punch ditetapkan Admin berwenang melalui Dashboard Admin. Perubahan disimpan sebagai versi baru dengan tanggal efektif, validasi server, alasan, audit perubahan sebelum/sesudah, dan riwayat immutable. Setiap punch menyimpan referensi versi policy yang berlaku saat itu; perubahan policy tidak mengubah histori. Rollback dilakukan dengan versi baru.

## Nilai awal demo (dapat diubah Admin)
Nilai seed portofolio: timezone Asia/Jakarta; jadwal Senin–Jumat 09.00–17.00 dengan istirahat 12.00–13.00; grace window punch 15 menit; radius GPS 150 m; GPS accuracy maksimum 100 m; usia pembacaan lokasi maksimum 60 detik; retensi foto 30 hari; retensi metadata attendance 365 hari. Admin dapat mengubahnya melalui dashboard dalam bounds server: radius 25–500 m; accuracy maksimum 10–200 m; usia lokasi 30–300 detik; retensi foto 1–90 hari; metadata 30–3.650 hari. Semua nilai adalah default demo sintetis, bukan kebijakan Telkom/anjuran legal.

CIDR kantor tidak diberi nilai aktif bawaan karena harus berasal dari public egress kantor yang benar-benar dikonfigurasi. Status lokasi/network seed adalah belum terverifikasi/nonaktif; WFO ditolak sampai Admin memverifikasi konfigurasi. IP fixture dokumentasi tidak boleh dipakai sebagai alamat nyata.

## Role admin final untuk demo
Pisahkan Employee, Manager, Attendance Policy Admin, Attendance Policy Approver, HR Attendance Admin, Org Admin, IAM Admin, Auditor dan Platform Admin. Hak selalu least-privilege dan dibatasi organization/location/team scope. Policy Admin dapat mengubah konfigurasi, tetapi tidak menyetujui perubahan sendiri jika maker-checker aktif. Policy Admin tidak otomatis dapat melihat foto; permission evidence.read diberikan terpisah. Platform Admin tidak otomatis memperoleh akses data HR. IAM Admin dapat memberi grant dalam scope yang ditugaskan tetapi tidak dapat menaikkan dirinya sendiri atau memberi role Platform Admin.

Penetapan akun demo ke role dilakukan lewat seeder yang jelas fiktif; pada deployment, Admin harus ditugaskan secara eksplisit. Default awal maker-checker aktif.
