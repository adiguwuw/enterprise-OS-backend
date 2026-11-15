# SRS dan Kriteria Penerimaan Lintas Modul

## Functional requirements
- FR-01 OIDC Authorization Code + PKCE; API memverifikasi access token issuer/audience/signature/expiry.
- FR-02 principal terikat issuer+subject unik; disable/revoke tidak bergantung pada email.
- FR-03 role mengelompokkan permission resource/action; grant memiliki scope, validity dan default deny.
- FR-04 ABAC server-side mempertimbangkan scope org, relasi resource, state dan waktu.
- FR-05 organisasi bertingkat efektif tanggal, work location dan position catalog; cegah siklus.
- FR-06 employee, employment, assignment dan lifecycle dengan field sensitif dibatasi.
- FR-07 directory searchable, paginated, masked dan scope controlled.
- FR-08 HR workflows memiliki maker/checker jika disyaratkan, alasan keputusan, audit dan concurrency control.
- FR-09 presensi WFO mencocokkan request IP dengan CIDR publik kantor yang dikonfigurasi; tidak memakai SSID browser.
- FR-10 WFH/WFA perlu remote-work request yang disetujui atasan, tanggal berlaku, alasan pegawai dan alasan keputusan atasan sebelum punch remote.
- FR-11 setiap punch check-in/out merujuk foto bukti privat yang terunggah dan koordinat GPS + accuracy/time; reject payload kurang, mime/size invalid, lokasi tidak memenuhi policy. Beri jalur koreksi manual audited.
- FR-12 presensi mengembalikan WorkLocation/kecamatan dari konfigurasi kantor untuk WFO; jangan menyimpulkan kecamatan dari IP publik. WFH/WFA memakai koordinat dan label wilayah terverifikasi/master data jika dipakai.
- FR-13 foto disimpan di private Supabase bucket via signed upload direct browser-to-storage; Express mengeluarkan URL/object key terbatas, melakukan confirm ownership/metadata; file tidak diproxy Vercel.
- FR-14 mutasi sensitif audit redacted; outbox/event hanya saat consumer konkret ada.
- FR-15 API errors konsisten dengan request_id.

## NFR presensi dan privacy
Foto/koordinat merupakan bukti sensitif. Mintakan izin kamera/GPS hanya saat pegawai memulai punch, beri tujuan dan status upload yang jelas. Minimize accuracy metadata dan retention sesuai kebijakan demo. Akses foto hanya pegawai terkait dan reviewer berwenang; signed read URL berumur pendek. Audit akses bukti. Tidak ada background tracking, face recognition, atau raw image/coordinate di logs, events, traces, analytics, URL. TLS, private bucket, type/size allowlist, checksum, signed URL one-time/short TTL, object key non-guessable, cleanup orphan upload. Tidak menjanjikan anti-spoof atau kepatuhan hukum.

## NFR umum
Parameterized SQL, input schema validation, CORS allowlist, rate/body limits, explicit trusted proxy, secrets via env. Liveness separate from readiness. Cursor pagination; bounded queries; atomic transitions; idempotency and If-Match where applicable. Accessibility keyboard/semantic labels/contrast. WSL + Docker Compose local. Hosted free tier is best effort, no SLA promise.

## Konvensi data dan status
UUID, timestamps UTC, business date + IANA timezone, money NUMERIC/currency. Cross-context opaque IDs, no FK/query across domains. 401 unauthenticated, 403 forbidden, 404 out-of-scope when needed to prevent enumeration, 409 conflict, 412 stale, 413 too large, 415 unsupported media, 422 domain validation, 429 rate limit.

## Klaim yang dilarang
Jangan menyebut produk patuh UU PDP/aturan ketenagakerjaan, production-grade/anti-fraud/anti-spoof, SLA atau setara praktik Telkom tanpa validasi independen.
