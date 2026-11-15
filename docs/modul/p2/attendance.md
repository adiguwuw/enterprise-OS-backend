# Attendance — PRD, SRS, TDD dan Kontrak

Phase 2 module specification. Semua lokasi, policy, employee dan foto test adalah sintetis. Tujuan portofolio: menunjukkan workflow absensi yang dapat ditinjau, berbasis izin dan memiliki evidence. Tidak meniru kebijakan Telkom.

## Goals dan scope
Employee melihat aturan hari ini, mengajukan WFH/WFA dengan alasan, punch check-in/out dengan foto + lokasi, melihat riwayat; manager menyetujui/menolak request dengan alasan dan memeriksa attendance dalam scope; HR mengelola shift, work location, egress CIDR dan koreksi. Tidak ada background tracking, face recognition, keputusan otomatis disipliner, atau klaim anti-spoof.

## Aturan kerja
- WFO: request remote IP peer harus cocok dengan CIDR public egress aktif milik WorkLocation. Jangan percaya body/header IP dari client. Di belakang proxy, konfigurasi trusted proxy eksplisit dan uji chain. Browser tidak membaca SSID/BSSID.
- WFH/WFA: request harus disetujui manager sebelum tanggal kerja, memuat alasan pegawai dan decision reason manager. Self-approval dilarang; scope manager wajib.
- Check-in dan check-out wajib punya photo evidence yang telah di-confirm dan GPS latitude/longitude, browser accuracy_m dan capture time. Request permission hanya saat punch. GPS bukan bukti bebas spoof.
- WFO memetakan kecamatan dari konfigurasi work location yang disetujui, bukan IP geolocation. Remote area label dari request/master wilayah; simpan coordinates sebagai bukti sensitif dengan akses terbatas.
- Jika izin kamera/GPS, jaringan atau storage gagal, punch ditolak dengan pesan jelas dan jalur manual correction/exception beralasan yang diaudit. Kebijakan exception harus dapat dikonfigurasi.
- Tidak ada unggah file melalui API/Vercel. Browser upload langsung ke Supabase private bucket melalui short-lived signed upload URL dari server. Hosted storage adalah free quota best-effort; kuota penuh berarti gagal aman/local demo, tanpa billing.
- Foto bukan data template biometrik. Jangan menjalankan face matching. Tidak menyimpan lokasi kontinyu.

## State machine
Remote request: DRAFT → SUBMITTED → APPROVED | REJECTED | CANCELLED. Decision terminal; perubahan perlu request baru atau audited correction. Punch: INTENT → EVIDENCE_UPLOADED → VERIFIED → CHECKED_IN → CHECKED_OUT; manual adjustment terpisah dan append-only. Duplicate idempotency key mengembalikan hasil sama. Check-out harus mengikuti check-in; policy crossing local date ditentukan shift timezone.

## ERD
- WorkLocation(id, org_id, name, address_label, kecamatan_code, lat, lon, timezone, active). Radius dan GPS thresholds berasal dari policy version aktif.
- OfficeNetwork(id, work_location_id, cidr, label, active, valid_from, valid_to).
- RemoteWorkRequest(id, employee_id, mode WFH|WFA, from_date,to_date,requested_area_label,reason,status,submitted_at,version).
- RemoteWorkDecision(id,request_id,manager_id,decision,reason,decided_at).
- EvidenceUploadIntent(id,employee_id,object_key,mime,size,checksum,expires_at,status).
- AttendanceEvidence(id,punch_id unique,bucket,object_key unique,mime,size_bytes,checksum,captured_at,uploaded_at,verified_at,retention_until).
- AttendancePunch(id,employee_id,kind,business_date,occurred_at,timezone,mode,work_location_id,office_network_id,remote_request_id,network_match,lat,lon,accuracy_m,location_captured_at,evidence_id,idempotency_key,status,version).
Raw IP should not persist unless a documented security need exists; never log it. Use UUID, constraints and indexes. Do not FK across bounded contexts. No cascading deletion of history.

## API
Prefix /api/v1, JSON, OIDC Bearer, request_id errors, cursor lists; Idempotency-Key for punch; ETag/If-Match for manager decision/adjustment.
- POST /attendance/remote-work-requests {mode,from_date,to_date,requested_area_label,reason}
- GET /attendance/remote-work-requests/me?from=&to=
- GET /attendance/remote-work-requests/pending
- POST /attendance/remote-work-requests/{id}/decision {decision,reason}
- POST /attendance/evidence-upload-intents {mime_type,size_bytes,checksum}; response evidence_id, object_key, upload_url, expires_at, headers
- POST /attendance/evidence-upload-intents/{id}/confirm
- POST /attendance/punches {kind,work_mode,evidence_id,latitude,longitude,accuracy_m,location_captured_at,remote_work_request_id?}
- GET /attendance/me?from=&to=; GET /attendance/team?from=&to=&cursor=
- GET /attendance/punches/{id}/evidence-url (short-lived, authorized)
- POST /attendance/adjustments; POST /attendance/adjustments/{id}/approve

