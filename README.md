# EnterpriseOS Backend

Backend Go untuk proyek portofolio EnterpriseOS. Semua perusahaan, pegawai, transaksi, dan data contoh adalah simulasi. Proyek ini tidak berafiliasi dengan Telkom dan tidak memakai data internal/rahasia Telkom.

## Blueprint final
Lihat `docs/README.md` untuk arsitektur, TDD, API, ERD, skenario uji, keputusan stack dan spesifikasi 24 modul.

## Lingkungan kerja
Pengembangan dilakukan dari WSL/Linux shell. Jalankan layanan lokal menggunakan Docker Compose dan gunakan data sintetis.

## Stack
Go, PostgreSQL, REST/JSON, OpenAPI; OIDC melalui Auth0 Free untuk demo hosted dan Keycloak lokal. Vercel Hobby Go Functions digunakan hanya untuk HTTP handler portfolio; proses Go lokal berjalan sebagai services terpisah.

## Biaya
Deployment memilih free tier saja. Jangan menambahkan service berbayar, memasang billing method, atau meng-upgrade plan. Bila kuota provider habis, layanan boleh berhenti sementara dan demo dijalankan lokal.

## Status
Repository ini memuat blueprint dan scaffold dokumentasi. Implementasi fitur dilakukan bertahap sesuai acceptance criteria pada blueprint.
