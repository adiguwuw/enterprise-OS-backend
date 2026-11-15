# Matriks Test Integrasi

| ID | Skenario | Hasil wajib |
|---|---|---|
| INT-01 | OIDC login lalu directory | principal valid; field terlihat saja |
| INT-02 | issuer/audience invalid atau JWKS gagal | fail closed, tanpa bypass |
| INT-03 | manager lihat unit sendiri dan luar scope | scope benar; tak ada count leak |
| INT-04 | HR buat employee+assignment valid | transaksi, audit dan projection konsisten |
| INT-05 | update paralel stale If-Match | 412, tak ada state parsial |
| INT-06 | command retry same idempotency key | satu efek domain |
| INT-07 | broker kirim event dua kali | inbox mencegah duplikasi |
| INT-08 | broker mati atau consumer gagal | outbox pending/retry/DLQ; tak hilang |
| INT-09 | role dicabut lalu request baru | deny; stale allow tidak lewat batas cache |
| INT-10 | payload actor/scope palsu | diabaikan/ditolak; identity dari token |
| INT-11 | SQL/HTML pada search | parameterized, escaped, cursor bounded |
| INT-12 | log/trace payroll/profile | tidak berisi PII/token/nominal sensitif |
| INT-13 | migration fresh dan upgrade | constraints/history benar |
| INT-14 | principal disable/employee separated | akses & directory sesuai lifecycle |
| INT-15 | keyboard/mobile/loading/empty/error | operable dan feedback benar |
| INT-16 | kuota/free DB limit | degrade tanpa paid upgrade; local run tersedia |
| ATT-01 | WFO IP kantor cocok/tidak cocok CIDR | hanya jaringan aktif yang match; lokasi berasal dari konfigurasi |
| ATT-02 | X-Forwarded-For palsu / proxy tak tepercaya | policy gagal aman; tidak dapat spoof jaringan kantor |
| ATT-03 | WFH/WFA request pending/rejected/expired | punch ditolak; request approved dan tanggal berlaku diperlukan |
| ATT-04 | alasan pengajuan/keputusan kosong atau self-approval | 422/403; tidak ada perubahan parsial |
| ATT-05 | GPS missing/stale/accuracy di luar policy | ditolak atau exception audited sesuai konfigurasi |
| ATT-06 | upload bukti expired, salah owner, MIME/size invalid | confirm/punch ditolak; file tidak dapat dibaca publik |
| ATT-07 | read URL evidence tanpa scope / setelah expiry | denied atau URL sementara kedaluwarsa; akses diaudit |
| ATT-08 | storage quota/provider unavailable | punch tidak sukses tanpa evidence; tidak beralih ke layanan berbayar |
| ATT-09 | duplicate/concurrent check-in atau urutan punch salah | satu state valid, tanpa duplikasi |
| ATT-10 | log/event/trace memuat foto, koordinat, raw IP atau alasan | redaksi/larangan terverifikasi; tidak ada kebocoran |
| ATT-11 | retention dan penghapusan evidence | kebijakan berjalan dan penghapusan tercatat tanpa merusak histori wajib |
| ATT-12 | koreksi manual attendance | maker-checker, alasan dan audit; tidak mengubah punch asli |
| POL-01 | admin mengubah radius/akurasi/shift/timezone/retensi | versi baru tervalidasi, effective-dated, beralasan dan diaudit |
| POL-02 | non-admin atau admin di luar scope mengubah policy | 403 dan tidak ada perubahan |
| POL-03 | policy overlap, nilai invalid, alasan kosong atau If-Match stale | ditolak atomik |
| POL-04 | policy berubah setelah punch tercatat | punch tetap menunjuk versi yang berlaku saat punch |
| POL-05 | maker-checker aktif | creator tidak dapat menyetujui versinya sendiri |
| POL-06 | tidak ada policy aktif pada waktu punch | fail closed atau exception audited; tidak ada default diam-diam |

| POL-07 | seed policy baru | default demo sesuai nilai baseline Asia/Jakarta/09-17/radius 150m/accuracy 100m/retensi foto 30 hari |
| POL-08 | admin memilih nilai batas minimum/maksimum dan di luar bounds | boundary accepted; out-of-bounds rejected server-side |
| RBAC-01 | Policy Admin mengubah policy dalam scope | diizinkan; outside scope 403 |
| RBAC-02 | Policy Admin mencoba melihat bukti foto tanpa grant | 403 |
| RBAC-03 | creator menyetujui versi sendiri | 403; approver lain dapat menyetujui |
| RBAC-04 | Platform Admin atau IAM Admin meminta data HR tanpa grant | 403; tidak ada akses implisit |
| RBAC-05 | IAM Admin self-grant / assign Platform Admin | ditolak dan diaudit |
| RBAC-06 | Manager melihat request di luar team scope | deny/404 sesuai anti-enumeration |

Test layers: unit domain/policy, component handler, PostgreSQL integration+constraints, OpenAPI/event contract, E2E journey, security (authz matrix, IDOR, injection, CSRF/XSS/CORS, rate/body limit, secret scan). Merge gate format/lint/test/migration/OpenAPI/docs. Catat hasil aktual.