Never accept actor, employee_id, IP, kecamatan, matched office/network or approval state from browser payload. Server derives actor and trusted network match.

## Test scenarios
- ATT-01 duplicate same idempotency key returns same punch; different duplicate sequence conflicts.
- ATT-02 invalid check-in/out order rejected atomically.
- ATT-03 business date computed using assigned shift/work location timezone, including overnight shift.
- ATT-04 requester cannot approve own remote request or adjustment.
- ATT-05 manager can see/decide only authorized team scope.
- ATT-06 concurrent punches for same employee/state cannot create duplicates.
- ATT-07 WFO from active office CIDR matches correct location; nonmatching IP denied.
- ATT-08 spoofed X-Forwarded-For/client-supplied IP cannot bypass network policy.
- ATT-09 remote punch requires approved request covering local date and correct mode.
- ATT-10 request and decision reasons are mandatory; decision actor/reason audited.
- ATT-11 GPS missing, stale or accuracy beyond configured threshold is rejected or enters explicit exception flow.
- ATT-12 WFO kecamatan comes from configured WorkLocation; IP geolocation never used.
- ATT-13 unconfirmed, expired, wrong-owner, wrong-type or oversize upload cannot be attached.
- ATT-14 object keys and read access are private; signed evidence URL requires scoped authorization and expires.
- ATT-15 storage quota/unavailable state does not create a punch without evidence; no bytes pass through API function.
- ATT-16 adjustment is append-only, requires reason/approval and creates audit record.
- ATT-17 logs, traces, errors and events contain no photo, signed URL, raw IP, coordinates or free-text reasons.
- ATT-18 retention cleanup removes evidence metadata/object according to configured policy, with deletion audit; preserve required attendance history as policy dictates.

## Security, privacy, NFR
Least privilege on photos; private bucket; randomized object key; MIME/size allowlist; short-lived upload/read signed URL; rate limit; image metadata stripping client-side is optional defense only, not validation. Data minimization, no analytics, no cache persistent, access audit. Define retention before deployment. No false claim of compliance with UU PDP or employment law. Use synthetic photos only. Vercel body/file-system constraints require direct storage upload.

## Acceptance / Definition of Done
OpenAPI schema, DB migration and frontend spec align; unit/domain + PostgreSQL integration + API contract + browser E2E cover scenarios; authz negative tests; upload adapter tests; audit redaction; documented WSL/local demo and free-tier quota behavior. No test is considered passed until executed and recorded.


## Dashboard Admin dan policy presensi
Admin berwenang mengatur radius GPS, ambang akurasi, shift/timezone, kalender kerja/libur, masa retensi foto/koordinat, aturan exception, WorkLocation dan office CIDR melalui Dashboard Admin. Nilai policy tidak di-hard-code; server tetap memvalidasi batas keamanan dan kelayakan nilai.

Setiap perubahan memiliki tanggal efektif dan alasan, menghasilkan versi policy baru, menampilkan before/after, serta mencatat actor/waktu/audit. Punch menyimpan policy_version_id yang berlaku. Versi yang telah dipakai tidak boleh diubah atau dihapus; koreksi/rollback membuat versi baru. Cegah overlap tanggal efektif untuk scope lokasi yang sama. Hanya role attendance.policy:manage dengan org/location scope yang sesuai dapat mengubah; maker-checker dapat diwajibkan dan creator tidak boleh menyetujui versinya sendiri. Tidak ada policy aktif berarti fail closed atau exception audited sesuai konfigurasi, bukan memakai nilai default diam-diam.

API admin: GET /api/v1/attendance/admin/policies; POST /attendance/admin/policies; POST /attendance/admin/policies/{id}/versions; POST /attendance/admin/policy-versions/{id}/approve bila maker-checker aktif; GET /attendance/admin/policies/{id}/history. Versi memuat radius, accuracy, shift, timezone, kalender, retensi, exception rules, effective dates, reason dan actor. Setiap endpoint memiliki OpenAPI schema dan permission.

Model data attendance harus memiliki AttendancePolicy dan AttendancePolicyVersion. Field geofence_radius_m pada WorkLocation diganti dengan referensi konfigurasi versi policy. OfficeNetwork/CIDR juga dikelola sebagai perubahan konfigurasi efektif-tanggal dan diaudit.

### Test policy admin
POL-01 policy update valid membuat versi baru dan audit; POL-02 unauthorized/out-of-scope ditolak; POL-03 nilai invalid/alasan kosong/overlap/stale version ditolak atomik; POL-04 punch historis tetap menunjuk versi lama; POL-05 maker-checker melarang self-approval; POL-06 ketiadaan policy aktif tidak menggunakan default tersembunyi.