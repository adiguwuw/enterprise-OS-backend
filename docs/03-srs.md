# SRS dan Kriteria Penerimaan Lintas Modul

## Functional requirements
- FR-01 OIDC Authorization Code + PKCE dan validasi access token API.
- FR-02 principal terikat issuer+subject unik, bukan email saja; disable/revoke.
- FR-03 permission berupa resource/action; role mengelompokkan permission; assignment mempunyai scope dan masa berlaku.
- FR-04 ABAC server-side memeriksa scope, relasi resource, status dan waktu; default deny.
- FR-05 organisasi bertingkat berversi/efektif tanggal, lokasi dan position catalog; cegah cycle.
- FR-06 data employee/employment/assignment dan lifecycle; pemisahan field sensitif.
- FR-07 directory searchable, paginated, masked, visibility controlled.
- FR-08 mutasi penting audit redacted, outbox jika event dipakai.
- FR-09 fase lanjut mematuhi spesifikasi per modul; tak ada query DB lintas domain.
- FR-10 API errors konsisten dan punya request_id.

## NFR
Security: TLS hosted, JWT verification, server authorization, parameterized SQL, CORS allowlist, CSRF strategy bila cookie, rate/body limits, secrets env. Privacy: synthetic data, minimization, no PII in logs/traces, retention documented. Reliability: atomic state transitions, idempotent retries, optimistic concurrency, explicit degradation. Performance: cursor pagination, bounded query, index dari access pattern, ukur baseline demo. Accessibility: keyboard, semantic labels, focus, contrast, reduced motion. Portability: Docker Compose dan instruksi WSL. Maintainability: OpenAPI, migrations, ADR, automated tests.

## Konvensi data
UUID, timestamptz UTC, date bisnis + timezone IANA, monetary NUMERIC + ISO currency. Terapkan DB constraints dan validasi domain. Historis tidak cascade-delete. Sensitive data minimization/masking.

## Acceptance lintas modul
401 token invalid; 403 action tidak berizin; 404 untuk resource tak ada/di luar scope sesuai risiko enumeration; 409 domain/duplicate conflict; 412 stale version; 422 business validation; 429 throttling. Seluruh list max page size. Kasus rinci dan ID test ada di tiap dokumen modul dan matriks integrasi.

## Bukan klaim
Tidak menyatakan patuh hukum Indonesia, setara proses internal Telkom, tahan audit legal, tersertifikasi, memiliki SLA produksi atau lulus pentest.
