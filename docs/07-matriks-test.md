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
| INT-16 | kuota/free DB limit | degrade tanpa paid upgrade; local run tersedia |\n| ATT-01 | WFO IP kantor cocok/tidak cocok CIDR | hanya jaringan aktif yang match; lokasi berasal dari konfigurasi |\n| ATT-02 | X-Forwarded-For palsu / proxy tak tepercaya | policy gagal aman; tidak dapat spoof jaringan kantor |\n| ATT-03 | WFH/WFA request pending/rejected/expired | punch ditolak; request approved dan tanggal berlaku diperlukan |\n| ATT-04 | alasan pengajuan/keputusan kosong atau self-approval | 422/403; tidak ada perubahan parsial |\n| ATT-05 | GPS missing/stale/accuracy di luar policy | ditolak atau exception audited sesuai konfigurasi |\n| ATT-06 | upload bukti expired, salah owner, MIME/size invalid | confirm/punch ditolak; file tidak dapat dibaca publik |\n| ATT-07 | read URL evidence tanpa scope / setelah expiry | denied atau URL sementara kedaluwarsa; akses diaudit |\n| ATT-08 | storage quota/provider unavailable | punch tidak sukses tanpa evidence; tidak beralih ke layanan berbayar |\n| ATT-09 | duplicate/concurrent check-in atau urutan punch salah | satu state valid, tanpa duplikasi |\n| ATT-10 | log/event/trace memuat foto, koordinat, raw IP atau alasan | redaksi/larangan terverifikasi; tidak ada kebocoran |\n| ATT-11 | retention dan penghapusan evidence | kebijakan berjalan dan penghapusan tercatat tanpa merusak histori wajib |\n| ATT-12 | koreksi manual attendance | maker-checker, alasan dan audit; tidak mengubah punch asli |

Test layers: unit domain/policy, component handler, PostgreSQL integration+constraints, OpenAPI/event contract, E2E journey, security (authz matrix, IDOR, injection, CSRF/XSS/CORS, rate/body limit, secret scan). Merge gate format/lint/test/migration/OpenAPI/docs. Catat hasil aktual.
